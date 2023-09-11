---
title: g0 特殊的goroutine
author: admin
type: post
date: 2021-02-11T07:05:29+00:00
url: /archives/22353
categories:
 - 程序开发
tags:
 - g0
 - golang

---
在上篇 [《golang中G、P、M 和 sched 三者的数据结构》][1]文章中，我们介绍了`G`、`M` 和 `P` 的数据结构，其中M结构体中第一个字段是 `g0`，这个字段也是一个 `goroutine`，但和普通的 `goroutine` 有一些区别，它主要用来实现对 goroutine 进行调度，下面我们将介绍它是如何实现调度goroutine的。

另外还有一个 `m0` , 它是一个全局变量，与 `g0` 的区别如下![](https://blogstatic.haohtml.com/uploads/2021/03/8e229b7806870bf4f17da207665b8a43.jpg)M0 与 g0的区别

本文主要翻译自 [Go: g0, Special Goroutine](https://medium.com/a-journey-with-go/go-g0-special-goroutine-8c778c6704d8) 一文，有兴趣的可以查阅原文，作者有一系列高质量的文章推荐大家都阅读一遍。ℹ️ 本文基于 Go 1.13。

我们知道在Golang中所有的`goroutine`的运行都是由`调度器`来负责管理的，go调度器尝试为所有的`goroutine`来分配运行时间，当有`goroutine`被阻塞或终止时，调度器会通过对`goroutine` 进行调度以此来保证所有CPU都处于忙碌状态，避免有CPU空闲状态浪费时间。

## goroutine 切换规则 

在此之前我们需要记住一些goroutine切换规则。runtime源码

```
// src/runtime/stubs.go

// mcall switches from the g to the g0 stack and invokes fn(g),
// where g is the goroutine that made the call.
// mcall saves g's current PC/SP in g->sched so that it can be restored later.
// It is up to fn to arrange for that later execution, typically by recording
// g in a data structure, causing something to call ready(g) later.
// mcall returns to the original goroutine g later, when g has been rescheduled.
// fn must not return at all; typically it ends by calling schedule, to let the m
// run other goroutines.
//
// mcall can only be called from g stacks (not g0, not gsignal).
//
// This must NOT be go:noescape: if fn is a stack-allocated closure,
// fn puts g on a run queue, and g executes before fn returns, the
// closure will be invalidated while it is still executing.
func mcall(fn func(*g))
```

对 `mcall()` 函数注释的翻译请参考文章 [Runtime: 当一个goroutine 运行结束后会发生什么](https://blog.haohtml.com/archives/23437)

一、将一个运行中的 Goroutine 切换到另一个的过程涉及到两个切换：

 * 将运行中的 `g` 切换到 `g0`
![2](https://blogstatic.haohtml.com/uploads/2021/02/fb5c81ed3a220004b71069645f112867-48.png)
 * 将 `g0` 切换到下一个将要运行的 `g`
 ![3](https://blogstatic.haohtml.com/uploads/2021/02/10fb15c77258a991b0028080a64fb42d-42.png)

二、在 Go 中，`goroutine` 的切换成本很低，每次切换之前都需要对当前G的状态（`PC/SP`）进行存储（`g.sched`），以便下次恢复运行时读取当前G的上下文信息：

 * Goroutine 在停止运行当前执行的指令时，需先将当前运行的指令存储在程序计数器（`PC`），同时与堆栈指针（`SP`）存储到 `g.sched` 字段。等 Goroutine 恢复执行时，需将程序计数器（ `PC`） 和堆栈指针（`SP`）从 `g.sched` 字段中再次读取出来，并将程序跳转到PC地址继续执行；
 * Goroutine 的堆栈，以便在再次运行时还原局部变量；

如果对这一块还不清楚的话， 推荐阅读 [Go：Goroutine 的切换过程实际上涉及了什么](https://studygolang.com/articles/30251)

让我们看看实际情况下goroutine的切换是怎样进行的。

## 调度 goroutine 

Go 使用 `GOMAXPROCES` 变量控制运行的系统线程个数，这就意味着Go必须在每个运行着的系统线程上调度和管理goroutine。这个调度工作被委托给一个特殊的goroutine，也就是 `g0`。调度过程中 `g0` 会把就绪状态的goroutine调度到系统线程上运行。因此`g0`是每个os线程创建的第一个 goroutine。![](https://blogstatic.haohtml.com/uploads/2021/02/3089d49cd75be4b9a22afe6da7f6eaa9.png)g0

为了更好理解 `g0` 的调度策略，假如现在有一个goroutine #G7，正在运行用户代码

```
ch := make(chan int)
[...]
ch <- v
```

当前 goroutine #G7 往 channel 里写入一个值，但此时接收端尚未准备好，此时goroutine将进行parked，即处于等待状态（ waiting mode ）,G7状态变由 `_Grunning` 为处于`_Gwaiting` 阻塞状态。![](https://blogstatic.haohtml.com/uploads/2021/02/a22be6b0b06be0329f2554e33b66d78d.png)g0

然后g0介入，`g0` 会替换 goroutine 并进行第一轮调度（第一次切换：将 G7 切换到 g0）。![](https://blogstatic.haohtml.com/uploads/2021/02/7a377028cef636f88dfb46c689744450.png)g0

`g0` 会获取一个新的goroutine。优先从当前与 `m` 绑定的 `p` 中的本地运行队列 `q.runq` 中获取一个新的 goroutine `#G2`（第二次切换：g0 再切换到 G2）![](https://blogstatic.haohtml.com/uploads/2021/02/b9eccfab7f4a32ce3ae4259026024d06.jpg)g0

一旦有接收器读取到channel 中的数据，goroutine #G7 将会立即解除阻塞状态。

```
v := <-ch
```

收到消息的goroutine #G7 将切换到 `g0` ，并通过将其放置在本地队列中, 并将状态由 `_Gwaiting` 变为 `_Grunnable` 。（言外之意想让哪个goroutine运行 g0 说了算）![](https://blogstatic.haohtml.com/uploads/2021/02/3b714daf61dc1a61295da06abf86a967.jpg)g0

至此，对 goroutine #G7 的一轮调度完成，G2 继续执行后面的用户代码。如果在 G2 里再也遇到类似阻塞的情况下，则重新发起一轮调度。

## g0 职责 

g0与普通的goroutine不同，g0有着固定且比较大的栈，这样go就可以在需要更大栈的时候就可以直接使用，不需要进行栈增长。

g0职责有：

 * Goroutine创建。
 当调用`go func(){ ... }()`或`go myFunction()`时，Go会将函数的创建委托给`g0`，随后再将其放置在本地队列中。新创建的goroutine优先执行，并放置在本地队列的顶部。为什么放在队顶可参考 [Go：并发与调度器亲和性（ Go: Concurrency & Scheduler Affinity ）][2]
 ![](https://blogstatic.haohtml.com/uploads/2021/02/19dd2a1e4f6cd938072c3e4c3b5f990d.jpg)g0

 * Defer方法分配。
 * GC操作，例如stw，扫描goroutine的栈以及一些标清操作。
 * 栈增长。在需要时，Go会增加goroutine的大小。该操作由`g0`在`prolog`方法中完成。

这个特殊的goroutine `g0`涉及许多其他操作（大量分配，cgo等），使我们的程序可以更高效地管理操作，并且需要更大的栈，以保持我们的程序在低内存下更加高效。

## 参考文章 

 * [https://medium.com/a-journey-with-go/go-g0-special-goroutine-8c778c6704d8](https://medium.com/a-journey-with-go/go-g0-special-goroutine-8c778c6704d8)
 * [Go：Goroutine 的切换过程实际上涉及了什么](https://studygolang.com/articles/30251)
 * [Go：并发与调度器亲和性（ Go: Concurrency & Scheduler Affinity ）][2]
 * [Go netpoller 网络模型之源码全面解析](https://mp.weixin.qq.com/s/HNPeffn08QovQwtUH1qkbQ)
 * [Go的启动周期M0和G0](https://www.bilibili.com/video/BV19r4y1w7Nx)

 [1]: https://blog.haohtml.com/archives/21010
 [2]: https://medium.com/a-journey-with-go/go-concurrency-scheduler-affinity-3b678f490488