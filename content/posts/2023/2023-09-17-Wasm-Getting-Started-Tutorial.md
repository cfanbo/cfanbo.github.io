---
title: "WebAssembly开发入门"
date: 2023-09-17T10:04:13+08:00
draft: true
---


# wasm简介

WebAssembly（Wasm）是一种通用字节码技术，它可以将其他编程语言（如 Go、Rust、C/C++ 等）的程序代码编译为可在浏览器或服务端环境直接执行的字节码程序。



# 使用场景

主要有两个使用场景，分别为 浏览器 和 服务端。

## 浏览器

wasm最早的出现是为了解决浏览器端的性能问题，让web应用可以达到与本地原生应用类似的性能。

![Image](https://blogstatic.haohtml.com/uploads/2023/09/640.png)

对于浏览器chrome 采用了v8 javascript引擎，其内置了一个 Wasm Runtime，因此可以实现对 wasm 的支持，这也正是浏览器可以运动wasm的原因。


## 服务端

2019 年 3 月，Mozilla 推出了 WebAssembly 系统接口（Wasi），以标准化 WebAssembly 应用程序与系统资源之间的交互抽象，例如文件系统访问、内存管理和网络连接，该接口类似于 POSIX 等标准 API。**Wasi 规范的出现极大地扩展了 WebAssembly 的应用场景，使得 Wasm 不仅限于在浏览器中运行，而且可以在服务器端得到应用**。同时，平台开发者可以针对特定的操作系统和运行环境提供 Wasi 接口的不同实现，允许跨平台的 WebAssembly 应用程序运行在不同的设备和操作系统上。

![Image](https://blogstatic.haohtml.com//uploads/2023/09/640-20230916111306855.png)


开发教程:

- Develop WASM Apps： https://wasmedge.org/docs/develop/overview
- Embed WasmEdge in Your Apps： https://wasmedge.org/docs/embed/overview



# 运行原理

下面介绍下为什么需要runtime以及常见的运行时有哪些？



## 为什么需要Runtime

WebAssembly (WASM) 需要运行时环境来提供执行和管理 WASM 模块的功能。下面是一些 wasm 需要运行时的原因：

1. 跨平台执行：WebAssembly 是一种跨平台的二进制指令集格式，用于在不同的环境中执行。运行时环境负责将 WASM 模块加载到特定的执行环境中，并提供必要的接口和功能，使其能够在不同的操作系统和硬件平台上运行。
2. 安全性限制：WebAssembly 是一种沙箱化的执行环境，它在运行时提供了严格的安全性限制。运行时负责执行这些限制，以确保 WASM 模块只能访问其限定的资源，并且不能执行恶意操作。
3. 内存管理：WASM 运行时负责管理线性内存，这意味着它负责分配、释放和管理 WASM 模块的内存资源。运行时提供了合适的内存管理功能，以确保模块运行期间的内存访问的正确性和安全性。
4. 调用外部功能：WASM 运行时提供了一种机制，使 WASM 模块能够与宿主环境进行交互，并调用外部的功能和服务。运行时环境定义了模块与宿主环境之间的接口和调用约定，使得模块能够访问特定功能，如文件系统、网络连接等。
5. 性能优化和代码优化：WASM 运行时可以对加载的 WASM 模块进行解析、编译和优化，以提高执行效率和性能。运行时环境可以实现一些优化技术，如即时编译（Just-in-Time Compilation）和调用间内联（Function Inlining），从而使模块的执行更高效。

综上所述，WASM 需要运行时环境来提供必要的功能和接口，以便在跨平台的环境中安全地加载、执行和管理模块。运行时环境扮演了连接 **WASM 模块**和宿主**环境之间**的**桥梁**角色，使得在不同的环境中使用 WASM 变得更加便捷和可靠。

## 常见 WASM Runtime

1. V8：V8 是 Google 开发的高性能 JavaScript 引擎，同时也支持执行 WebAssembly。V8 是 Chrome 浏览器的默认 JavaScript 引擎，也被其他项目广泛采用。

2. SpiderMonkey：SpiderMonkey 是 Mozilla Firefox 浏览器使用的 JavaScript 引擎，也支持执行 WebAssembly。SpiderMonkey 是一个功能强大的运行时，提供了许多高级功能和调试选项。

3. Wasmtime：Wasmtime 是一个快速、可扩展的 WebAssembly 运行时，由 Bytecode Alliance 开发。它提供了一组 API，使开发人员可以在自己的应用程序中嵌入和执行 WebAssembly 模块。

4. Wasmer：Wasmer 是一个通用的 WebAssembly 运行时，支持在各种环境中执行 WebAssembly。它提供了一组 API 和工具，使开发人员能够在不同的宿主语言中嵌入和执行 WASM 模块。Wasmer 支持多种语言绑定，包括 Rust、C/C++、Python 和 Go。同时还提供了一些功能，例如内存管理、引用类型支持和自定义 API 导入等一些有用的功能和工具。

5. Wasmer Edge：Wasmer Edge（前身是 WasmEdge） 是一个专门为边缘计算场景优化的 WASM 运行时，它可以实现将云原生和无服务器应用程序范例引入边缘计算。它是在 Wasmer 项目的基础上进行发展的，目标是在资源受限的环境（如边缘设备）上提供高性能的 WASM 执行。Wasmer Edge 通过针对边缘计算优化的内存管理、模块加载机制和 I/O 接口等功能，提供了更高效的执行体验。Wasmer Edge 还支持一些边缘设备平台的特定功能，如 ARM 架构的 NEON SIMD 指令集。

6. Emscripten：Emscripten 是一个将 C/C++ 代码编译为 WebAssembly 的工具链。它基于 LLVM 编译器框架，可以将现有的 C/C++ 代码转换为可以在 Web 浏览器或其他支持 WebAssembly 的环境中执行的 JavaScript 和 WebAssembly 混合代码。

   

这只是一些常见的 WebAssembly 运行时示例，还有其他运行时可用，具体取决于您的使用场景和需求。



# WasmEdge 简介

WasmEdge (之前名为 SSVM) 是为边缘计算优化的轻量级、高性能、可扩展的 WebAssembly (Wasm) 虚拟机，可用于云原生、边缘和去中心化的应用。WasmEdge 是目前市场上 [最快的 Wasm 虚拟机](https://ieeexplore.ieee.org/document/9214403)。WasmEdge 是由 [CNCF](https://www.cncf.io/) (Cloud Native Computing Foundation 云原生计算基金会)托管的官方沙箱项目。其[应用场景](https://github.com/WasmEdge/WasmEdge/blob/master/docs/use_cases-zh.md)包括 serverless apps, 嵌入式函数、微服务、智能合约和 IoT 设备.

它有以下几个特点：

## 高性能

- 与Linux容器相比，WasmEdge在启动时可以快100倍，在运行时可以快20%
- WasmEdge应用程序的大小可能是类似Linux容器应用程序的1/100



## WASI 扩展

- Network sockets
- Async processing
- Tensorflow inference
- Key-value stores
- Database connectors
- Gas meters



## JavaScript支持

- ES6 module and std API support
- Node and NPM module API support
- React SSR streaming
- Implement JS APIs in Rust
- Much lighter than containerized v8



## 云原生管理和编排

- Container tooling support
- Kubernetes ecosystem support
- Data plane plug-ins
- Sidecar apps in service meshes
- Dapr microservices



## 跨平台

- Linux
- Mac OS X
- Windows
- Microkernel and RTOS
- Intel x86, ARM, and M1 CPUs



## 易扩展

- 用 C++ 构建 WasmEdge 扩展
- 使用 C、Go 和 Rust 中的原生函数构建定制的 WasmEdge 运行时



# Wasm开发

开发wasm可以选择多种语言，如C/C++、Rust 或 Go，由于 Rust 可以开发无GC的应用程序，因此这里我们选择Rust，这也是大多数情况下的首选开发语言。

## 安装 Rust

```shell
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Wasi（WebAssembly System Interface）是用于 WebAssembly 的系统级接口，旨在实现 WebAssembly 在不同环境中与宿主系统交互。它提供标准化的方式，使得 WebAssembly 可以进行文件 I/O、网络操作和系统调用等系统级功能访问。

`wasm32-wasi` 是 Rust 的编译目标之一，用于将 Rust 代码编译为符合 Wasi 标准的 Wasm 模块。通过将 Rust 代码编译为 wasm32-wasi 目标，可以将 Rust 的功能和安全性引入到 WebAssembly 环境中，同时利用 wasm32-wasi 提供的标准化系统接口实现与宿主系统的交互。

为 Rust 编译器添加编译 `wasm32-wasi` 目标

```shell
$  rustup target add wasm32-wasi
```

验证

```shell
$ rustup target list --installed
wasm32-wasi
x86_64-apple-darwin
```

看到 `wasm32-wasi` 表示安装编译目标成功



## 安装 WasmEdge

由于 wasm 需要 Runtime 的支持才可以运行，因此我们还需要选择一种Runtime，前面我们介绍了 WasmEdge 这个 Runtime，因此这里选择 WasmEdge 运行时

```shell
$ curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash
```

验证

```shell
$ wasmedge -v
wasmedge version 0.13.4
```



## 开发Wasm

直接参考官网提供的 Rust开发Wasm教程(https://wasmedge.org/docs/develop/rust/http_service/server)，这里是一个 HTTPServer 服务。

创建一个 rust 项目

```shell
$ cargo new hello
```

添加 `Cargo.toml` 配置依赖项

```toml
[package]
name = "hello"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
bytes = "1"
tokio_wasi = { version = "1", features = ["rt", "macros", "net", "time", "io-util"]}
warp_wasi = "0.3"
```

这里需要引入 tokio_wasi 和 warp_wasi 依赖。

> `tokio-wasi` 是一个用于与 WebAssembly System Interface（WASI）交互的 Rust 库，它与 Tokio 异步运行时集成，为在 WASI 环境中编写异步 I/O 代码提供了便捷的工具和功能。

`src/main.rs` 内容

```rust
use warp::Filter;

#[tokio::main(flavor = "current_thread")]
async fn main() {
    // GET /
    let help = warp::get()
        .and(warp::path::end())
        .map(|| "Try POSTing data to /echo such as: `curl localhost:8080/echo -XPOST -d 'hello world'`\n");

    // POST /echo
    let echo = warp::post()
        .and(warp::path("echo"))
        .and(warp::body::bytes())
        .map(|body_bytes: bytes::Bytes| {
            format!("{}\n", std::str::from_utf8(body_bytes.as_ref()).unwrap())
        });

    let routes = help.or(echo);
    warp::serve(routes).run(([0, 0, 0, 0], 8080)).await
}
```

编译wasm

```shell
# Build the Rust code
cargo build --target wasm32-wasi --release

# Use the AoT compiler for better performance
wasmedge compile target/wasm32-wasi/release/hello.wasm hello.wasm

# Run the example
wasmedge hello.wasm
```

> 以上内容来自 https://github.com/WasmEdge/wasmedge_hyper_demo 仓库中的  `server-warp`
>
> ```shell
> git clone https://github.com/WasmEdge/wasmedge_hyper_demo
> cd wasmedge_hyper_demo/server-warp
> ```



开启一个新终端测试

```shell
$ curl http://localhost:8080/echo -X POST -d "WasmEdge"
WasmEdge
```

可以看到 HTTPServer 服务是OK的。

## 容器化

要想实现容器化，需要对Docker进行一些设置，分 Decoker Desktop 和 Docker CLI 版本

- [Install Docker Desktop + Wasm (Beta)](https://docs.docker.com/desktop/wasm/)
- [Install Docker CLI + Wasm](https://github.com/chris-crone/wasm-day-na-22/tree/main/server)



这里以 Docker Desktop  为例，参考教程： https://wasmedge.org/docs/start/getting-started/quick_start_docker 

安装 Docker Desktop 4.15+，同时启用 Containerd 映像存储功能。

![Image](https://blogstatic.haohtml.com//uploads/2023/09/640-20230916132145746.png)

创建 Dockerfile 文件

```dockerfile
FROM wasmedge/slim-runtime:0.10.1
COPY hello.wasm /http-server.wasm
CMD ["wasmedge", "--dir", ".:/", "/http-server.wasm"]
```

构建镜像

```shell
$ docker build -t cfanbo/wasm-app:v1 .
```

启动容器服务

```shell
$ docker run -itd -p 8080:8080 --name wasmapp cfanbo/wasm-app:v1
```

测试

```shell
$ curl http://localhost:8080/echo -X POST -d "WasmEdge"
WasmEdge
```

另外也可以通过多阶段构建实现，参考教程 https://github.com/second-state/rust-examples/blob/main/hello/Dockerfile

# 参考资料

- https://wasmedge.org/
- https://wasmedge.org/book/en/
- https://wasmedge.org/docs/start/overview
- https://github.com/WasmEdge/wasmedge_hyper_demo
- https://github.com/WasmEdge/WasmEdge
- https://github.com/second-state
- https://www.secondstate.io/
- https://github.com/second-state/wasm-learning
- https://docs.docker.com/desktop/wasm/
- https://mp.weixin.qq.com/s/zvRivwrOJGhJqfav9_4EVg