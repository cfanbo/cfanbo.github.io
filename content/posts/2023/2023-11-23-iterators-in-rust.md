---
title: Rust中的迭代器iter
type: post
toc: true
date: 2023-11-16T07:31:19+00:00
url: /posts/iterators-in-rust
categories:
- 程序开发
tags:
- rust
---

迭代器模式允许你对一个序列的项进行某些处理。**迭代器**（*iterator*）负责遍历序列中的每一项和决定序列何时结束的逻辑。当使用迭代器时，我们无需重新实现这些逻辑。

在 Rust 中，迭代器是 **惰性的**（*lazy*），这意味着在调用方法使用迭代器之前它都不会有效果。例如，示例中的代码通过调用定义于 `Vec` 上的 `iter` 方法在一个 vector `v1` 上创建了一个迭代器。这段代码本身没有任何用处：

```rust
let v1 = vec![1, 2, 3];
let v1_iter = v1.iter();
```

迭代器被储存在 `v1_iter` 变量中。一旦创建迭代器之后，可以选择用多种方式利用它。

# 迭代器分类

Rust 中迭代器根据 `所有权` 可分为 `iter()`、`iter_mut()`、`into_iter()` 三种迭代器，使用场景：

- 获取集合元素不可变引用的迭代器，对应方法为 `iter()`

- 获取集合元素可变引用的迭代器，对应方法为 `iter_mut()`
- 获取集合元素所有权的迭代器，对应方法为 `into_iter()`

也就是说当你在 Rust 中看到调用了 `iter()` 方法，则表示这里使用了不可变迭代器，只能读取元素值，无法修改；如果看到 `iter_mut()` 则表示使用了可变迭代器，您可以对原始值进行修改；而如果看到 `into_iter()` 的话，则说明使用了集合元素的所有权引用，当迭代器使用完毕后，就对应的内存将自动根据所有权规则被释放回收。

# 不可变引用迭代器 iter()

```rust
fn main() {
    let a = [1, 2, 3];

    // 不可变引用迭代器
    let mut iter = a.iter();
    println!("{:?}", iter.len()); // 当前集合元素个数

    // A call to next() returns the next value...
    assert_eq!(Some(&1), iter.next());
    println!("{:?}", iter.len());

    assert_eq!(Some(&2), iter.next());
    println!("{:?}", iter.len());

    assert_eq!(Some(&3), iter.next());
    println!("{:?}", iter.len());

    // 所有值已迭代完毕，从此以后再调用就是 None
    assert_eq!(None, iter.next());

    // More calls may or may not return `None`. Here, they always will.
    assert_eq!(None, iter.next());
    assert_eq!(None, iter.next());

    // 上面使用了 iter() 获取不可变迭代器，因此可以打印原始变量值
    println!("len={}", iter.len());
    println!("{:?}", iter);
    println!("{:?}", a);
}
```

首先声明一个包含三个元素的数组，然后使用 `iter()` 获取不可变引用迭代器，也就是这种迭代器并不影响原来变量 `a` 的值，这时迭代器元素个数为 `3`。接着调用三次 `iter.next()` 读取出下一个元素，并在每次调用后面打印其元素个数，可以看到其个数为递减，走到为 `0` 为止。

>  这里的 `next` 方法的方法被称为 **消费适配器**（*consuming adaptors*），因为调用它们会消耗迭代器

从第四调用 `iter.next()` 开始，后面再调用 `iter.next()` 迭代器则将为 `None`, 同时元素个数将一直为 `0`，执行程序最后输出结果

```shell
3
2
1
0
len=0
Iter([])
[1, 2, 3]
```

方法`iter.next()` 返回的值为引用，因此断言也使用值引用的方式判等。



# 可变迭引用代器 iter_mut()

迭代器的读取示例

```rust
fn main() {
    let mut a = ["1".to_string(), "2".to_string(), "3".to_string()];

    let mut an_iter = a.iter_mut();
    println!("len={}", an_iter.len());

    assert_eq!(Some(&mut "1".to_string()), an_iter.next());
    println!("len={}", an_iter.len());

    assert_eq!(Some(&mut "2".to_string()), an_iter.next());
    println!("len={}", an_iter.len());

    assert_eq!(Some(&mut "3".to_string()), an_iter.next());
    println!("len={}", an_iter.len());

    assert_eq!(None, an_iter.next());
    println!("len={}", an_iter.len());

    println!("{:?}", an_iter);
    println!("{:?}", a);
}
```

执行结果

```shell
3
2
1
0
len=0
Iter([])
[1, 2, 3]
```

可以看到对可变引用迭代器元素读取操作，它与不可变迭代器 `iter()` 没有什么不一样；接着我们再看一个对值进行修改的示例

```rust
fn main() {
    let x = &mut [1, 2, 4];
    for elem in x.iter_mut() {
        *elem += 2;
    }
    assert_eq!(x, &[3, 4, 6]);
}
```

这里我们通过 `iter_mut()` 来实现一个可变迭代器，接着依次修改每个元素的值，从而达到修改原始变量 `x` 内容的效果，只所有能修改，正是因为它们是对同一个值的引用。

# 所有权的迭代器 into_iter()

```rust
fn main() {
    let a = ["1".to_string(), "2".to_string(), "3".to_string()];

    let mut an_iter = a.into_iter(); // 从此以后数组 a 已经被消耗掉

    assert_eq!(Some("1".to_string()), an_iter.next());
    assert_eq!(Some("2".to_string()), an_iter.next());
    assert_eq!(Some("3".to_string()), an_iter.next());
    assert_eq!(None, an_iter.next());

    println!("{:?}", a); // 出错, 调用 into_iter() 导致a所有权被转移
}
```

编译出错

```shell
error[E0382]: borrow of moved value: `a`
   --> src/main.rs:11:22
    |
2   |     let a = ["1".to_string(), "2".to_string(), "3".to_string()];
    |         - move occurs because `a` has type `[String; 3]`, which does not implement the `Copy` trait
3   |
4   |     let mut an_iter = a.into_iter(); // 从此以后数组 a 已经被消耗掉
    |                         ----------- `a` moved due to this method call
...
11  |     println!("{:?}", a); // 出错
    |                      ^ value borrowed here after move
    |
note: `into_iter` takes ownership of the receiver `self`, which moves `a`
   --> /Users/sxf/.rustup/toolchains/stable-x86_64-apple-darwin/lib/rustlib/src/rust/library/core/src/iter/traits/collect.rs:267:18
    |
267 |     fn into_iter(self) -> Self::IntoIter;
    |                  ^^^^
    = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)
help: you can `clone` the value and consume it, but this might not be your desired behavior
    |
4   |     let mut an_iter = a.clone().into_iter(); // 从此以后数组 a 已经被消耗掉
    |                        ++++++++

For more information about this error, try `rustc --explain E0382`.
error: could not compile `my_crate_demo` (bin "my_crate_demo") due to previous error
```

根据所有权的转移概念，可以得知当调用  `a.into_iter()` 时，所有权发生了转移，因此最后一行打印变量 `a`的语句将在编译时出错，上面的错误信息证实了这一点。

上面的示例是针对字符串来讲的，对于整数数组 `[1,2,3]` 而言，调用 `into_iter()` 实际上会将这个数组复制一份，再将复制后的数组转换成迭代器，并消耗掉这个复制后的数组，因此最后的打印语句能把原来那个 a 打印出来。

```rust
fn main() {
    let a = [1, 2, 3];

    let mut an_iter = a.into_iter(); // 从此以后数组 a 已经被消耗掉

    assert_eq!(Some(1), an_iter.next());
    assert_eq!(Some(2), an_iter.next());
    assert_eq!(Some(3), an_iter.next());
    assert_eq!(None, an_iter.next());

    println!("{:?}", a); // 正常工作
}
```

# 总结

如果只是对集合中的值进行一些读取的话，则直接使用不可变引用迭代器 `iter()` 即可；而如果对对原来集合的值进行修改的话，则需要使用可变迭代器 `iter_mut() `；而如果需要获取原来集合的所有权的话，则可以使用所有权迭代器 `into_iter()`



# 参考资料

- [什么是所有权?](https://kaisery.github.io/trpl-zh-cn/ch04-01-what-is-ownership.html)
- [引用与借用](https://kaisery.github.io/trpl-zh-cn/ch04-02-references-and-borrowing.html#引用与借用)
- [迭代器](https://kaisery.github.io/trpl-zh-cn/ch13-02-iterators.html)
