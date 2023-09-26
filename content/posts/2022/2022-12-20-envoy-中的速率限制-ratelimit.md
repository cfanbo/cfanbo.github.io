---
title: Envoy 中的速率限制 ratelimit
author: admin
type: post
date: 2022-12-20T03:45:08+00:00
url: /archives/32108
toc: true
categories:
 - 系统架构
tags:
 - envoy
 - ratelimit

---
在 Envoy 架构中 [Rate limit service][1] 共支持 [global rate limiting][2] 和 [local rate limit filter][3] 两种速率限制。推荐使用 https://github.com/envoyproxy/ratelimit 库。

# Global rate limiting 

Envoy 提供了两种全局限速实现

 1. 每个连接 或 每个HTTP请求 速率限制检查。
 2. 基于配额，具有定期负载报告，允许在多个 Envoy 实例之间公平共享全局速率限制。此实现适用于每秒请求负载较高的大型 Envoy 部署，这些负载可能无法在所有 Envoy 实例之间均匀平衡。

## Per connection or per HTTP request rate limiting 

Envoy 直接与全局 gRPC rate limiting service 集成，配置参考 [https://www.envoyproxy.io/docs/envoy/latest/configuration/other\_features/rate\_limit#config-rate-limit-service][1]。

速率服务可以使用任何 `RPC/IDL` 协议实现，但Envoy 提供了一个用 Go 编写的参考[实现][4]，它使用 Redis 作为后端。速率集成有以下两种特征:

 1. Network 级别过滤器：对应 `Per connection` ，Envoy 将为安装过滤器的侦听器上的每个 `新连接` 调用速率限制服务。该配置指定一个特定的域名和描述符设置为速率限制。这具有限制每秒传输侦听器的连接速率的最终效果。配置参考 [https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/network\_filters/rate\_limit_filter#config-network-filters-rate-limit][5]

1. HTTP 级别过滤器: 对应 `Per HTTP request`，Envoy 将为安装过滤器的 Listener 以及路由表指定应调用全局速率限制服务的侦听器上的每个 `新请求` 调用速率限制服务。对目标上游集群的所有请求以及从原始集群到目标集群的所有请求都可以进行速率限制。配置参考 [https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/rate_limit_filter#config-http-filters-rate-limit](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/rate_limit_filter#config-http-filters-rate-limit)

Envoy 还支持本地速率限制 `local rate limiting`，本地速率限制可以与全局速率限制结合使用，以减少全局速率限制服务的负载。例如，本地令牌桶速率限制可以吸收非常大的负载突发，否则可能会压倒全局速率限制服务。因此，速率限制分两个阶段应用，在细粒度全局限制完成作业之前，由令牌桶限制执行初始粗粒度限制。

## Quota based rate limiting 

速率限制服务的开源参考实现目前不可用。费率限制配额扩展目前可以与Google Cloud费率限制服务一起使用。

基于配额的全局速率限制只能应用于 HTTP 请求。 Envoy 将使用 HTTP 过滤器配置对请求进行分桶，并从速率限制配额服务请求配额分配。配置参考 [https://www.envoyproxy.io/docs/envoy/latest/configuration/other\_features/rate\_limit#config-rate-limit-quota-service][6]

# Local rate limiting 

Envoy 除了支持通过 [local rate limit filter](https://www.envoyproxy.io/docs/envoy/latest/configuration/listen%60ers/network_filters/local_rate_limit_filter#config-network-filters-local-rate-limit) 过滤器对 L4 连接进行本地（非分布式）速率限制。同时还支持通过 [HTTP local rate limit filter][7] 对 HTTP 请求进行本地速率限制。这可以在 `侦听器`级别或更具体的级别（例如： `virtual host` 或 `route level`）全局激活。

也就是说对于 `local rate limit` 可以在两个地方对其进行配置，一个是 Listener 中的 [Network filter][8] ，另一个是 HTTP 中的 [HTTP Filters][9]，注意配置在这两点的不同之处。

一般本地速率限制与全局速率限制结合使用，以减少全局速率限制服务的负载。

当请求的路由或虚拟主机具有每个过滤器本地速率限制配置时，HTTP 本地速率限制过滤器应用令牌桶速率限制。

如果检查了本地速率限制令牌桶，并且没有可用令牌，则返回 `429` 响应（响应是可配置的）。本地速率限制过滤器然后设置 [x-envoy-ratelimited][10] 响应标头。可以配置要返回的[其他响应标头][11]。

根据配置项 [local\_rate\_limit\_per\_downstream_connection][12] 的值，令牌桶在所有 workers 之间共享或在 a per connection 的基础上共享。这导致每个 Envoy 进程或每个下游连接应用本地速率限制。默认情况下，速率限制适用于每个 Envoy 进程。

## 配置示例 

以下配置来自官方提供的示例文件 [https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http\_filters/local\_rate\_limit\_filter#example-configuration][13]

 1. 全局设置速率限制器的示例过滤器配置（例如：所有虚拟主机/路由共享相同的令牌桶）[下载][14]：

```
13          http_filters:
14          - name: envoy.filters.http.local_ratelimit
15            typed_config:
16              "@type": type.googleapis.com/envoy.extensions.filters.http.local_ratelimit.v3.LocalRateLimit
17              stat_prefix: http_local_rate_limiter
18              token_bucket:
19                max_tokens: 10000
20                tokens_per_fill: 1000
21                fill_interval: 1s
22              filter_enabled:
23                runtime_key: local_rate_limit_enabled
24                default_value:
25                  numerator: 100
26                  denominator: HUNDRED
27              filter_enforced:
28                runtime_key: local_rate_limit_enforced
29                default_value:
30                  numerator: 100
31                  denominator: HUNDRED
32              response_headers_to_add:
33              - append_action: OVERWRITE_IF_EXISTS_OR_ADD
34                header:
35                  key: x-local-rate-limit
36                  value: 'true'
37              local_rate_limit_per_downstream_connection: false
```

这里 `rate limit` 工作在HTTP 中的 `HTTP Filters` 过滤器，使用扩展 [extensions.filters.http.local_ratelimit.v3.LocalRateLimit][15] 。

`token_bucket`： 是令牌桶的配置(参考，这里的意思是说令牌桶共 10000 个，每 `token_bucket.fill_interval` 个周期定时向桶中填充 `token_bucket.tokens_per_fill` 个令牌。

`filter_enable` 启用状态。

1. 全局禁用速率限制器但为特定路由启用的示例过滤器配置 [下载](https://www.envoyproxy.io/docs/envoy/latest/_downloads/1f38d20a31590590dae5758871b7dd1b/local-rate-limit-route-specific-configuration.yaml)：


```
13          http_filters:
14          - name: envoy.filters.http.local_ratelimit
15            typed_config:
16              "@type": type.googleapis.com/envoy.extensions.filters.http.local_ratelimit.v3.LocalRateLimit
17              stat_prefix: http_local_rate_limiter
```

1. 路由具体配置 [下载](https://www.envoyproxy.io/docs/envoy/latest/_downloads/1f38d20a31590590dae5758871b7dd1b/local-rate-limit-route-specific-configuration.yaml):


```
21          route_config:
22            name: local_route
23            virtual_hosts:
24            - name: local_service
25              domains: ["*"]
26              routes:
27              - match: {prefix: "/path/with/rate/limit"}
28                route: {cluster: service_protected_by_rate_limit}
29                typed_per_filter_config:
30                  envoy.filters.http.local_ratelimit:
31                    "@type": type.googleapis.com/envoy.extensions.filters.http.local_ratelimit.v3.LocalRateLimit
32                    stat_prefix: http_local_rate_limiter
33                    token_bucket:
34                      max_tokens: 10000
35                      tokens_per_fill: 1000
36                      fill_interval: 1s
37                    filter_enabled:
38                      runtime_key: local_rate_limit_enabled
39                      default_value:
40                        numerator: 100
41                        denominator: HUNDRED
42                    filter_enforced:
43                      runtime_key: local_rate_limit_enforced
44                      default_value:
45                        numerator: 100
46                        denominator: HUNDRED
47                    response_headers_to_add:
48                    - append_action: OVERWRITE_IF_EXISTS_OR_ADD
49                      header:
50                        key: x-local-rate-limit
51                        value: 'true'
52              - match: {prefix: "/"}
53                route: {cluster: default_service}
```

这里一共配置了两个路由，对其中的一个路由 “/path/with/rate/limit” 进行了限制，第二个路由不做任何限制。

请注意，如果此过滤器配置为全局禁用并且没有虚拟主机或路由级别令牌桶，则不会应用任何速率限制。

# 总结 

 1. 对于 `global rate limit` 和 `local rate limit` 两者即可以工作在 [Listeners][16] 中的 [Network filters][17]，也可以工作在 HTTP 中的 [HTTP filters][17] 中。
 2. 通常情况下 `local rate limit` 和 `global rate limit` 两者配合使用，优先使用 `local rate limit` 服务，以减少对 `global rate limit` 服务的负载。
 3. 对于 `local rate limit` 主要是基于令牌桶限制算法，见 [https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/local\_ratelimit/v3/local\_rate_limit.proto#envoy-v3-api-field-extensions-filters-http-local-ratelimit-v3-localratelimit-local-rate-limit-per-downstream-connection][12]

# 参考 

 * [https://www.envoyproxy.io/docs/envoy/latest/intro/arch\_overview/other\_features/global\_rate\_limiting][18]
 * [https://www.envoyproxy.io/docs/envoy/latest/intro/arch\_overview/other\_features/local\_rate\_limiting][19]
 * [https://www.aboutwayfair.com/tech-innovation/understanding-envoy-rate-limits](https://www.aboutwayfair.com/tech-innovation/understanding-envoy-rate-limits)

 [1]: https://www.envoyproxy.io/docs/envoy/latest/configuration/other_features/rate_limit#config-rate-limit-service
 [2]: https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/other_features/global_rate_limiting#arch-overview-global-rate-limit
 [3]: https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/other_features/local_rate_limiting#arch-overview-local-rate-limit
 [4]: https://github.com/envoyproxy/ratelimit
 [5]: https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/network_filters/rate_limit_filter#config-network-filters-rate-limit
 [6]: https://www.envoyproxy.io/docs/envoy/latest/configuration/other_features/rate_limit#config-rate-limit-quota-service
 [7]: https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/local_rate_limit_filter#config-http-filters-local-rate-limit
 [8]: https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/network_filters/local_rate_limit_filter#config-network-filters-local-rate-limit
 [9]: https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/local_rate_limit_filter#id2
 [10]: https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/router_filter#config-http-filters-router-x-envoy-ratelimited
 [11]: https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/local_ratelimit/v3/local_rate_limit.proto#envoy-v3-api-field-extensions-filters-http-local-ratelimit-v3-localratelimit-response-headers-to-add
 [12]: https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/local_ratelimit/v3/local_rate_limit.proto#envoy-v3-api-field-extensions-filters-http-local-ratelimit-v3-localratelimit-local-rate-limit-per-downstream-connection
 [13]: https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/local_rate_limit_filter#example-configuration
 [14]: https://www.envoyproxy.io/docs/envoy/latest/_downloads/0e039036f53cb78357a657f1cf05a026/local-rate-limit-global-configuration.yaml
 [15]: https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/local_ratelimit/v3/local_rate_limit.proto
 [16]: https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/listeners
 [17]: https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/local_rate_limit_filter
 [18]: https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/other_features/global_rate_limiting
 [19]: https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/other_features/local_rate_limiting