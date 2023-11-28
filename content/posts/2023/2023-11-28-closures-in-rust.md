---
title: Rust中与闭包相关的三个trait
date: 2023-11-28T11:08:36+08:00
toc: true
type: post
url: /posts/closures-in-rust
categories:
- 程序开发
tags:
- rust
- trait
- 闭包
---

在 Rust 中，闭包就是一种能捕获 `上下文环境变量` 的函数。

```rust
let range = 0..10;
let get_range_count = || range.count();  
```

代码里的这个 `get_range_count` 就是闭包，range 是被这个闭包捕获的环境变量。

虽然说它是一种函数，但是不通过 `fn` 进行定义。**在 Rust 中，并不把这个闭包的类型处理成 fn 这种函数指针类型，而是有单独的类型定义。**

切记这里是将闭包处理成是 `单独的类型定义`，这一点区别与其它开发语言。

至于按哪一种类型来处理，这个没有办法得知，因为只有在Rust编译器在编译的时候才可以确定其类型，并且在确定类型时，还需要根据这个闭包捕获上下文环境变量时的行为来确定。

根据闭包行为划分为三类trait（ 主因是受到所有权影响）：

1. `FnOnce` 适用于能被调用一次的闭包，`所有闭包`都至少实现了这个 trait，因为所有闭包都必须能够被调用。一个会将捕获的值移出闭包体的闭包只实现 `FnOnce` trait，这是因为它只能被调用一次。其获取了上下文环境变量的所有权。
2. `FnMut` 适用于不会将捕获的值移出闭包体的闭包，但它可能会修改被捕获的值，这类闭包可以被调用多次。其只获取了上下文环境变量的 &mut 引用。
3. `Fn` 适用于既不将被捕获的值移出闭包体也不修改被捕获的值的闭包，当然也包括不从环境中捕获值的闭包。这类闭包可以被调用多次而不改变它们的环境，这在会 `多次并发调用闭包`的场景中十分重要。其只获取了上下文环境变量的 & 不可变引用。

在编译时会将闭包确定为以上三种类型中的一种或多种组合。 Rust 给我们暴露了 `FnOnce`、`FnMut`、`Fn` 这 3 个 trait，就刚对应上面这三类数据类型。

它们在标准库中的定义为

```rust
trait FnOnce<Args> {
    type Output;
    fn call_once(self, args: Args) -> Self::Output;
}
trait FnMut<Args>: FnOnce<Args> {
    fn call_mut(&mut self, args: Args) -> Self::Output;
}
trait Fn<Args>: FnMut<Args> {
    fn call(&self, args: Args) -> Self::Output;
}
```



# 环境变量所有权 FnOnce

FnOnce 代表的闭包类型只能被调用一次。

```rust
fn main() {
    let range = 0..10;
    let get_range_count = || range.count();
    assert_eq!(get_range_count(), 10); // ✅
    get_range_count(); // ❌
}
```

编译发现报错

```shell
  Compiling playground v0.0.1 (/playground)
error[E0382]: use of moved value: `get_range_count`
 --> src/main.rs:5:5
  |
4 |     assert_eq!(get_range_count(), 10); // ✅
  |                ----------------- `get_range_count` moved due to this call
5 |     get_range_count(); // ❌
  |     ^^^^^^^^^^^^^^^ value used here after move
  |
note: closure cannot be invoked more than once because it moves the variable `range` out of its environment
 --> src/main.rs:3:30
  |
3 |     let get_range_count = || range.count();
  |                              ^^^^^
note: this value implements `FnOnce`, which causes it to be moved when called
 --> src/main.rs:4:16
  |
4 |     assert_eq!(get_range_count(), 10); // ✅
  |                ^^^^^^^^^^^^^^^

For more information about this error, try `rustc --explain E0382`.
error: could not compile `playground` (bin "playground") due to previous error
```

在第一次调用这个函数时， `range` 变量的所有权已经交给了闭包函数，因此第二次调用闭包函数时会编译器发现变量所有权被移走而导致的错误。

同时错误信息里给出闭包实现了 `FnOnce` 这个 `trait`，也就是说这个闭包函数只能调用一次。



# 环境变量可变引用 FnMut

FnMut 代表的闭包类型能被调用多次，并且能修改上下文环境变量的值，不过有一些副作用，在某些情况下可能会导致错误或者不可预测的行为。

```rust
fn main() {
    let nums = vec![0, 4, 2, 8, 10, 7, 15, 18, 13];
    let mut min = i32::MIN;
    let ascending = nums.into_iter().filter(|&n| {
        if n <= min {
            false
        } else {
            min = n;  // 这里修改了环境变量min的值
            true
        }
    }).collect::<Vec<_>>();
    assert_eq!(vec![0, 4, 8, 10, 15, 18], ascending); // ✅
}
```

这个函数实现递增的整数向量闭包，这里使用了 `into_iter` 函数，表示获取迭代器元素的所有权，在上一节 https://blog.haohtml.com/posts/iterators-in-rust/ 我们对其用法进行了介绍。`|&n|` 代表闭包接受一个引用作为参数，而 `min` 作为一个可变变量（使用了 `mut` 关键字）可以在闭包内进行修改，程序执行完后，此时 `min` 变量值为 `18`。

# 环境变量不可变引用  Fn

Fn 代表的这类闭包能被调用多次，但是对上下文环境变量没有副作用。

```rust
fn main() {
    let nums = vec![0, 4, 2, 8, 10, 7, 15, 18, 13];
    let min = 9;
    let greater_than_9 = nums.into_iter().filter(|&n| n > min).collect::<Vec<_>>();
    assert_eq!(vec![10, 15, 18, 13], greater_than_9); // ✅
}
```

闭包过滤出所有大于min的元素值，在filter 函数里并没有对环境变量 `min` 做任何的修改。

# 说明

根据 Rust 的设计，trait 之间存在层次结构。

![3种闭包维恩图](https://blogstatic.haohtml.com//uploads/2023/09/image-20231128124333557.png)

如果一个闭包满足 `Fn` 这个trait 的条件，那么它也一定满足 `FnMut` 和 `FnOnce` 这两个trait 的条件。如果满足 `FnMut` 这个trait的话，则一定满足 `FnOnce` trait 的条件，因此一个实现了 `Fn` 的闭包实际上是实现了 `Fn` 、`FnMut` 和 `FnOnce` 这三个trait。

由此可看出上面的不可用引用  `Fn` 例子是实现了三个trait的组合的，同样下方的示例也实现了三个trait。

```rust
fn main() {
    let x = 4;
    let f = ||{
        println!("{}", x);
    };
    f();
    f();
}
```

这里闭包满足了 `Fn` 这个trait，根据上面层次结构定义，它同样满足了 `FnOnce`  和 `FnMut` 这三个trait 条件。



# 参考资料

- https://kaisery.github.io/trpl-zh-cn/ch13-01-closures.html
- https://time.geekbang.org/column/article/724942
