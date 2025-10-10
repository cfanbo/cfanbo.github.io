---
title: 从Vec的 swap_remove 方法中学到的性能优化
date: 2025-07-10T10:02:20+08:00
type: post
toc: true
url: /posts/vec_swap_remove
categories:
- 程序开发
- rust
tags:
- rust
---

今天在看 tokio 源码时，发现一个 [unpark_worker_by_id](https://github.com/tokio-rs/tokio/blob/master/tokio/src/runtime/scheduler/multi_thread/idle.rs#L129-L145) 函数里调用了标准库Vec 的 [swap_remove](https://github.com/rust-lang/rust/blob/master/library/alloc/src/vec/mod.rs#L2014-L2036)方法，它的实现大概如下

```rust
    pub fn swap_remove(&mut self, index: usize) -> T {
        #[cold]
        #[cfg_attr(not(panic = "immediate-abort"), inline(never))]
        #[optimize(size)]
        fn assert_failed(index: usize, len: usize) -> ! {
            panic!("swap_remove index (is {index}) should be < len (is {len})");
        }

        let len = self.len();
        if index >= len {
            assert_failed(index, len);
        }
        unsafe {
            // We replace self[index] with the last element. Note that if the
            // bounds check above succeeds there must be a last element (which
            // can be self[index] itself).
            let value = ptr::read(self.as_ptr().add(index));
            let base_ptr = self.as_mut_ptr();
            ptr::copy(base_ptr.add(len - 1), base_ptr.add(index), 1);
            self.set_len(len - 1);
            value
        }
    }
```

它的作用主要是将 vec 里指定 `index` 元素从vec中移除并返回，同时将 vec 最后一个位置的元素移动到指定 `index` 位置，同时减小vec的长度，并保证元素内容与长度一致，从而达到删除元素的效果。

```rust
fn main() {
    let mut v = vec![10, 20, 30, 40];

    let removed = v.swap_remove(1);

    println!("removed: {}", removed); // 20
    println!("v: {:?}", v);           // [10, 40, 30]
}
```

这个方法与 `remove` 不同，它并不保证原来元素的顺序的，这点从上面打印结果可以看出来。但有一个优势就是执行高效，它的时间复杂度为 `O(1)`，而 `remove` 的时间复杂度则为 `O(n)`，这里对vec 的顺序并不关心，所有使用了 swap_remove。

为了方便大家理解，这里给出 Vec 的定义

```rust
pub struct Vec<T, A: Allocator = Global> {
    buf: RawVec<T, A>,
    len: usize,
}
```

但有个疑问，就是如果移除的元素是最后一位，那这行代码是不是没有必要呢？
```rust
ptr::copy(base_ptr.add(len - 1), base_ptr.add(index), 1);
```

这种情况下，我们一般会使用 `if` 语句做个判断，如果移除的元素 `非` 最后一位元素才执行这行代码，但这里为什么没有添加呢？要知道它可是标准库里的代码，对性能要求极其高的。

而是这背后正是一个非常典型的「**性能与 CPU 分支预测优化**」考虑。

我们知道源码都需要经过一个编译阶段，从而生成目标文件。而在编译时如果发现了 `if` 语句将产生一些分支预测指令，从而增加一些CPU 开销。而现代 CPU 的**分支预测**失败会导致流水线停顿，代价可能是 `10-20` 个时钟周期，但将一个元素复制到自己的位置，只是几个时钟周期的内存操作。

**那什么又是分支预测呢？**

现代 CPU 采用**流水线**执行指令。当遇到条件分支（如 `if`）时，CPU 需要提前决定执行哪个分支，否则流水线会停顿。

```rust
if index != len - 1 {  // <- 分支点
    // 分支 A：执行复制
    ptr.add(len - 1).copy_to(ptr.add(index), 1);
}
// 分支 B：跳过复制，继续后续代码
```



**预测成功**

指 CPU 的分支预测器**猜对了**实际会走哪个分支。

```rust
// 假设前几次调用都是删除中间元素
swap_remove(vec, 2);  // index != len-1, 走分支A
swap_remove(vec, 5);  // index != len-1, 走分支A  
swap_remove(vec, 3);  // index != len-1, 走分支A
swap_remove(vec, 1);  // index != len-1, 走分支A
// CPU 学习到：大概率走分支A

swap_remove(vec, 4);  // index != len-1
// ✅ CPU 预测：走分支A → 实际：走分支A → 预测成功！
```

这时 代价很小，大概 **1-2** 周期，流水线继续执行。

**预测失败**

CPU 的分支预测器**猜错了**

```rust
// CPU 已经学习到大概率走分支A
swap_remove(vec, len-1);  // index == len-1
// ❌ CPU 预测：走分支A → 实际：走分支B（跳过复制）→ 预测失败！
```

这时代价很大，大概 **10-20** 周期，因为：

1. CPU 已经开始执行分支A的指令（复制操作）
2. 发现预测错误，必须**丢弃**已执行的指令
3. **刷新流水线**
4. 从分支B重新开始执行



分支预测器工作原理为：

CPU 内部有一个**分支历史表**（Branch History Table），记录：

- 这个分支指令的地址
- 历史上走了哪个方向
- 预测下次走哪个方向

**简化模型**：

```
地址: 0x1234 (if index != len-1)
历史: A A A A A A A A  (最近8次都走分支A)
预测: 下次走A (强烈偏向)
```

当模式稳定时，预测成功率很高（95%+）。 当模式随机或突然改变时，预测失败率上升。

**总结**

- 标准库里未添加 if 语句进行判断，主要是为了减少**分支预测**带来的性能开支
- 另外，多数场景下，删除最后一个元素的机率相对来说要少一些，从而也在业务测减少了自我复制的调用
