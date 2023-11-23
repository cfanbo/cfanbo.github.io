---
title: Rust 中的 Result 与 Option
date: 2023-11-07T11:50:32+08:00
type: post
toc: true
url: "/posts/rust-result-and-option"
categories:
- 程序开发
tags:
- rust
---

在 Rust 中有两个常用的 `enum` 枚举类型，分别为 `Result` 和 `Option`，本节介绍它们两者各自的使用场景和用法。

这里我们先给出结论

- 结果 `Result` 表示 `成功` 或 `失败`
- 选项 `Option` 表示 `有` 或者 `无`

当从本地读取一个文件时，这时候可能读取成功，也有可能由于文件不存在或权限不足导致读取时候，这种场景一般就需要使用 `Result`；而当从一组数据集中查询指定元素是否存在时，这时有可能存在，也有可能不存在（用None 表示)，这时情况就应该选择Option。

由此看到，这两个枚举类型的区别理解起来还是挺简单的，下面我们单独对每一种类型做一下详细的介绍。

# 结果 Result 

定义

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`Result<T, E>` 类型拥有两个取值：

- `Ok(value)` 表示操作成功，并包装操作返回的 `value`（`value` 拥有 `T` 泛类型）。
- `Err(why)`，表示操作失败，并包装 `why`，它（但愿）能够解释失败的原因（`why` 拥有 `E` 类型）。

举个例子，这里打开一个文件，如果文件存在则打印文件句柄信息，否则打印错误信息，查看 [playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=f11421eb85a986b552c5e1f3898d9e16)

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");  // 返回 Result<T, E>

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
    println!("{:?}", greeting_file);
}
```

这里`File::open` 的返回值是 `Result<T, E>`。泛型参数 `T` 会被 `File::open` 的实现放入成功返回值的类型 `std::fs::File`，这是一个文件句柄。错误返回值使用的 `E` 的类型是 `std::io::Error`。这些返回类型意味着 `File::open` 调用可能成功并返回一个可以读写的文件句柄。

对于结果 `Result<T, E>`我们一般是通过 `match` 来根据不同情况进行处理。

如果文件存在，则运行结果

```shell
   Compiling my_crate_demo v0.1.2 (/Users/sxf/workspace/rust/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.87s
     Running `target/debug/my_crate_demo`
File { fd: 3, path: "/Users/sxf/workspace/rust/hello/hello.txt", read: true, write: false }
```

这里打印了一些文件句柄相关信息。

当文件不存在，或者权限不足导致打开文件失败

```shell
Compiling my_crate_demo v0.1.2 (/Users/sxf/workspace/rust/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.52s
     Running `target/debug/my_crate_demo`
thread 'main' panicked at src/main.rs:8:23:
Problem opening the file: Os { code: 2, kind: NotFound, message: "No such file or directory" }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

可以看到打印的错误信息提示`No such file or directory`，告诉我们这里错误是由于文件未找到引起的，而非权限原因。

以上是系统内置的Result定义，我们也可以给其起一个自定义别名，如

```
type MyResult = Result<u32, String>;
```

另晚通过 `type` 关键字来声明，这里 `MyResult` 对应的正是 `Result<u32,String>` ，即成功时返回的是`u32`类型，失败时返回String错误字符串。

```rust
type MyResult = Result<u32, String>;

fn div(x: u32, y: u32) -> MyResult {
    if y == 0 {
        Err(String::from("error"))
    } else {
        Ok(x / y)
    }
}

fn main() {
    // ok
    let x = div(10, 2);
    match x {
        Ok(num) => {
            println!("val={}", num);
        }
        Err(err) => println!("{:?}", err),
    };

    // error
    let x = div(10, 0);
    match x {
        Ok(num) => {
            println!("val={}", num);
        }
        Err(err) => println!("{}", err),
    };
}
```

运行结果为

```shell
val=5
error
```

这里直接通过函数 `Ok(T)` 表示成功返回值，通过 `Err(E)` 表示失败返回值，这两个函数经常以后会经常的用到。



# 选项 Option

定义

```rust
pub enum Option<T> {
    None,
    Some(T),
}
```

它表现为以下两个 “option”（选项）中的一个：

- `Some(T)`：找到一个属于 `T` 类型的元素
- `None`：找不到相应元素

示例，声明一个Option类型变量，并读取打印其值

```rust
fn main() {
    // 定义一个 Some(T) 类型的Option, 变量类型默认后面赋值推理出来
    let num = Some(5);
    match num {
        Some(val) => println!("val={}", val),
        None => println!("None"),
    }

    // 定义一个 None 类型的 Option, 手动指定类型，否则 match 里的 Some(T) 条件则无法编译通过
    let n: Option<u32> = Option::None;
    match n {
        Some(val) => println!("val={}", val),
        None => println!("None"),
    }
}
```

这里我们分别针对Option枚举值 `Some(T)` 和 `None` 两类值，根据match条件进行了处理，

这里的 `Some(val)` 用法，表示匹配的值赋值给了变量`val`，因为我们需要将其打印出来。如果我们不需要这个值的话，则可以通过 `Some(_)` 的方法将其舍弃掉。

有一点需要注意，对于`match` 匹配是穷尽的，其分支必须覆盖了所有的可能性，如

```rust
fn main() {
    // 未考虑 None 的情况
    let num = Some(5);
    match num {
        Some(val) => println!("val={}", val),
        // None => println!("None"),
    }
}
```

此时编译出错

```shell
    Blocking waiting for file lock on build directory
   Compiling my_crate_demo v0.1.2 (/Users/sxf/workspace/rust/hello)
error[E0004]: non-exhaustive patterns: `None` not covered
   --> src/main.rs:4:11
    |
4   |     match num {
    |           ^^^ pattern `None` not covered
```

错误给出提示未覆盖到`None`的情况。

另外在满足match分支的情况下，还可以添加更多的匹配条件，如针对特定的值做特别的处理

```rust
fn main() {
    let num = Some(0);
    match num {
        Some(0) => println!("zero"),
        Some(val) => println!("val={}", val),
        None => println!("None"),
    }
}
```

这里变量 num 的值为 `Some(0)`，因此匹配到第一个分支，打印 `zero`; 如果将变量值改为`Some(2)`的话，则将匹配第二个分支，打印 `val=2`，同样如果将变量值改为  `Option::None`，则匹配到第三个分支，打印`None`。

除此之外，还可以匹配一段区间值，如

```rust
fn main() {
    let number = Some(5);

    match number {
        Some(n) if (1..=9).contains(&n) => println!("The number is in the range 1 to 9"),
        Some(_) => println!("The number is not in the range 1 to 9"),
        None => println!("There is no number"),
    }
}
```

此时，将匹配到第一个分支，打印 `The number is in the range 1 to 9`, 这里的 `if (1..=9)` 后面的等号表示包含数字9。如果变量number大于5，则匹配第二个条件，这里我们将匹配的值通过 Some(_) 的分支舍弃。

以上是针对 Option类型 `match` 的几种常见用法， 更多 用法参考 https://kaisery.github.io/trpl-zh-cn/ch06-02-match.html



# 总结

针对枚举类型 `Result` 和 `Option` 的处理，需要用到 `match` 控制流运算符, 其用法也是比较的灵活。使用过程中我们只需要记住一点，对于 match 的匹配是无穷尽的，需要保证考虑到所有的匹配情况。



#  参考资料

- https://rustwiki.org/zh-CN/rust-by-example/std/result.html
- https://rustwiki.org/zh-CN/rust-by-example/std/option.html
- https://kaisery.github.io/trpl-zh-cn/ch06-02-match.html
- https://kaisery.github.io/trpl-zh-cn/ch09-02-recoverable-errors-with-result.html
