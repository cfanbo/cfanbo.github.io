---
title: envoy中 lua filter 与 wasm filter使用教程
url: "/posts/Lua-Filter-and-Wasm-Filter-in-Envoy"
date: 2023-09-22T13:14:18+08:00
type: post
categories: 
- 程序开发
tags:
- envoy
- lua 
- wasm
---

在 Envoy 中当我们需要对 [http_connection_manager](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/network/http_connection_manager/v3/http_connection_manager.proto#envoy-v3-api-msg-extensions-filters-network-http-connection-manager-v3-httpconnectionmanager) 中的请求进行修改时，如添加或删除一个请求header，一般通过 `HTTP Filter` 过滤器来实现。 

而在Envoy 包含的几十个Filter中，通常会选择 `Lua Filter `([extensions.filters.http.lua.v3.Lua](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/lua/v3/lua.proto#extensions-filters-http-lua-v3-lua)) 或 `Wasm Filter` ([extensions.filters.http.wasm.v3.Wasm](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/wasm/v3/wasm.proto#extensions-filters-http-wasm-v3-wasm))这两类过滤器。

# Lua Filter 与 Wasm Filter

下表是 `Lua Filter` 与 `HTTP Filter` 的对比

|          | Lua Filter              | Wasm Filter                              |
| -------- | ----------------------- | ---------------------------------------- |
| 编程语言 | Lua，解释型脚本语言     | WebAssembly，编译型语言                  |
| 运行环境 | Envoy 内置的 Lua 虚拟机 | Envoy 内嵌的 WebAssembly 虚拟机          |
| 生态系统 | 丰富的 Lua 库可供使用   | 逐渐形成的 WebAssembly 生态系统          |
| 性能     | 较低                    | 较高                                     |
| 安全性   | 较弱                    | 较强                                     |
| 可移植性 | 受宿主环境和依赖库限制  | 平台无关的二进制格式，可在不同环境中运行 |

在不同的环境中Lua 的行为和功能可能略有差异，特别是在与底层操作系统和硬件交互的方面，而 Wasm 则没有这个问题。

但对于选择哪类 Filter 扩展 Envoy 的过滤器逻辑时，需要根据你的需求和对编程语言的熟悉程度。

总之 `Lua Filter` 在易用性、生态系统和成熟度方面具有优势，而 `Wasm Filter` 则在性能、安全性和可移植性方面具有很大的优势。 

Envoy 提供了对这两种过滤器的支持，你可以根据具体情况和偏好进行选择。

# Lua Filter 

如果选择 Lua Filter 来实现的话，只需要根据官方提供的文档(https://www.envoyproxy.io/docs/envoy/latest/start/sandboxes/lua)直接编写lua脚本实现以下两个函数即可。

```
envoy_on_request（request_handle）
envoy_on_response（response_handle）
```

如官方提供的示例 https://github.com/envoyproxy/envoy/blob/main/examples/lua/envoy.yaml

```lua
function envoy_on_response(response_handle)
  response_handle:headers():add("header_key_1", "header_value_1")
end


local mylibrary = require("lib.mylibrary")
function envoy_on_request(request_handle)
  request_handle:headers():add("foo", mylibrary.foobar())
end
```

在请求函数 `envoy_on_request()` 里添加了一个 `foo` 请求header, 值是通过调用 [lib.mylibrary](https://github.com/envoyproxy/envoy/blob/main/examples/lua/lib/mylibrary.lua) 文件中的 `foobar()` 函数来实现的。这样当一个请求到达envoy时，envoy自动添加一个foo新的请求头，然后再将这个HTTP请求发给upstream服务，这时上游的服务即可以将这个请求头读取出来。

在响应函数 `envoy_on_response()` 中，同样是添加了一个请求header，不过它是在envoy将HTTP响应内容发送客户端时才添加的，这个时候客户端可以看到这个 `header_key_1`请求头。

下面我们测试一下使用Lua实现的效果。

首先我们修改一个 `envoy.yaml` 里的配置，主要修改两个地方。

一个是路由匹配规则，将 `prefix: "/multiple/lua/scripts"` 修改为 `prefix: "/"`，表示对所有路由都启用lua filter 过滤器。然后修改上游集群服务，最后两行改为 httpbin.org ，端口为 `80`
```yaml
address: web_service
port_value: 8080
```

修改为

```yaml
address: httpbin.org
port_value: 80
```

然后启动服务

```shell
$ envoy -c envoy.yaml
```

此时 envoy 将监听在 `8000` 端口。

再新开一个终端，发送一个HTTP请求

```shell
$ curl -v localhost:8000/headers
*   Trying 127.0.0.1:8000...
* Connected to localhost (127.0.0.1) port 8000 (#0)
> GET /headers HTTP/1.1
> Host: localhost:8000
> User-Agent: curl/8.1.2
> Accept: */*
>
< HTTP/1.1 200 OK
< date: Fri, 22 Sep 2023 02:32:24 GMT
< content-type: application/json
< content-length: 237
< server: envoy
< access-control-allow-origin: *
< access-control-allow-credentials: true
< x-envoy-upstream-service-time: 613
< header_key_1: header_value_1  # Lua添加的响应header
<
{
  "headers": {
    "Accept": "*/*",
    "Foo": "bar",  # Lua 添加的请求header
    "Host": "localhost",
    "User-Agent": "curl/8.1.2",
    "X-Amzn-Trace-Id": "Root=1-650cfcb8-64bbb20412a21c2f4219e789",
    "X-Envoy-Expected-Rq-Timeout-Ms": "15000"
  }
}
```

可以看到此时 `Lua Filter` 已生效。总体来说Lua脚本的实现还是非常的简单的，只需要了解一些Lua的基本语法就可以了。



# Wasm Filter

下面我们再看一下通过 Wasm Filter 的实现

## Wasm扩展概述

在 Envoy 中可通过 `Wasm ABI` 将Envoy内部 C++ API ”翻译“ 到 `Wasm Runtime`。

Wasm 运行时类型默认为 Envoy 构建时使用的第一个可用的 Wasm 引擎。

优先级依次为 `v8 -> wasmtime -> wamr -> wavm`。可用的 Wasm 运行时类型注册为扩展。 Envoy 代码库中包含以下运行时（https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/wasm/v3/wasm.proto#extensions-wasm-v3-vmconfig）：

| Name                                                         | Description                                                  | 状态   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------ |
| [envoy.wasm.runtime.v8](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/wasm/v3/wasm.proto#extension-envoy-wasm-runtime-v8) | V8-based runtime                                             | 启用   |
| [envoy.wasm.runtime.wasmtime](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/wasm/v3/wasm.proto#extension-envoy-wasm-runtime-wasmtime) | Wasmtime runtime                                             | 未启用 |
| [envoy.wasm.runtime.wamr](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/wasm/v3/wasm.proto#extension-envoy-wasm-runtime-wamr) | [github](https://github.com/bytecodealliance/wasm-micro-runtime/) | 未启用 |
| [envoy.wasm.runtime.wavm](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/wasm/v3/wasm.proto#extension-envoy-wasm-runtime-wavm) | WAVM runtime                                                 | 未启用 |
| [envoy.wasm.runtime.null](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/wasm/v3/wasm.proto#extension-envoy-wasm-runtime-null) | Compiled modules linked into Envoy                           |        |

>  对于 `envoy.wasm.runtime.null` ，必须编译Wasm模块并将其链接到Envoy二进制文件中。注册的名称在代码字段中以 `inline_string` 的形式给出。


## 开发SDK

Rust: https://github.com/proxy-wasm/proxy-wasm-rust-sdk

C++: https://github.com/proxy-wasm/proxy-wasm-cpp-sdk

Go: https://github.com/tetratelabs/proxy-wasm-go-sdk

Python: https://github.com/bytecodealliance/wasmtime-py

开发 wasm可以使用Rust、C++、Go 或 Pytohon任意一种语言，不过强烈推荐采用 Rust 开发，因为它开发出来的是GC 二进制文件，相比用Go 开发，无论是性能还是文件大小都比较占优势。

官方提供的wasm示例参考：https://www.envoyproxy.io/docs/envoy/latest/start/sandboxes/wasm-cc。



## 开发 Wasm

这里我们以Rust SDK仓库中 https://github.com/proxy-wasm/proxy-wasm-rust-sdk/blob/master/examples/http_headers/README.md 代码为例，看一下其实现与配置。

`Cargo.yaml`配置

```yaml
[package]
publish = false #表示禁止向包仓库发布该包，该包只用于本地或特定的使用场景
name = "proxy-wasm-example-http-headers"
version = "0.0.1"
authors = ["Piotr Sikora <piotrsikora@google.com>"]
description = "Proxy-Wasm plugin example: HTTP headers"
license = "Apache-2.0"
edition = "2018"

# 定义一个库 cdylib
[lib]
crate-type = ["cdylib"]

[dependencies]
log = "0.4" 
proxy-wasm = { path = "../../" } # 指定依赖项

# 发布构建配置
[profile.release]
lto = true #启用链接时间优化（Link-Time Optimization）
opt-level = 3 #启用最高级别的优化
codegen-units = 1 #指定编译单元的数量为 1
panic = "abort" #在发生 panic 时直接终止程序
strip = "debuginfo" #去除调试信息
```

在 `dependencies` 引入了当前仓库的依赖项，也就是 `/src/` 目录里几个核心文件。这里还引入了 `log` 这个库。

1. 文件 `/src/lib.rs`内容

```rust
use log::info;
use proxy_wasm::traits::*;
use proxy_wasm::types::*;

// 通过宏 proxy_wasm::main! 定义入口点和根上下文。
// 在这里，我们设置日志级别为 LogLevel::Trace，并将根上下文设置为 HttpHeadersRoot。
proxy_wasm::main! {{
    proxy_wasm::set_log_level(LogLevel::Trace);
    proxy_wasm::set_root_context(|_| -> Box<dyn RootContext> { Box::new(HttpHeadersRoot) });
}}

struct HttpHeadersRoot;

impl Context for HttpHeadersRoot {}

impl RootContext for HttpHeadersRoot {
    fn get_type(&self) -> Option<ContextType> {
        Some(ContextType::HttpContext)
    }

    fn create_http_context(&self, context_id: u32) -> Option<Box<dyn HttpContext>> {
        Some(Box::new(HttpHeaders { context_id }))
    }
}

struct HttpHeaders {
    context_id: u32,
}

impl Context for HttpHeaders {}

impl HttpContext for HttpHeaders {
    fn on_http_request_headers(&mut self, _: usize, _: bool) -> Action {
        // 添加一个请求头部
        self.add_http_request_header("req-hello", "req-world");
      
        for (name, value) in &self.get_http_request_headers() {
            info!("#{} -> {}: {}", self.context_id, name, value);
        }

        match self.get_http_request_header(":path") {
            Some(path) if path == "/hello" => {
                self.send_http_response(
                    200,
                    vec![("Hello", "World"), ("Powered-By", "proxy-wasm")],
                    Some(b"Hello, World!\n"),
                );
                Action::Pause
            }
            _ => Action::Continue,
        }
    }

    fn on_http_response_headers(&mut self, _: usize, _: bool) -> Action {
        // 添加一个响应头
        self.add_http_response_header("resp_header_key_1", "resp_header_value_1");
      
        for (name, value) in &self.get_http_response_headers() {
            info!("#{} <- {}: {}", self.context_id, name, value);
        }
        Action::Continue
    }

    fn on_log(&mut self) {
        info!("#{} completed.", self.context_id);
    }
}
```

> 下面对 https://github.com/proxy-wasm/proxy-wasm-rust-sdk/blob/master/examples/http_headers/ 提供的内容进行了部分修改，主要为了演示 request header 和 response header 的修改效果。
>
> `proxy-wasm` 是一个用于构建网络代理的库，它提供了一组用于拦截和修改网络流量的 API。

上面我们在官方的基础上添加了两行代码，一个是添加了一个请求头`req-hello: req-world`

 ```rust
 fn on_http_request_headers(&mut self, _: usize, _: bool) -> Action {
     // 添加一个请求头部
     self.add_http_request_header("req-hello", "req-world");
     
   	...
 }
 ```

同时还在响应时添加了`esp_header_key_1", "resp_header_value_1` 响应头

```rust
fn on_http_response_headers(&mut self, _: usize, _: bool) -> Action {
		// 添加一个响应头
    self.add_http_response_header("resp_header_key_1", "resp_header_value_1");
  
  	...
}
```

2. 编译文件

```shell
$ cargo build --target wasm32-wasi --release
```

 这时将生成一个目标文件 `target/wasm32-wasi/release/proxy_wasm_example_http_headers.wasm`



3. `envoy.yaml`配置(参考 https://github.com/envoyproxy/envoy/blob/main/examples/wasm-cc/envoy.yaml)

```yaml
static_resources:
  listeners:
  - address:
      socket_address:
        address: 0.0.0.0
        port_value: 10000
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          codec_type: auto
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains:
              - "*"
              routes:
              - match:
                  prefix: "/"
                route:
                  cluster: web_service

          http_filters:
          - name: envoy.filters.http.wasm
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.wasm.v3.Wasm
              config:
                name: "my_plugin"
                root_id: "my_root_id"
                # if your wasm filter requires custom configuration you can add
                # as follows
                configuration:
                  "@type": "type.googleapis.com/google.protobuf.StringValue"
                  value: |
                    {}
                vm_config:
                  vm_id: "my_vm_id"
                  code:
                    local:
                      filename: "./target/wasm32-wasi/release/proxy_wasm_example_http_headers.wasm"
          - name: envoy.filters.http.router
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router

  clusters:
  - name: web_service
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: service1
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: httpbin.org
                port_value: 80
```

相关配置可参考 https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/wasm/v3/wasm.proto#envoy-v3-api-msg-extensions-wasm-v3-vmconfig

上面官方提供的配置是通过 docker-compose 来运行的，这里为了让大家更好的理解如何在envoy调用wasm的，放弃这种方式，直接采用原生的运行方式

这里我们只需要将 

````
filename: "/etc/envoy/proxy-wasm-plugins/proxy_wasm_example_http_headers.wasm"
````

这行配置路径修改成上面生成的文件路径 

```
filename: "./target/wasm32-wasi/release/proxy_wasm_example_http_headers.wasm"
```

 好了，现在我们运行一下envoy服务

```shell
$ envoy -c envoy.yaml
```



新开一个终端，发起一个请求

```shell
$ curl -v http://localhost:10000/headers
*   Trying 127.0.0.1:10000...
* Connected to localhost (127.0.0.1) port 10000 (#0)
> GET /headers HTTP/1.1
> Host: localhost:10000
> User-Agent: curl/8.1.2
> Accept: */*
>
< HTTP/1.1 200 OK
< date: Fri, 22 Sep 2023 05:02:11 GMT
< content-type: application/json
< content-length: 249
< server: envoy
< access-control-allow-origin: *
< access-control-allow-credentials: true
< x-envoy-upstream-service-time: 376
< resp_header_key_1: resp_header_value_1
<
{
  "headers": {
    "Accept": "*/*",
    "Host": "localhost",
    "Req-Hello": "req-world",
    "User-Agent": "curl/8.1.2",
    "X-Amzn-Trace-Id": "Root=1-650d1fd3-756b39585a7e1e3519a11ac7",
    "X-Envoy-Expected-Rq-Timeout-Ms": "15000"
  }
}
```

可以看到请求头`Req-Hello` 和 响应头 `resp_header_key_1`，至此说明我们的 `wasm  filter` 正常工作了。

# 总结 

对于这两类 filter 如何选择，需要根据具体情况来决定。如果你特别的在意性能的话，则还是推荐wasm  filter，同时最好使用Rust语言开发。如果对性能要求不高的话，或者 lua filter 是一个不错的选择。

# 参考资料

- https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/wasm/wasm

- https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/lua/v3/lua.proto#extensions-filters-http-lua-v3-lua

- https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/wasm/v3/wasm.proto

- https://github.com/proxy-wasm/proxy-wasm-rust-sdk

- https://antweiss.com/blog/extending-envoy-with-wasm-and-rust/

- https://content.red-badger.com/resources/extending-istio-with-rust-and-webassembly

  
