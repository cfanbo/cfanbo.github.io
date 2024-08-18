---
title: 在rust中实现自定义错误
date: 2024-08-10T14:31:27+08:00
type: post
toc: true
url: /posts/custom-errors-in-rust
categories:
  - 程序开发
tags:
  - rust
  - anyhow
  - thiserror
---



[上一篇](https://blog.haohtml.com/posts/error-hanlding-unwrap-and-expect-in-rust/#google_vignette) 我们介绍了一些错误处理的最基本的用法，主要是指对 `panic!` 、`unwrap`、`expect`  和 `?`  这些宏或函数的介绍。但这仅仅是一些最基本的处理方法，对于自定义错误这一块并没有做任何介绍。

实际开发中可能默认的错误类型，并无法满足我们的业务需求，这时一般需要通过定义自己的错误类型来实现。在rust中错误类型是通过 `enum` 枚举定义的，对此官方文档也做了一些简介，本文主要介绍一些业务开发过程中对错误的处理方案，当然主要是一些最基本的用法。

# 自定义 Error

在 Rust 中，自定义错误类型是一种常见的类型，特别是当你需要提供比标准错误类型更具体的错误信息时。Rust 中的错误处理是通过 `Result` 和 `Error` trait 来实现的。以下是如何实现一个自定义错误的示例：

1. 定义一个错误枚举类型。
2. 实现 `std::fmt::Display` 为自定义错误提供用户友好的错误信息。
3. 实现 `std::error::Error` trait，这通常是通过派生 `Error` trait 来完成的。

下面是一个简单的示例：

```rust
use std::fmt;
use std::error::Error;

// 定义一个自定义错误枚举
#[derive(Debug)]
enum MyError {
    Io(std::io::Error), // 包装标准库的I/O错误
    NotFound(String),    // 一个未找到的错误，包含一个描述信息
    InvalidInput(String), // 无效输入错误，包含一个描述信息
}

// 为自定义错误实现 Display trait
impl fmt::Display for MyError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match *self {
            MyError::Io(ref err) => write!(f, "IO error: {}", err),
            MyError::NotFound(ref desc) => write!(f, "Not found: {}", desc),
            MyError::InvalidInput(ref desc) => write!(f, "Invalid input: {}", desc),
        }
    }
}

// 为自定义错误实现 Error trait
impl Error for MyError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        match *self {
            MyError::Io(ref err) => Some(err),
            _ => None,
        }
    }
}

// 使用自定义错误类型的例子
fn do_something_that_might_fail(input: &str) -> Result<(), MyError> {
    if input.is_empty() {
        Err(MyError::InvalidInput("Input cannot be empty".to_string()))
    } else {
        // 假设这里有一些可能失败的操作
        Ok(())
    }
}

fn main() {
    match do_something_that_might_fail("") {
        Ok(_) => println!("Success!"),
        Err(e) => println!("Error occurred: {}", e),
    }
}
```

在这个示例中，`MyError` 是一个自定义的错误枚举，它包含了几种可能的错误情况。我们为它实现了 `Display` 和 `Error` trait，这样就可以在需要的时候提供错误信息，并且可以与其他错误处理机制无缝集成。在 `main` 函数中，我们展示了如何使用 `do_something_that_might_fail` 函数，它可能会返回我们的自定义错误。

执行结果

```rust
Error occurred: Invalid input: Input cannot be empty
```

这里的错误格式是在实现 `std::fmt::Display` 这个 trait 中定义的。

``` rust
 MyError::InvalidInput(ref desc) => write!(f, "Invalid input: {}", desc)
```

而错误信息 `desc` 则是在结构体类型字段 `InvalidInput(String)` 中指定的。

> 上面这种错误写法在定义Error时，通过结构体字段指定了错误 message。当然可以不指定错误信息，而是在错误信息时，再自定义错误信息



不过上面这种手动实现 std::fmt::Display 的方法太繁琐了，我们可以使用一些宏来简化这个过程， 它将帮助自动派生(derive) Display 以及一些常用的 trait。

这里以 `thiserror` 这个库为示例，来看下它的实现方式。

# thiserror

首先要安装一个 [thiserror](https://crates.io/crates/thiserror) 这个crate。

```toml
[dependencies]
thiserror = "1.0.63"
```

然后，你可以使用 `thiserror::Error` 宏来定义你的自定义错误类型：

```rust
use thiserror::Error;

#[derive(Error, Debug)]
enum MyError {
    #[error("IO 错误")] // 错误信息
    Io(#[from] std::io::Error), // 使用 #[from] 来自动转换 std::io::Error

    #[error("很抱歉，未找到 `{0}`)")] // 错误信息
    NotFound(String),

    #[error("Invalid input: `{0}`)")] // 错误信息
    InvalidInput(String),
}

// 使用自定义错误类型的例子
fn do_something_that_might_fail(input: &str) -> Result<(), MyError> {
    if input.is_empty() {
        Err(MyError::InvalidInput("Input cannot be empty".to_string()))
    } else {
        // 假设这里有一些可能失败的操作
        Ok(())
    }
}
```

这里不同类型错误信息是通过 `#[error("...")]` 来定义的，这里支持多种写法，如

```rust
#[error("{var}")] ⟶ write!("{}", self.var)
#[error("{0}")] ⟶ write!("{}", self.0)
#[error("{var:?}")] ⟶ write!("{:?}", self.var)
#[error("{0:?}")] ⟶ write!("{:?}", self.0)
```

如果包含多个参数的，则可以

```rust
enum MyError {}
    #[error("invalid header (expected {expected:?}, found {found:?})")] // 两个参数值
    InvalidHeader {
        expected: String,
        found: String,
    },
}
```

另外对于 `MyError::Io`  这类错误信息还通过 `#[from]` 来实现错误类型的转换，除此之外还有更高级的用法 ，参考文档 https://crates.io/crates/thiserror。

使用  `thiserror` 这个crate 看起来是不是比手动写要简单的多了呀！



# anyhow

`anyhow` 是一种基于特征对象的错误类型，它可以实现简化错误信息，在 Rust 应用程序中轻松地进行通用错误处理。它可以包装任何实现了 `std::error::Error` trait 的错误类型。

安装 anyhow

```toml
[dependencies]
thiserror = "1.0.63"
anyhow = "1.0.86"
```

##  基本用法

将系统内置错误和自定义错误转化为 `anyhow::Error`

```rust
use anyhow::Result;
use serde::Deserialize;
use serde_json;

#[derive(Deserialize, Debug)]
struct ClusterMap {
    name: String,
    address: String,
}

fn get_cluster_info() -> Result<ClusterMap> {
    let config = std::fs::read_to_string("cluster.json")?; // io::Result<String> 自动转换为 anyhow::Error
    let map: ClusterMap = serde_json::from_str(&config)?; // serde_json::Error 自动转换为 anyhow::Error
    Ok(map)
}
fn main() {
    match get_cluster_info() {
        Ok(_) => println!("Success"),
        Err(e) => println!("Error: {}", e),
    }
}  
```

这里 `Result<T>` 与 `anyhow::Result<T, anyhow::Error>` 是相等的，因此函数返回值也可以写成 `Result<ClusterMap, anyhow::Error> `，

上面两次调用函数都有可能产生错误，尽管返回的错误类型不一样，但当发生错误时都将自动转换为 `anyhow::Error` ，开发者不需要关心底层错误的具体类型是什么，而是统一按 `anyhow::Error` 类型处理。



## 与thiserror用法

`anyhow` 与 `thiserror` 的组合用法，主要是指通过 `thiserror` 实现的自定义错误 与 `anyhow::Error` 两种错误的互转。

```rust
use anyhow::Context;
use std::fs;
use thiserror::Error;

// 定义一个自定义错误枚举
#[derive(Error, Debug)]
enum MyError {
    #[error("IO 错误")]
    Io(#[from] std::io::Error), // 使用 #[from] 来自动转换 std::io::Error

    #[error("很抱歉，未找到 `{0}`")]
    NotFound(String),

    #[error("Invalid input: `{0}`")]
    InvalidInput(String),

    #[error(transparent)] // 显示底层的错误类型，没有附加信息
    Other(#[from] anyhow::Error), // 使用 #[from] 来自动转换 anyhow::Error
}

fn read_content(filename: &str) -> anyhow::Result<String, anyhow::Error> {
    let content = fs::read_to_string(filename).with_context(|| "Failed to read file")?; // 通过 with_context() 将内置错误转换成 anyhow::Error 错误返回
    Ok(content)
}

// 使用自定义错误类型的例子
fn do_something_that_might_fail(input: &str) -> anyhow::Result<(), MyError> {
    if input.is_empty() {
        return Err(MyError::InvalidInput("Input cannot be empty".to_string()));
    }
    // 假设这里有一些可能失败的操作
    let _result = read_content(input)?; // 将 anyhow::Error 转为 MyError::Other，之所以可以实现，是在上面定义MyError时进行了转换配置
    Ok(())
}

fn main() {
    let input = "foo.txt".to_string(); // 修改此值再看一下运行结果
    match do_something_that_might_fail(&input) {
        Ok(_) => println!("Success!"),
        // Err(e) => println!("Error occurred: {}", e), // MyError 类型直接打印错误，如果知道底层错误类型的话，也可以 match 处理
        Err(e) => {
            match e {
                // MyError 类型
                MyError::InvalidInput(v) => println!("无效请求 {}", v),
                MyError::Io(e) => println!("IO错误: {}", e),
                MyError::NotFound(v) => println!("未找到 {}", v),
                MyError::Other(e) => println!("其它错误: {}", e),
            }
        }
    }
}

```

上面是两种错误类型的转换，在  `do_something_that_might_fail()` 函数里返回的是 `MyError`类型，而内部函数 `read_content()`返回的则是 `anyhow::Error` 类型。当 `do_something_that_might_fail()` 函数返回时，又将 `anyhow::Error` 转换成了 `MyError::Other` 类型，之所以会自动转换是由于我们在定义 `MyError` 结构时，使用了`thiserror` 中的的 `#[from]`，并通过 `#[error(transparent)]` 指定了错误信息使用最底层的系统错误信息，这里是没有办法手动定义这个错误信息的。

上面在 `read_content()`函数里将系统错误转换成了`anyhow::Error`, 而在`do_something_that_might_fail` 函数里又将 `anyhow::Error` 转换成了自定义错误 `MyError`，可以看到通过使用 `thiserror` 配置，实现它们了不同类型之间的互转。



当然你也可以手动将 系统错误类型转换为 `anyhow::Error` 类型，如

```rust
// 将自定义错误转换为 anyhow::Error
read_data().map_err(|e| Error::new(e))?;
```

可以看到，`anyhow` 提供了一个统一的错误类型 `anyhow::Error`，用于表示任何类型的错误。这使得错误处理简单且一致。另外它也提供丰富的错误信息，允许我们在错误中包含额外的上下文信息，例如导致错误的函数和文件名。这会方便于错误调试。

# 总结

日常开发中，对于错误信息如果一起使用 `thiserror` 和 `anyhow` 这两个 crate，开发效率将大大提升，这也是多开发者的首先组合。

其中 `thiserror` 主要用来实现对错误类型的定义，而   `anyhow` 则主要有来对错误进行封装并进行错误传递，特别适合于不同函数之间的调用，屏蔽了不同类型的错误信息。

对于这两个错误处理的crate，这里只是介绍了最基本的用法 ，更多用法可参考官方文档。



# 参考资料

- https://crates.io/crates/thiserror
- https://crates.io/crates/anyhow







