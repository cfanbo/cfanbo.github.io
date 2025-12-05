---
title: 理解 tokio 中的任务
date: 2025-11-17T16:36:48+08:00
type: post
url: /posts/understanding-tasks-in-tokio
toc: true
categories:
- rust
tags:
- tokio
- rust
---



## 什么是 Task

在 tokio 里, task 是一种轻量级、非阻塞的执行单元，它类似于操作系统线程，但对它的调度不是由操作系统来完成的，而是 tokio runtime 来完成的。这种通用的模式，还有一个名称就是  [green threads](https://en.wikipedia.org/wiki/Green_threads)。它很类似于 [Go’s goroutines](https://tour.golang.org/concurrency/1), [Kotlin’s coroutines](https://kotlinlang.org/docs/reference/coroutines-overview.html), or [Erlang’s processes](http://erlang.org/doc/getting_started/conc_prog.html#processes)。

它有三个主要特点，分别是 `轻量级`、`协作式调度` 和 `非阻塞`。



### 轻量级

与Golang里的 goroutine一样，相比操作系统线程而言，它的创建速度非常的快，且很小。对它的调用完成是由用户态代码来完成的。不像操作系统线程那样，每一次调度都需要上下文的切换，因此在用户态切换任务的成本也是极其的低。同样对于它的运行与销毁也是低成本的。



### 协作式调度 

多数操作系统实现的是`抢占式多任务处理`，对于多任务调度一般是由调度器（如操作系统调度器）来负责的，它会根据任务的执行时长，动态的抢占并暂停它，然后执行其它任务，最终实现每个任务都在同步执行的样子。

但 tokio 属于 `协作式调度` 机制，这允许一个任务执行一段时间并 **主动出让** 执行权，告知 `tokio runtime` 它目前无法继续执行，你可以去执行其它任务，等一会会有人告诉你什么时候可以再过来执行我（tokio 里的 driver）。

> Golang 里的调度机制的是抢占式调度机制，而 tokio 里使用的是协作式调度机制，这是它们的区别



### 非阻塞

当 `worker thread` 执行异步任务时，如果遇到阻塞操作，则此任务将主动出让执行权。这时 `worker thread` 重新获取一个新的任务（优先从本地任务队列获取，如果找不到，再依次全局任务队列、从其它 worker thread队列窃取、各类 driver ）。

注意，这里的任务不应该是执行系统调用或其他可能**阻塞线程**的操作，因为这会阻止同一线程上的其他任务执行。

> 这里说的阻塞是指 会阻塞操作系统线程的任务，从而导致无法被tokio 调度的任务，如从磁盘读取一个大文件内容，这时这个worker thread 将一直处于阻塞状态，导致它无法执行其它任务。



## Task 分类

根据任务执行是否可以出让划分，大致可以分成两类：可出让 和 不可出让。



### 可出让执行权

这里是指哪些允许在执行期间主动出让执行权的任务，这样就可以允许 worker thread 继续执行其它的任务。这类任务可以充分发挥系统硬件，最大化执行效率。

对于此类任务，一般是通过 `spawn` 函数创建的。

如在一个协程里执行了 sleep() 函数, 从而导致阻塞，这时 worker thread 去执行另一个协程，从而实现并发执行。

```rust
use tokio::time::{sleep, Duration, Instant};

async fn task_a() {
    println!("A: 开始执行");

    // 这里的 .await 会让出执行权
    sleep(Duration::from_secs(1)).await;

    println!("A: 结束执行");
}

async fn task_b() {
    println!("B: 开始执行");

    // 这里也会让出执行权，让调度器去执行其它任务
    sleep(Duration::from_millis(500)).await;

    println!("B: 结束执行");
}

#[tokio::main]
async fn main() {
    let start = Instant::now();
    let h1 = tokio::spawn(task_a());
    let h2 = tokio::spawn(task_b());

    for h in vec![h1, h2] {
        h.await.unwrap();
    }

    println!("{:?}", start.elapsed());
}
```

执行结果：

```shell
A: 开始执行
B: 开始执行
B: 结束执行
A: 结束执行
1.002575208s
```

另外也可以手动调用 yield_now 显式主动出让执行权，如

```rust
use tokio::task;
use tokio::time::Instant;

async fn task_a() {
    println!("A: 开始执行");

    // 显式让出执行权，调度器可以执行其他任务
    task::yield_now().await;

    println!("A: 结束执行");
}

async fn task_b() {
    println!("B: 开始执行");

    // 显式让出执行权
    task::yield_now().await;

    println!("B: 结束执行");
}

#[tokio::main]
async fn main() {
    let start = Instant::now();
    let h1 = tokio::spawn(task_a());
    let h2 = tokio::spawn(task_b());

    for h in vec![h1, h2] {
        h.await.unwrap();
    }

    println!("{:?}", start.elapsed());
}
```

执行结果:

```shell
A: 开始执行
B: 开始执行
A: 结束执行
B: 结束执行
119.209µs
```



### 不可出让执行权

对于一些阻塞任务，一经调用则将无法暂停，这也就预示着这些任务只要开始执行，就只能等待它执行完成，在此期间线程只能处理它这一件事。如 从磁盘读取文件之类的操作，就属于这类任务。

在 tokio 里有，针对这类任务，一般是通过 spawn_blocking 和 block_in_place 创建。



#### spawn_blocking

如果使用 spawn_blocking 函数的话，则直接创建一个系统线程，专门用来执行这类的阻塞任务。而对于我们上面介绍的非阻塞任务，一般是由 worker thread 来完成的，每一个 worker thread 下面包含一个任务队列，专门存放等待执行的任务。

也就是说对这类任务的、执行，是使用的完全不一样的线程来负责的。

```rust
use std::thread;
use tokio::task;
use tokio::time::{sleep, Duration, Instant};

#[tokio::main(flavor = "multi_thread")]
async fn main() {
    let start = Instant::now();

    // 异步任务
    let async_task = tokio::spawn(async {
        println!("async_task start, thread: {:?}", thread::current().id());
        sleep(Duration::from_secs(1)).await; // 非阻塞
        println!("async_task end, thread: {:?}", thread::current().id());
    });

    // 阻塞任务（CPU-heavy 或同步阻塞）
    let blocking_task = task::spawn_blocking(|| {
        println!("blocking_task start , thread: {:?}", thread::current().id());
        // 模拟耗时阻塞操作
        std::thread::sleep(Duration::from_secs(2));
        println!("blocking_task end, thread: {:?}", thread::current().id());
    });

    // 等待两个任务完成
    let _ = tokio::join!(async_task, blocking_task);

    println!("{:?}", start.elapsed());
}

```

执行结果：

```shell
async_task start, thread: ThreadId(17)
blocking_task start , thread: ThreadId(18)
async_task end, thread: ThreadId(2)
blocking_task end, thread: ThreadId(18)
2.001832625s
```

可以看到对于 blocking_task 的线程id 一直是不变化的，而对于 async task 的worker thread 则是随机的，这也就证实了对于这类任务是一由个专门的线程来处理的。



#### block_in_place

对于 block_in_place 创建的线程，它同样是由 worker thread 来负责的。

```rust
block_in_place(|| {
    // 阻塞代码
});

```

只不过在创建任务后，它并不是将其放在当前worker thread 的任务队列，而是直接就执行。这就造成了当前worker thread处于阻塞状态，为了防止阻塞导致当前队列里的任务长时间得不到执行，这时 tokio 会再创建一个新的 worker thread，重新找一些任务来执行。

这时 worker thread 变成了一个专门的线程。

```rust
use std::thread;
use tokio::task;
use tokio::time::{sleep, Duration};

#[tokio::main(flavor = "multi_thread", worker_threads = 1)] // 单 worker，方便观察
async fn main() {
    println!("Main thread id: {:?}", thread::current().id());

    // 普通 async task
    let t1 = tokio::spawn(async {
        println!("Async task 1 start, thread: {:?}", thread::current().id());
        sleep(Duration::from_secs(3)).await;
        println!("Async task 1 end, thread: {:?}", thread::current().id());
    });

    // 阻塞任务，使用 block_in_place
    let t2 = tokio::spawn(async {
        println!("Blocking task start, thread: {:?}", thread::current().id());

        task::block_in_place(|| {
            println!(
                "Inside block_in_place start, thread: {:?}",
                thread::current().id()
            );
            std::thread::sleep(Duration::from_secs(5)); // 阻塞
            println!(
                "Inside block_in_place end, thread: {:?}",
                thread::current().id()
            );
        });

        println!("Blocking task end, thread: {:?}", thread::current().id());
    });

    // 等待任务完成
    let _ = tokio::join!(t1, t2);
}
```

执行结果：

```shell
Main thread id: ThreadId(1)
Async task 1 start, thread: ThreadId(2)
Blocking task start, thread: ThreadId(2)
Inside block_in_place start, thread: ThreadId(2)
Async task 1 end, thread: ThreadId(3)   <-- 新 worker thread 执行
Inside block_in_place end, thread: ThreadId(2)
Blocking task end, thread: ThreadId(2)
```

步骤：

1. 普通 async task 刚开始打印 `Async task 1 start, thread: ThreadId(2)`，当前线程ID 为 2
2. 后来这个线程被 block_in_place 调用，变成了专用线程，因此打印 `Inside block_in_place... thread: ThreadId(2)`；同时重新了一个新的 worker thread，继续从原来的worker thread 将任务窃取过来继续执行
3. 这时，普通 async task 变成了新的worker thread 3，因此打印 `Async task 1 end, thread: ThreadId(3)`



#### 对比总结

1. 对于 spawn_blocking 创建的任务，它会创建一个专门的线程来处理这个任务
2. 对于 block_in_palce 会导致抢占原来的 worker thread，并创建一个新的 worker thread，与其它worker thread 一起处理原来工作线程剩下的 任务



## 参考资料

- https://docs.rs/tokio/latest/tokio/task/index.html





