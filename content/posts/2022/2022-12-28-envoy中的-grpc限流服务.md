---
title: Envoy中的 gRPC限流服务
author: admin
type: post
date: 2022-12-28T02:16:20+00:00
url: /archives/32129
toc: true
categories:
 - 系统架构
tags:
 - envoy
 - ratelimit

---
[上一节](https://blog.haohtml.com/archives/32108) 我们大概介绍了一下Envoy中有关速率限制（限流）的一些内容，这一节我们看一下对于外部的 gRPC限流服务它又是如何工作和配置的。

在 Envoy 中对服务限流的配置除了可以在 Envoy 本身中实现外，还可以在通过外部服务实现，此时 Envoy 将通过 gRPC 协议调用外部限流服务，官方对此实现有一套现成的解决方案，主要是redis数据库+令牌桶算法实现，可参考官方

本文中的限制器或限流器均是同一个意思。

# Envoy 实现限流 

此实现是基于令牌桶算法实现，本身比较的简单，比较适合一般的使用场景。

这里是官方提供的一个配置[示例][1]

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

重点关注配置项 `token_bucket` ，这里的配置表示当前最多有 10000 个令牌可以被使用，其中令牌在使用的过程中，只要桶中不足10000 个令牌时，则会以每秒再产生 1000 个令牌的速度产生新的令牌并放入令牌桶中，这样就可以实现后期每秒 1000个请求的需求。

这种配置方法比较简单，也不需要依赖第三方组件，大部分场景下已经足够我们使用了。

# gRPC限流服务 

对于这种专业的限流服务，需要依赖于一些第三方组件，官方的方案主要是基于Redis数据库来实现的，当然也可以换成其它的数据库。

对于Envoy是如何与限流服务交互的其实也很好理解

 1. 当用户发送一个请求时，Envoy首先拦截到，并会通过gRPC服务调用限流服务，此时会携带一些请求标记类的信息；
 2. 当限流服务收到这个请求后，通过分析请求中的标记生成一个带有过期时间的键KEY（如果key已存在则忽略生成步骤），其值首次为0，本质上就是一个Redis中的计数器，以后每过来一个请求则累计1
 3. 限流服务对 gRPC 请求进行响应
 4. Envoy 收到限流服务响应时，根据响应类型作相应的处理，是直接允许本次请求通过，还是直接给客户端响应 `429` 码，表示请求过多

可以看到交互还是很简单的，其实我们最主要关注是 Envoy 与 gRPC 之间是如何协同工作的。

## 定义 

应用程序请求是基于域(domain)和一组描述符(descriptors)的速率限制决定的，因此在 `Envoy` 和 `限流服务` 的配置都是根据这两个概念来实现的。

`Domain`：域是一组速率限制的容器。 Ratelimit 服务已知的所有域必须是全局唯一的。它们作为不同团队/项目具有不冲突的速率限制配置的一种方式。

`Descriptor`：描述符是域拥有的键/值对列表，Ratelimit 服务使用它来选择在限制时使用的正确速率限制。描述符区分大小写。

## 描述符列表 

每个配置都包含一个顶级描述符列表和其下可能的多个嵌套列表。格式为：

```
domain: <unique domain ID>
descriptors:
  - key: <rule key: required>
    value: <rule value: optional>
    rate_limit: (optional block)
      name: (optional)
      replaces: (optional)
       - name: (optional)
      unit: <see below: required>
      requests_per_unit: <see below: required>
    shadow_mode: (optional)
    descriptors: (optional block)
      - ... (nested repetition of above)
```

描述符列表中的每个描述符都必须有一个key。它还可以选择具有一个值以启用更具体的匹配。 “rate_limit”块是可选的，如果存在则设置实际的速率限制规则。请参阅下文了解规则的定义方式。如果不存在速率限制并且没有嵌套描述符，则描述符实际上被列入白名单。否则，嵌套描述符允许更复杂的匹配和速率限制场景。

## 速率定义 

```
rate_limit:
  unit: <second, minute, hour, day>
  requests_per_unit: <uint>
```

速率限流块指定匹配时将使用的实际速率限流。目前该服务支持每秒、分钟、小时和天的限制。未来可能会根据用户需求增加更多类型的限制。对于其它字段的定义请参考

## 配置 

上面我们介绍了一些与限流服务相关的概念，我们再看一下如何配置限流服务。要启用gRPC限流服务需要在Envoy端和gRPC服务端两个地方进行一些相关配置，且它们之间的配置要合理才可以，先看一下Envoy端配置

### Envoy端 

Envoy 配置在`envoy.yaml`

```
static_resources:
  clusters:
    - name: ratelimit
      type: STRICT_DNS
      connect_timeout: 1s
      lb_policy: ROUND_ROBIN
      protocol_selection: USE_CONFIGURED_PROTOCOL
      http2_protocol_options: {}
      load_assignment:
        cluster_name: ratelimit
        endpoints:
          - lb_endpoints:
              - endpoint:
                  address:
                    socket_address:
                      address: 127.0.0.1
                      port_value: 8081
    - name: webserver
      connect_timeout: 1s
      type: STRICT_DNS
      lb_policy: ROUND_ROBIN
      load_assignment:
        cluster_name: webserver
        endpoints:
          - lb_endpoints:
              - endpoint:
                  address:
                    socket_address:
                      address: 192.168.3.206
                      port_value: 80
  listeners:
    - address:
        socket_address:
          address: 0.0.0.0
          port_value: 8888
      filter_chains:
        - filters:
          - name: envoy.filters.network.http_connection_manager
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
              codec_type: AUTO
              stat_prefix: ingress
              http_filters:
              - name: envoy.filters.http.ratelimit
                typed_config:
                  "@type": type.googleapis.com/envoy.extensions.filters.http.ratelimit.v3.RateLimit
                  domain: mydomain
                  request_type: external
                  stage: 0
                  rate_limited_as_resource_exhausted: true
                  failure_mode_deny: false
                  enable_x_ratelimit_headers: DRAFT_VERSION_03
                  rate_limit_service:
                    grpc_service:
                      envoy_grpc:
                        cluster_name: ratelimit
                    transport_api_version: V3
```

这里我们首先通过`static_resources.cluster`声明了一个grpc限流服务集群 `ratelimit`, 监听地址为 `127.0.0.1:8080`。

接着就是对限流的配置，这里使用了全局限流器，并使用了 `HTTP` 的 `HTTP Filter` 过滤器扩展`envoy.extensions.filters.http.ratelimit.v3.RateLimit`，并指定了 `domain:mydomain`域，因此在限流服务端这个域必须得存在；`request_type: external` 表示启用外部限流服务；`rate_limit_service` 指定了限流服务集群为 `ratelimit`。到此为止我们也只是声明了一些限流服务相关的信息，那到底具体怎么使用呢？

接着我们通过为每个 `Route` 指定将在该标头中设置的任何值以及请求的路径传递给速率限制器服务，这里指定了根路由 `/`,也就是说整个域名都是有效的。

```
route_config:
  name: route
  virtual_hosts:
  - name: backend
    domains: ["*"]
    routes:
    - match: { prefix: "/" }
      route:
        cluster: webserver
        rate_limits:
        - stage: 0
          actions:
          - {request_headers: {header_name: "x-ext-auth-ratelimit", descriptor_key: "ratelimitkey"}}
          - {request_headers: {header_name: ":path", descriptor_key: "path"}}
```

这里使用了多个 `request_headers` 项，此时将表达 `joint key` 限流服务，除了`request_headers`外还有其它几个字段，它必须是下面的其中一项：

 * [source_cluster][2]
 * [destination_cluster][3]
 * [request_headers][4]
 * [remote_address][5]
 * [generic_key][6]
 * [header\_value\_match][7]
 * [dynamic_metadata][8]
 * [metadata][9]
 * [extension][10]
 * [masked\_remote\_address][11]
 * [query\_parameter\_value_match][12]

有一点需要特别注意，当为多个路由指定了不同的限流配置时，其先后顺序是有一定的影响的，对于Envoy来讲，是从上到下进行服务请求，因此都是将根路由`/` 放在配置的最下方，如

```
route_config:
  name: route
  virtual_hosts:
  - name: backend
    domains: ["*"]
    rate_limits:
      actions:
      - generic_key:
          descriptor_value: "bar"
          descriptor_key: "bar"
    routes:
    - match:
        prefix: /header/
      route:
        cluster: webserver
        rate_limits:
        - actions: # 支持多项配置
          - generic_key:
              descriptor_value: "foo"
              descriptor_key: "foo"
    # 请求头
    - match:
        prefix: /post
      route:
        cluster: httpbin
        rate_limits:
          stage: 0
          actions:
          - header_value_match:
              descriptor_key: "request"
              descriptor_value: "post_method"
              headers:
                name: ":method"
                string_match:
                  exact: "GET"
    - match:
        prefix: /anything/
      route:
        cluster: httpbin
        rate_limits:
          actions:
          - request_headers:
              descriptor_key: "ratelimitkey"
              header_name: "x-ext-ratelimit"
          - request_headers:
              descriptor_key: "ratelimitkey-2"
              header_name: "x-ext-value"
    #　域名全局限制
    - match:
        prefix: /
      route:
        cluster: webserver
```

### 限流服务端 

上面是Envoy端的配置，下面我们再看看gRPC限制服务端的配置

```
domain: mydomain
descriptors:
  - key: ratelimitkey
    descriptors:
      - key: path
        rate_limit:
          requests_per_unit: 2
          unit: second
  - key: database
    value: default
    rate_limit:
      unit: second
      requests_per_unit: 500
```

指定域为 `mydomain` 与Envoy端的一致，而 `descriptiors` 则表示描述符，并且描述符是支持嵌套的。

此配置表示采用 `ratelimitkey` 和 `path` 附带的值，并将它们构建为用于速率限制的联合密钥。

我们这里只指定了两个配置，但本文章中我们只用到了第一个配置项，看到配置还是挺简单的。

然后我们参考官方的方案，先设置一些环境变量，再启用服务

```
git clone https://github.com/envoyproxy/ratelimit.git
cd ratelimit
make compile
​
​
export USE_STATSD=false LOG_LEVEL=debug REDIS_SOCKET_TYPE=tcp REDIS_URL=192.168.3.58:6379 RUNTIME_ROOT=/home/sxf/workspace/ratelimit RUNTIME_SUBDIRECTORY=ratelimit
```

环境变量 `RUNTIME_ROOT` 表示 RUNTIME 根目录，而 `RUNTIME_SUBDIRCTORY` 表示配置文件所在的子目录，服务启用从 `RUNTIME_ROOT/RUNTIME_SUBDIRECTORY/config/` 目录里查找所有 *.conf 配置文件，参考

这里同时指定了Redis 一些配置相关信息，并启用了Debug模式，禁用了统计功能。

```
# 将上面的配置内容写入 /home/sxf/workspace/ratelimit/ratelimit/config/config.yaml,然后启用服务
bin/ratelimit
```

如果一切正常的话，服务将输出

```
WARN[0000] statsd is not in use
INFO[0000] Tracing disabled
WARN[0000] connecting to redis on 192.168.3.58:6379 with pool size 10
DEBU[0000] Implicit pipelining enabled: false
DEBU[0000] loading domain: mydomain
DEBU[0000] Creating stats for key: 'mydomain.foo_foo'
DEBU[0000] loading descriptor: key=mydomain.foo_foo ratelimit={requests_per_unit=2, unit=MINUTE, unlimited=false, shadow_mode=false}
DEBU[0000] Creating stats for key: 'mydomain.bar_bar'
DEBU[0000] loading descriptor: key=mydomain.bar_bar ratelimit={requests_per_unit=1, unit=MINUTE, unlimited=false, shadow_mode=false}
DEBU[0000] Creating stats for key: 'mydomain.request_post_method'
DEBU[0000] loading descriptor: key=mydomain.request_post_method ratelimit={requests_per_unit=3, unit=MINUTE, unlimited=false, shadow_mode=false}
DEBU[0000] loading descriptor: key=mydomain.ratelimitkey_foo
DEBU[0000] Creating stats for key: 'mydomain.ratelimitkey_foo.ratelimitkey-2'
DEBU[0000] loading descriptor: key=mydomain.ratelimitkey_foo.ratelimitkey-2 ratelimit={requests_per_unit=3, unit=MINUTE, unlimited=false, shadow_mode=false}
DEBU[0000] waiting for runtime update
WARN[0000] Listening for gRPC on '0.0.0.0:8081'
WARN[0000] Listening for debug on '0.0.0.0:6070'
WARN[0000] Listening for HTTP on '0.0.0.0:8080'
```

最终启用了三个端口

`:8081` gRPC服务端口，与Envoy通讯使用

`:6070` golang中 pprof 性能分析，

`:8080` 查看交互端点和服务健康检查，

这里只是简单介绍了其用法，更多配置信息可查看官方网站

到此，两边的配置都基本完成了，我们可以将Envoy服务启用，并用压力测试工具访问url，会发现限流服务正在发挥作用。

# 参考资料 

 *
 * [https://github.com/jbarratt/envoy\_ratelimit\_example][13]

 [1]: https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/local_rate_limit_filter#example-configuration
 [2]: https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-ratelimit-action-source-cluster
 [3]: https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-ratelimit-action-destination-cluster
 [4]: https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-ratelimit-action-request-headers
 [5]: https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-ratelimit-action-remote-address
 [6]: https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-ratelimit-action-generic-key
 [7]: https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-ratelimit-action-header-value-match
 [8]: https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-ratelimit-action-dynamic-metadata
 [9]: https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-ratelimit-action-metadata
 [10]: https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-ratelimit-action-extension
 [11]: https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-ratelimit-action-masked-remote-address
 [12]: https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-ratelimit-action-query-parameter-value-match
 [13]: https://github.com/jbarratt/envoy_ratelimit_example