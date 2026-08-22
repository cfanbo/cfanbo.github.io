---
title: Rust 中的迭代器：iter、iter_mut 与 into_iter
type: post
toc: true
date: 2023-11-16T07:31:19+00:00
url: /posts/iterators-in-rust
categories:
- 程序开发
tags:
- rust
---

迭代器模式允许我们对一个序列中的元素进行某些处理。**迭代器**（*iterator*）负责遍历序列中的每一项，并决定序列何时结束。当使用迭代器时，我们无需重新实现这些遍历逻辑。

# 什么是 Iterator？

在 Rust 中，**迭代器（Iterator）**是一种用于逐个访问一组元素的对象。它负责记录当前迭代的位置，并通过不断获取下一个元素来完成整个序列的遍历。

Rust 中的迭代器是**惰性的（lazy）**。也就是说，创建一个迭代器并不会立即执行遍历操作，只有在真正使用迭代器时，才会逐个获取元素。

例如：

```rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();
```

这里通过 `iter()` 创建了一个迭代器，并将其保存到 `v1_iter` 中。但是这一步并不会立即访问 `v1` 中的元素。

要真正获取元素，可以调用迭代器的 `next()` 方法：

```rust
let v1 = vec![1, 2, 3];

let mut v1_iter = v1.iter();

println!("{:?}", v1_iter.next()); // Some(&1)
println!("{:?}", v1_iter.next()); // Some(&2)
println!("{:?}", v1_iter.next()); // Some(&3)
println!("{:?}", v1_iter.next()); // None
```

Rust 中的 `Iterator` trait 定义了迭代器的基本行为，其中最核心的方法就是 `next()`：

```rust
trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

其中：

* `Item` 表示迭代器每次产生的元素类型
* `next()` 用于获取下一个元素
* 当还有元素时返回 `Some(value)`
* 当所有元素都遍历完成后返回 `None`

因此，只要一个类型实现了 `Iterator` trait，它就可以作为迭代器使用。

例如：

```rust
struct Counter {
    current: usize,
    max_limit: usize,
}

impl Iterator for Counter {
    type Item = usize;

    fn next(&mut self) -> Option<Self::Item> {
        if self.current < self.max_limit {
            let current = self.current;
            self.current += 1;
            Some(current)
        } else {
            None
        }
    }
}
```

这里 `Counter` 实现了 `Iterator` trait，因此 `Counter` 本身就是一个迭代器。

使用时可以直接：

```rust
let counter = Counter {
    current: 0,
    max_limit: 5,
};

for value in counter {
    println!("{}", value);
}
```

这里 `for value in counter{}` 可以理解为使用 `counter.into_iter()` 获取迭代器后进行遍历，即：

 ```rust
 for value in counter.into_iter() {
     println!("{}", value);
 }
 ```

输出：

```text
0
1
2
3
4
```

所以，从 Rust 的类型系统来看，**迭代器并不是某一种固定的数据结构，而是实现了 `Iterator` trait 的类型。**

# `iter()`、`iter_mut()` 和 `into_iter()` 是什么？

在 Rust 中，集合类型通常会提供多种获取迭代器的方式，其中最常见的就是：

```rust
iter()
iter_mut()
into_iter()
```

它们的主要区别在于**迭代过程中如何处理集合元素的所有权和借用关系**。

可以简单概括为：

| 方法            | 迭代器产生的元素 | 集合是否被消费 |
| ------------- | -------- | ------- |
| `iter()`      | `&T`     | 否       |
| `iter_mut()`  | `&mut T` | 否       |
| `into_iter()` | `T`      | 是       |

例如：

```rust
let mut v = vec![1, 2, 3];
```

使用 `iter()`：

```rust
for value in v.iter() {
    println!("{}", value);
}
```

这里 `value` 是 `&i32`。迭代器只是借用了集合中的元素，并不会取得元素的所有权，因此遍历完成后 `v` 仍然可以使用。

使用 `iter_mut()`：

```rust
for value in v.iter_mut() {
    *value += 1;
}
```

这里 `value` 是 `&mut i32`，因此可以通过这个可变引用修改 `v` 中的元素。

使用 `into_iter()`：

```rust
for value in v.into_iter() {
    println!("{}", value);
}
```

这里 `value` 是 `i32`。迭代器直接产生元素本身，并取得元素的所有权。同时，`v` 本身也会被消费，之后不能再使用原来的 `v`。

因此可以简单理解为：

```text
iter()
    ↓
不可变借用
    ↓
&T
```
```text
iter_mut()
    ↓
可变借用
    ↓
&mut T
```
```text
into_iter()
    ↓
取得元素所有权
    ↓
T
```

>需要注意的是，`iter()`、`iter_mut()` 和 `into_iter()` **并不是 `Iterator` trait 中定义的三个方法，也不是三种不同的 `Iterator` trait**。

它们只是获取迭代器的不同方式：

* `iter()` 和 `iter_mut()` 通常是集合类型提供的方法
* `into_iter()` 对应 `IntoIterator` trait，用于将一个对象转换为迭代器

因此，当我们看到：

```rust
v.iter()
```

可以理解为：

> 从 `v` 获取一个遍历其元素的迭代器，并以不可变引用的方式访问元素。

看到：

```rust
v.iter_mut()
```

可以理解为：

> 从 `v` 获取一个遍历其元素的迭代器，并以可变引用的方式访问元素。

看到：

```rust
v.into_iter()
```

则可以理解为：

> 消费 `v`，并获取一个能够逐个取得其元素所有权的迭代器。

接下来分别看三种方式的具体使用。

## 不可变引用迭代器 `iter()`

如果只是读取集合中的元素，而不需要修改元素，可以使用 `iter()`。

```rust
fn main() {
    let a = [1, 2, 3];

    let mut iter = a.iter();

    println!("{:?}", iter.len());

    assert_eq!(Some(&1), iter.next());
    println!("{:?}", iter.len());

    assert_eq!(Some(&2), iter.next());
    println!("{:?}", iter.len());

    assert_eq!(Some(&3), iter.next());
    println!("{:?}", iter.len());

    // 所有值已迭代完毕
    assert_eq!(None, iter.next());

    // 之后继续调用 next() 仍然返回 None
    assert_eq!(None, iter.next());
    assert_eq!(None, iter.next());

    println!("len={}", iter.len());
    println!("{:?}", iter);
    println!("{:?}", a);
}
```

执行结果：

```shell
3
2
1
0
len=0
Iter([])
[1, 2, 3]
```

这里通过 `iter()` 获取的迭代器，其元素类型为 `&i32`，因此 `next()` 返回的是 `Option<&i32>`。

由于 `iter()` 只是借用 `a` 中的元素，并没有取得元素的所有权，因此迭代完成后，原来的 `a` 仍然可以正常使用。

## 可变引用迭代器 `iter_mut()`

如果不仅需要读取元素，还需要通过迭代器修改集合中的元素，可以使用 `iter_mut()`。

```rust
fn main() {
    let mut a = [1, 2, 4];

    for elem in a.iter_mut() {
        *elem += 2;
    }

    assert_eq!(a, [3, 4, 6]);
}
```

这里通过 `iter_mut()` 获取了一个可变引用迭代器，其元素类型为 `&mut i32`。

因此：

```rust
for elem in a.iter_mut()
```

中的 `elem` 是原数组元素的可变引用。

通过：

```rust
*elem += 2;
```

可以修改 `elem` 所引用的原始元素。

之所以能够修改原集合中的元素，是因为 `iter_mut()` 返回的是元素的**可变引用**，这些引用指向的仍然是原集合中的元素。

## 所有权迭代器 `into_iter()`

如果需要在迭代过程中取得集合元素的所有权，可以使用 `into_iter()`。

```rust
fn main() {
    let a = [
        "1".to_string(),
        "2".to_string(),
        "3".to_string(),
    ];

    let mut iter = a.into_iter();

    assert_eq!(Some("1".to_string()), iter.next());
    assert_eq!(Some("2".to_string()), iter.next());
    assert_eq!(Some("3".to_string()), iter.next());
    assert_eq!(None, iter.next());

    println!("{:?}", a); // 编译错误
}
```

编译时会产生类似错误：

```shell
error[E0382]: borrow of moved value: `a`
```

这是因为：

```rust
let mut iter = a.into_iter();
```

会消费 `a`。

`into_iter()` 获取的是 `a` 的所有权，因此之后不能再使用原来的 `a`。

这里需要注意，`into_iter()` 取得的是**值的所有权**，而不是“所有权引用”。

例如对于：

```rust
let a = [
    "1".to_string(),
    "2".to_string(),
    "3".to_string(),
];

let mut iter = a.into_iter();
```

`a` 的所有权已经转移给了迭代器。之后调用 `iter.next()` 时，迭代器会逐个把元素的所有权交给调用者。

---

对于整数数组：

```rust
fn main() {
    let a = [1, 2, 3];

    let mut iter = a.into_iter();

    assert_eq!(Some(1), iter.next());
    assert_eq!(Some(2), iter.next());
    assert_eq!(Some(3), iter.next());
    assert_eq!(None, iter.next());

    println!("{:?}", a);
}
```

这里最后的 `println!` 可以正常执行。

这并不是因为 `into_iter()` 会自动复制数组，而是因为 `[i32; 3]` 实现了 `Copy` trait。

因此，在发生复制的情况下，原来的 `a` 仍然可以使用。

而前面的 `[String; 3]` 中，`String` 不实现 `Copy`，所以调用 `into_iter()` 后，原来的数组就不能继续使用。

# 总结

如果只是读取集合中的元素，可以使用 `iter()` 获取元素的不可变引用：

```text
iter() → &T
```

如果需要修改集合中的元素，可以使用 `iter_mut()` 获取元素的可变引用：

```text
iter_mut() → &mut T
```

如果需要取得集合元素的所有权，可以使用 `into_iter()`：

```text
into_iter() → T
```

因此可以简单记忆为：

```text
iter()       → &T
iter_mut()   → &mut T
into_iter()  → T
```

需要特别注意的是，**`iter()`、`iter_mut()` 和 `into_iter()` 是获取迭代器的不同方式，而真正定义“迭代器”的是 `Iterator` trait。**

# 参考资料

* [什么是所有权?](https://kaisery.github.io/trpl-zh-cn/ch04-01-what-is-ownership.html)
* [引用与借用](https://kaisery.github.io/trpl-zh-cn/ch04-02-references-and-borrowing.html#%E5%BC%95%E7%94%A8%E4%B8%8E%E5%80%9F%E7%94%A8)
* [迭代器](https://kaisery.github.io/trpl-zh-cn/ch13-02-iterators.html)
