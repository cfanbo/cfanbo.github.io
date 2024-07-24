---
title: 在Rust中如何调用一个模块或方法
date: 2024-07-10T12:25:42+08:00
type: post
toc: true
url: /posts/how-to-call-a-module-in-rust
categories:
  - 程序开发
tags:
  - rust
  - mod
---

在 Rust 中有 `包`、`crate`、`模块` 概念，本文我们介绍一下它们之间的关系和调用方法。

# 包 和 Crate

在Rust中，*包*（*package*）是提供一系列功能的一个或者多个 crate。一个包会包含一个 `Cargo.toml` 文件，阐述如何去构建这些 crate。

我们先看一下通过 `cargo new` 创建一个 `my_project` 包。

```shell
➜  cargo new my_project   
    Creating binary (application) `my_project` package
note: see more `Cargo.toml` keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

➜  rust tree my_project 
my_project
├── Cargo.toml
└── src
    └── main.rs

2 directories, 2 files
```

它将创建一个 `Cargo.toml`文件，内容：

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"

[dependencies]
```

文件里并没有提到 `src/main.rs`，因为 Cargo 遵循的一个约定：*src/main.rs* 就是一个与包同名的二进制 crate 的 crate 根。同样的，Cargo 知道如果包目录中包含 *src/lib.rs*，则包带有与其同名的库 crate，且 *src/lib.rs* 是 crate 根。crate 根文件将由 Cargo 传递给 `rustc` 来实际构建库或者二进制项目。

在Rust中crate有两种形式，分别为 二进制crate 和 库 crate。二进制crate一般是在 `main.rs`，它有一个 main() 入口函数，可以编译成一个二进制文件，而对于 库 crate，它一般定义在 `lib.rs` 文件，它没有`main()`函数，也不会编译成可执行程序，它一般提供一些诸如函数之类的东西，然后作为库被第三方调用。

在一个包中对 crate的要求有以下两点：

- 一个包至少包含一个crate，它可以是二进制crate，也可以是库crate
- 可包含多个二进制crate，但最多只能包含一个库crate(library crate)

如果在一个项目中，我们有一个只包含 *src/main.rs* 的包，意味着它只含有一个名为 `my-project` 的二进制 crate。如果一个包同时含有 *src/main.rs* 和 *src/lib.rs*，则它有两个 crate：一个二进制的和一个库的，**且名字都与包相同**。

通过将文件放在 *src/bin* 目录下，一个包可以拥有多个二进制 crate：每个 *src/bin* 下的文件都会被编译成一个独立的二进制 crate。

# 模块

官方文档提供一个简单的参考，用来解释模块、路径、`use`关键词和`pub`关键词如何在编译器中工作，以及大部分开发者如何组织他们的代码。我们将在本章节中举例说明每条规则，不过这是一个解释模块工作方式的良好参考。

- **从 crate 根节点开始**: 当编译一个 crate, 编译器首先在 crate 根文件（通常，对于一个库 crate 而言是*src/lib.rs*，对于一个二进制 crate 而言是*src/main.rs*）中寻找需要被编译的代码。

- 声明模块: 在 crate 根文件中，你可以声明一个新模块；比如，你用

  ```rust
  mod garden;
  ```

  声明了一个叫做 `garden`的模块。编译器会在下列路径中寻找模块代码：

  - 内联，在大括号中，当`mod garden`后方不是一个分号而是一个大括号
  - 在文件 *src/garden.rs*
  - 在文件 *src/garden/mod.rs*

- 声明子模块: 在除了 crate 根节点以外的其他文件中，你可以定义子模块。比如，你可能在`src/garden.rs`中定义了

  ```rust
  mod vegetables;
  ```

  。编译器会在以父模块命名的目录中寻找子模块代码：

  - 内联，在大括号中，当`mod vegetables`后方不是一个分号而是一个大括号
  - 在文件 *src/garden/vegetables.rs*
  - 在文件 *src/garden/vegetables/mod.rs*

- **模块中的代码路径**: 一旦一个模块是你 crate 的一部分，你可以在隐私规则允许的前提下，从同一个 crate 内的任意地方，通过代码路径引用该模块的代码。举例而言，一个 garden vegetables 模块下的`Asparagus`类型可以在`crate::garden::vegetables::Asparagus`被找到。

- **私有 vs 公用**: 一个模块里的代码默认对其父模块私有。为了使一个模块公用，应当在声明时使用`pub mod`替代`mod`。为了使一个公用模块内部的成员公用，应当在声明前使用`pub`。

- **`use` 关键字**: 在一个作用域内，`use`关键字创建了一个成员的快捷方式，用来减少长路径的重复。在任何可以引用`crate::garden::vegetables::Asparagus`的作用域，你可以通过 `use crate::garden::vegetables::Asparagus;`创建一个快捷方式，然后你就可以在作用域中只写`Asparagus`来使用该类型。

官方提供的示例（https://kaisery.github.io/trpl-zh-cn/ch07-02-defining-modules-to-control-scope-and-privacy.html）

```shell
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

这个 backyard 包只有一个二进制crate，不包含库crate。其中 main.rs 调用代码

```rust
use crate::garden::vegetables::Asparagus; // 引入作用域

pub mod garden; // 声明模块，同crate

fn main() {
    let plant = Asparagus {};
    println!("I'm growing {plant:?}!");
}
```

其中 `pub mod garden` 用来声明当前二进制crate中的模块 garden，告诉编译器应该包含在*src/garden.rs*文件中发现的代码，接着就可以通过 `use` 调用模块内容了。

`src/garden.rs`

```rust
pub mod vegetables;
```

`src/garden/vegetables.rs`

```rust
#[derive(Debug)]
pub struct Asparagus {}
```



# 调用

如果我们想在 main.rs 里调用一个模块方法或函数，一般有两种方法。

一种方法是将这个函数放在为crate里，然后在二进制crate调用，这种属于跨crate调用的情况，这种方法的好处是维护方面，对于 main.rs 文件我们只需要维护最简单的情况即可。另外，作为库crate还可以被多个二进制crate复用，因此这是目前比较流行和推荐的用法 。

另外一种方法是直接将函数写在二进制crate里调用，这属于crate内部之间的调用，一般用在比较小的项目中，一般代码量比较少，维护成本低的情况下采用这种方法。

下面是项目my_project两种不同的调用方法。

## 1. 跨 crate 调用（推荐）

这里指在二进制crate 里调用 库crate的情况。

将 `util` 模块定义在 `src/lib.rs` 中，它就是库 crate 的一部分。通常库 crate 的模块定义和实现都放在 `src/lib.rs` 中。以下是一个示例：

项目结构：

```shell
my_project
├── Cargo.toml # 项目名称定义在这个文件 my_project
├── src
│   ├── lib.rs
│   ├── main.rs
│   └── util.rs
```

`src/lib.rs`:

```rust
pub mod util;
```

`src/util.rs`:

```rust
pub struct Network;

impl Network {
    pub fn new() -> Self {
        Network
    }

    pub fn show(&self) {
        println!("Network::show() called");
    }
}
```

在 Rust中对struct的定义和实现是**分开**的，这里声明了一个空的结构体 `Network`，接着通过 `impl` 来实现具体的逻辑功能。

在 Rust 中，声明结构体（struct）时，如果结构体没有任何字段，是可以省略花括号 `{}` 的。这种结构体称为**单元结构体**（unit struct）。

`src/main.rs`:

```rust
use my_project::util::Network; 

fn main() {
    let net = Network::new();
    net.show();
}
```

先通过 `use` 将相应的结构体引入，其中 `my_project` 是指包名，其在 `Cargo.toml`文件中通过 `[package].name` 定义。

## 2. 相同 crate 内部调用

这里指调用当前crate里的模块，即可以库crate内部的相互调用，也可以是二进制crate内部的相互调用。

将 `util` 模块直接定义在 `src/main.rs` 中，或者通过 `mod` 关键字在 `main.rs` 中引入，它是二进制 crate 的一部分。

项目结构：

```rust
my_project
├── Cargo.toml
├── src
│   ├── main.rs
│   └── util.rs
```

`src/main.rs`:

```rust
mod util; // 声明模块

use util::Network; // 通过 use 引入作用域

fn main() {
    let net = Network::new();
    net.show();
}
```

`src/util.rs`:

```rust
pub struct Network;

impl Network {
    pub fn new() -> Self {
        Network
    }

    pub fn show(&self) {
        println!("Network::show() called");
    }
}
```

## 结论

在这两种情况下，`util` 模块都可以正常工作，具体取决于您希望它属于库 crate 还是二进制 crate。如果您希望模块在多个二进制 crate 中复用，最好将它放在库 crate 中。如果它仅用于单个二进制 crate，直接在 `main.rs` 中定义或引入即可。

# 参考

-  [定义模块来控制作用域与私有性](https://kaisery.github.io/trpl-zh-cn/ch07-02-defining-modules-to-control-scope-and-privacy.html#定义模块来控制作用域与私有性)

- [引用模块项目的路径](https://kaisery.github.io/trpl-zh-cn/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#引用模块项目的路径)

- [使用 `use` 关键字将路径引入作用域](https://kaisery.github.io/trpl-zh-cn/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#使用-use-关键字将路径引入作用域)

-  [将模块拆分成多个文件](https://kaisery.github.io/trpl-zh-cn/ch07-05-separating-modules-into-different-files.html#将模块拆分成多个文件)
