---
title: Rust 里 thread::park() 与 thread::yield_now() 的区别
date: 2025-08-14T16:18:20+08:00
type: post
toc: true
url: /posts/thread-park-vs-thread-yield_now
categories:
- 程序开发
- rust
- tokio
tags:
- rust
---

在看 tokio 调度源码时，会有一些操作线程park的函数，而在rust标准库里也同样有类似的方法，那就是 [thread::park()](https://doc.rust-lang.org/std/thread/fn.park.html) ，同时还有一个咋一看效果类似的函数 [thread::yield_new()](https://doc.rust-lang.org/std/thread/fn.yield_now.html)， 两个函数都有实现 **`类似`**暂停执行代码的效果，那它们到底又何区别呢？

希望通过这篇文章可以让大家搞明白它们两者的区别和使用场景。

我们先看一下 `thread::park() `

## thread:park()

对于 park 函数的作用主要是实现当前线程的阻塞，并出让CPU，这时OS调度器会将其它线程调度到CPU，继续执行其它任务。但是一旦调用这个函数后，后续线程将一直处于阻塞状态，也就是说此线程将无法获取CPU处理任务，直到调用 unpark() 函数，才恢复正常。

从线程状态角度来看，它的转换

```text
running ---> [thread::park()] ---> blocked (等待唤醒)
blocked ---> [thread::unpark()]---> runnable ---> running (被unpark唤醒后)
```

总结

- 一经调用 `thread::park()` 函数后，线程将一直处于阻塞状态
- 期间没有机会获得CPU
- 需要外部干预（调用unpark）才可以恢复执行状态

这是官方文档的一个示例

```rust
use std::thread;
use std::time::Duration;

let parked_thread = thread::Builder::new()
    .spawn(|| {
        println!("Parking thread");
        thread::park();
        println!("Thread unparked");
    })
    .unwrap();

// Let some time pass for the thread to be spawned.
thread::sleep(Duration::from_millis(10));

println!("Unpark the thread");
parked_thread.thread().unpark(); // 这里恢复阻塞的线程状态，继续执行后续语句 println!("Thread unparked")

parked_thread.join().unwrap();
```

执行结果

```shell
Parking thread
Unpark the thread
Thread unparked
```



## Thread::yield_new()

对于 `yield_new()` 来讲，它是 **暂时** 出让CPU，此时OS调度器将CPU调度到其它线程执行任务。这一点类似于 park 。唯一不同的是，它随时都有机会被再次执行，只有那么一个时间段无法继续执行而已。例如CPU执行完了其它线程，或CPU已经没有其它任务可执行，又或者OS调度器根据算法又将CPU再将调度到这个线程，则此线程任务将继续执行。这里并不需要像 park 那样，需要人工干预(调用解除函数)线程才可以得到继续执行，对于它的执行时机完全由操作系统调度器来实现的。

从线程状态角度来看，它的转换

```text
running ---> runnable (短暂) ---> running
```

总结

- 它只是短暂让出 CPU 给其他线程
- 线程并不会阻塞，只是状态由 running 变为了 runnable，一旦有机会将继续执行
- 当前线程随时都可能被操作系统重新调度回 CPU

示例

```rust
use std::thread;

fn main() {
    let handle1 = thread::spawn(|| {
        for i in 0..5 {
            println!("线程1: {}", i);
            thread::yield_now(); // 主动让出CPU
        }
    });

    let handle2 = thread::spawn(|| {
        for i in 0..5 {
            println!("线程2: {}", i);
            thread::yield_now(); // 主动让出CPU
        }
    });

    handle1.join().unwrap();
    handle2.join().unwrap();
}
```

执行结果大概率是 线程1 与 线程2 **随机交叉**输出的，并不是先输出线程1，再输出线程2 ，打印顺序完全视操作系统的调度机制决定，每次输出的顺序都可能不一样。

 这是本电脑的输出

```shell
线程1: 0
线程1: 1
线程2: 0
线程1: 2
线程1: 3
线程1: 4
线程2: 1
线程2: 2
线程2: 3
线程2: 4
```





## 对比

| 方法          | 调用前状态 | 调用时变化                                   | 调用后状态                                    | CPU 占用                       |
| ------------- | ---------- | -------------------------------------------- | --------------------------------------------- | ------------------------------ |
| `park()`      | running    | 线程进入 **blocked**（阻塞状态）             | blocked → 需要 `unpark()` 或事件才能 runnable | 释放 CPU                       |
| `yield_now()` | running    | 线程暂时让出时间片，但仍在 **runnable** 队列 | runnable → 等待调度器再次分配 CPU → running   | CPU 暂时让出，但线程可随时执行 |



## 总结

对于 `park` 必须手动通过 `unpark` 解除状态。

另外需要注意的是，如果在调用 park 之前，先调用了 `unpark` ，则会直接继续执行代码的，这点在文档 https://doc.rust-lang.org/std/thread/fn.park.html#park-and-unpark 里介绍，开发时一定要注意。

简单理解技巧：

**park** = 我要去睡觉了（park 线程停下，CPU 给别人），需要我干活的时候记得叫醒我（unpark）

**yield_now** = 等一下，让我先喝口水，喘口气（让出时间片，线程还在队列里，随时可能再次执行），喝完水随时都可以让我干活



## 参考资料

- https://doc.rust-lang.org/std/thread/fn.park.html
- https://doc.rust-lang.org/std/thread/struct.Thread.html#method.unpark
- https://doc.rust-lang.org/std/thread/fn.yield_now.html
- [异步初响：从 #[tokio::main] 感知调度脉搏](https://www.bilibili.com/video/BV197njzqEs6/)
