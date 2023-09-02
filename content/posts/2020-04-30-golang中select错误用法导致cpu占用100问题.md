---
title: Golang中select用法导致CPU占用100%的问题分析
author: admin
type: post
date: 2020-04-30T05:50:11+00:00
url: /archives/20017
categories:
 - 程序开发
tags:
 - golang

---
上一节( [golang中有关select的几个知识点](https://blog.haohtml.com/archives/19670))中介绍了一些对于select{}的一些用法，今天介绍一下有关select在 `for语句` 中由于使用不当引起的CPU占用100% 的案例。

先看代码

```
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan int, 10)
	// 读取chan
	go func() {
		for {
			select {
			case i := <-ch:
				// 只读取15次chan
				fmt.Println(i)
			default:
				// 读取15次chan以后的操作一直在这个空语句无任何IO操作的default条件里死循环，无法出让P，以保证一个GPM关系。
				// 而如果无default条件的话，则系统当读取完15次chan后，当前goroutine会发生 chan IO 阻塞, Go调度器根据GPM的调度关系,会将当前执行关系中的G切换出去，再从LRQ队列中取一个新的G，重新组成一个GPM继续执行，以实现合理利用计算机资源，提高GO的高并发性能
			}
		}
	}()
	// 写入10个值到chan
	for i := 0; i < 15; i++ {
		ch <- i
	}
	// 模拟程序效果使用
	time.Sleep(time.Minute)
}
```

## 实现功能 

通过操作chan来实现消费者

## 问题现象 

但在运行的时候，即发现CPU占用率100%，下面我们分析一下什么原因引起的。![8f70268c7a19a44bb46b9910fa7da6a1](https://blogstatic.haohtml.com/uploads/2020/04/6740e15e4366606fc37febdc7e56f03d.png)golang

## 问题分析 

程序运行时，先使用go关键字创建一个 `goroutine`，里面是一个 `for` 循环语句。`for` 语句里面通过 `select{}` 来监听是否有 `chan` 的 `IO` 操作，当 `ch` 中有可以读取的数据时，则将值打印出来。没有的话则执行 `default` 语句，而这里 `default` 语句为空，所以继续下一次for语句，`for{}` 是一个死循环语句。

当读取 `15` 次 `ch` 后，由于`ch` 会永远处于阻塞状态，所以会一直执行 `default` 条件，然后再执行 `for` 循环。此时这段逻辑基本演变成了一个空的 `for{}` 语句，所以会导致CPU占用 `100%`。

如果没有外层的 `for{}` 语句的话，这样写则没有任何问题的。

## 解决办法 

既然我们知道这个 `goroutine` 会一直在死循环的执行空的语句，导致一直占用着 `cpu` 不放，我们只需要让当前goroutine出让CPU控制权给其它 `goroutine` 即可。根据 [GPM调度原理](https://mp.weixin.qq.com/s/ihJFa5Wir4ohhZUXVSBvMQ)，这里我们只需要让操作 `chan` 的IO语句进行阻塞即可，这样 `Go Seched` 就会发现goroutine发生阻塞，于是将当前 `G` 切换出去，重新调度一个新的G过来，开始一个新的GPM关系继续运行。
这里最简单的实现方法就注释掉 **`default`** 语句即可。将当前 `goroutine` 死循环状态变成阻塞状态。

注意这里的阻塞和执行空的 `default` 是两码事，阻塞是执行到这里主停止不再继续执行了，而这里有空的 `default`, 表示的是无执行代码，但本次循环是可以结束的，继续下一次循环，就是一个死循环而已。

**对于Go中的阻塞需要了解一下有哪些场景会发生，可以参考上面提到的GPM文章。**
常见的阻塞一般发生在像网络请求、系统调用进行磁盘IO操作、执行Sleep函数等，而针对每一种阻塞的处理方式也不一样。
如果是网络导致的阻塞的话，则直接将G切换到网络轮询器NetPoller继续执行, 为PM重新调度过来一个新的G继续执行，等网络请求完成后，再将G追加到LRQ的队列尾部等待再次执行。
而对于系统调用产生的IO阻塞这种情况，则 `Go Seched` 则直接将M和G同时切换出去，并保持MG继续执行IO操作，出让出来的P再分配一个新的MG继续执行。等原来IO操作完成后,将原来的G放入LRQ队列，等待P的再次执行，而原来的M放在一旁等待被重复调用使用。可以看到原来的G和M关系到此结束了，下次与G执行的M不一定是原来的M了， 所以G再次执行就需要知道运行状态，堆栈之类的信息，而这些正是存储在G的结构体中了。

## 对于GPM关系中的几个要点 

Go 调度器模型我们通常叫做**G-P-M 模型**，他包括 4 个重要结构，分别是**G、P、M、Sched**。

 * G 并非执行体，每个 G 需要绑定到 P 才能被调度执行。
 * P 的数量决定了系统内最大可并行的 G 的数量（前提：物理 CPU 核数  >= P 的数量）。
 * M 并不保留 G 状态，这是 G 可以跨 M 调度的基础。M的数量是由Go Runtime来调整，目前默认最大限制为10000个。
 * Sched：Go 调度器，它维护有存储 M 和 G 的队列以及调度器的一些状态信息等。
 * Go 调度器中有两个不同的运行队列：全局运行队列(GRQ)和本地运行队列(LRQ)。
 * 多个 Goroutine 通过用户级别（**用户态**）的上下文切换来共享内核线程 M 的计算资源（**内核态**），但对于操作系统来说并没有线程上下文切换产生的性能损耗。![](https://blog.haohtml.com/wp-content/uploads/2018/10/go-mpg-1.jpg)

调度器循环的机制大致是从各种队列、P 的本地队列中获取 G，切换到 G 的执行栈上并执行 G 的函数，调用 Goexit 做清理工作并回到 M，如此反复。

推荐 [Go 为什么这么“快”](https://mp.weixin.qq.com/s/ihJFa5Wir4ohhZUXVSBvMQ)[]()