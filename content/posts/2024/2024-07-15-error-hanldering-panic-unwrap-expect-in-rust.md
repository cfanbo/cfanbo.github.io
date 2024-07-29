---
title: Rust 中常见的几种错误处理方法
date: 2024-07-15T13:34:41+08:00
type: post
toc: true
url: /posts/error-hanlding-unwrap-and-expect-in-rust
categories:
  - 程序开发
tags:
  - rust
---



Rust 中错误可分为两大类：**可恢复的**（*recoverable*）和 **不可恢复的**（*unrecoverable*）错误。

对于一个可恢复的错误，比如文件未找到或权限不足的错误，我们很可能只想向用户报告问题，让用户来决定后续操作。

不可恢复的错误总是 bug 出现的征兆，比如试图访问一个超过数组末端的位置，因此我们要立即停止程序，主要通过 `panic!` 实现。

# panic!

`panic!`属于不可恢复错误类型，一旦发生程序将立即退出。

```rust
fn main() {
    panic!("crash and burn");
}
```

这里由用户调用 `panic!` 宏来实现程序的中止，同时自定义错误信息。

```shell
➜  cargo run
   Compiling hello-world v0.1.0 (/Users/sxf/workspace/rust/hello-world)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.68s
     Running `target/debug/hello-world`
thread 'main' panicked at src/main.rs:2:5:
crash and burn
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

切记：一旦调用 `panic! `，程序将立即中止运行，且无法恢复。

至于什么时候使用 `panic!` ，可参考文档：[要不要 `panic!`](https://kaisery.github.io/trpl-zh-cn/ch09-03-to-panic-or-not-to-panic.html#要不要-panic)



# 系统默认错误 unwrap

unwrap 底层调用的 panic!，因此也属于不可恢复错误。

调用 `unwrap()` 获取 Result 中 `Ok(value)` 的value，如果 Result 的值为 Err，则直接 `panic!()` 并中止程序。

```rust
use std::fs::File;

fn main() {
    // 尝试打开一个文件，如果失败则触发 panic 并输出自定义错误信息
    let _file = File::open("non_existent_file.txt").unwrap();
    println!("hello")
}

```

执行结果

```shell
➜  cargo run
   Compiling hello-world v0.1.0 (/Users/sxf/workspace/rust/hello-world)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 1.24s
     Running `target/debug/hello-world`
thread 'main' panicked at src/main.rs:5:53:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

调用 `Result::uwrap()` 函数获取 Result 中的 Ok() 的值，将根据不同情况进行处理

1. 当 Result 为 `Ok(value)` 时，则直接返回 `value`
2. 当 Result 为 `Err` 时，则内部将调用  `panic!` 函数并中止程序运行，因此调用 `unwrap()` 时，最好可以保证不会发生err，否则程序将被中止

> 由于调用了panic! ，因此 hello 没有打印出来

对于 Result 为 Err 时，我们还可以调用 `unwrap_or_else()` 来自定义匿名函数调用

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let _greeting_file = File::open("non_existent_file.txt").unwrap_or_else(|error| {
      	// 如果 Result 为 Err
        if error.kind() == ErrorKind::NotFound {
            File::create("non_existent_file.txt").unwrap_or_else(|error| {
              	// 如果 Result 为 Err, 通过 panic!() 宏自定义错误并中止程序运行
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```





# 自定义错误 expect

同 `unwrap() ` 一样，底层同样调用 `panic!`，因此同样属于不可恢复错误，相比 `unwrap()`可实现错误信息自定义。

表示当错误发生时，不使用系统默认的错误信息，而是自定义错误信息，可用在 `Result` 和 `Option` 两种错误类型。

**使用 Result**

 ```rust
 use std::fs::File;
 
 fn main() {
     // 尝试打开一个文件，如果失败则触发 panic 并输出自定义错误信息
     let file = File::open("non_existent_file.txt").expect("无法打开文件，文件不存在或无法访问");
 }
 ```

执行结果

```shell
➜  cargo run
	Compiling hello-world v0.1.0 (/Users/sxf/workspace/rust/hello-world)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.44s
     Running `target/debug/hello-world`
thread 'main' panicked at src/main.rs:5:53:
无法打开文件，文件不存在或无法访问: Os { code: 2, kind: NotFound, message: "No such file or directory" }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

这里打印出指定的错误信息，同时还打印了错误类型信息。 `Os { code: 2, kind: NotFound, message: "No such file or directory" }`

**使用 Option**

```rust
fn main() {
    let some_option: Option<i32> = None;

    // 尝试解构 Option，如果为 None 则触发 panic 并输出自定义错误信息
    let _value = some_option.expect("未找到值：Option 为 None");
}

```

执行结果

```shell
➜  cargo run
	Compiling hello-world v0.1.0 (/Users/sxf/workspace/rust/hello-world)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.61s
     Running `target/debug/hello-world`
thread 'main' panicked at src/main.rs:5:30:
未找到值：Option 为 None
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

直接打印出了自定义的错误信息。

有时候我们可能还需要同时打印系统的默认错误信息

```rust
use std::fs::File;

fn main() {
    // 使用 expect 方法结合系统错误信息与自定义错误信息
    let _file = File::open("non_existent_file.txt")
        .expect(&format!("无法打开文件：文件不存在或无法访问，错误信息：{}", std::io::Error::last_os_error()));
}

```

这里通过 `std::io::Error::last_os_error())` 来实现最后的系统错误信息。

执行结果

```shell
➜  cargo run
   Compiling hello-world v0.1.0 (/Users/sxf/workspace/rust/hello-world)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 1.30s
     Running `target/debug/hello-world`
thread 'main' panicked at src/main.rs:5:53:
无法打开文件：文件不存在或无法访问，错误信息：No such file or directory (os error 2): Os { code: 2, kind: NotFound, message: "No such file or directory" }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

另外一种方法是通过  `unwrap_or_else()`

```rust
use std::fs::File;

fn main() {
    // 使用 unwrap_or_else 结合系统错误信息与自定义错误信息
    let _file = File::open("non_existent_file.txt")
        .unwrap_or_else(|err| panic!("无法打开文件：文件不存在或无法访问，错误信息：{}", err));
}

```

执行结果

```shell
➜  cargo run
   Compiling hello-world v0.1.0 (/Users/sxf/workspace/rust/hello-world)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 1.19s
     Running `target/debug/hello-world`
thread 'main' panicked at src/main.rs:6:31:
无法打开文件：文件不存在或无法访问，错误信息：No such file or directory (os error 2)
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

如果仔细观察的话，会发现 `unwrap_or_else()` 这种方法并没有打印 错误类型 `Os { code: 2, kind: NotFound, message: "No such file or directory" }` 这一部分。



# 错误传播 ？

对于上面几种属于不可恢复的错误，而对于错误传播则属于可恢复的错误。

有时候我们需要将一些错误信息向上层传播，对于错误的处理在上层逻辑中实现，如我们平时常见的函数用法。

传统写法：

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}

```

这里有两个地方可能出错，分别对每一次可能出错的情况都分别进行了单独的处理，我们再看下另一种简单的方法

```rust
#![allow(unused)]
fn main() {
  use std::fs::File;
  use std::io::{self, Read};

  fn read_username_from_file() -> Result<String, io::Error> {
      let mut username_file = File::open("hello.txt")?;
      let mut username = String::new();
      username_file.read_to_string(&mut username)?;
      Ok(username)
  }
}
```

这里函数里只对Result为 `Ok()` 的情况做了处理，而对于 Err 的情况，直接返回函数，交给调用层来处理，因此代码量相对要少许多。

这里通过 `?` 实现了函数的错误向上传播，函数执行过程为

1. 当 `File::open()`失败的时候，通过 `?` 将错误直接return，函数结束
2. 当 `read_to_string()`失败的时候，同样通过 `?` 错误 return，函数结果
3. 如果前面都执行成功的话，则直接调用  Ok() 函数，同样函数结束

可以看出`?`  的作用类似于

```rust
   	let ok_value = match Result {
        Ok(value) => value,
        Err(e) => return Err(e),
    };
```

这里推荐对于一些常见错误处理通过函数进行封装，然后在调用层做错误情况的处理。

# 总结

1. panic! 一旦被调用，程序将立即中止，且不可恢复。

2. 对于 `unwrap()` 和 `expect()` 两者底层调用的都是 `panic!`，expect  相比 unwrap() 来说，实现了自定义错误信息。另外也可以通过调用  `unwrap_or_else` 实现更复杂的逻辑处理。
3. 它们只有在 Result 为 Err 时才会调用 `panic!` 中止程序的执行
4. 如果想通过 `?` 向上传播错误，则不允许调用 unwrap() 和 expect(), 因为它们底层会调用 `panic!`，这会导致程序立即中止且无法恢复！



# 参考文档

- https://kaisery.github.io/trpl-zh-cn/ch09-00-error-handling.html
- [Rust 中的 Result 与 Option](https://blog.haohtml.com/posts/rust-result-and-option/)