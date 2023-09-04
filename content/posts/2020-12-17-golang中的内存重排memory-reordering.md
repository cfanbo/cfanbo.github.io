---
title: Golang中的内存重排(Memory Reordering)
author: admin
type: post
date: 2020-12-17T10:58:36+00:00
url: /archives/20315
categories:
 - 程序开发
tags:
 - golang
 - 内存重排

---
## 什么是内存重排 

内存重排指的是内存的读/写指令重排。

## 为什么要内存重排 

为了提升程序执行效率，减少一些IO操作，一些硬件或者编译器会对程序进行一些指令优化，优化后的结果可能会导致程序编码时的顺序与代码编译后的先后顺序不一致。

就拿做饭场景来说吧，是先蒸米还是先炒菜，这两者是没有冲突的，编译器在编译时有可能与你要求的顺序不一样。

## 编译器重排 

如下面这段代码

```
X = 0
for i in range(100):
    X = 1
    print X
```

要实现打印100次1，很显示在for里面每次都执行X=1语句有些浪费资源，如果将初始变量值修改为1，是不是要快的多。编译器也分析到了这一点，于是在编译时对代码做了以下优化

```
X = 1
for i in range(100):
    print X
```

最终输出结果是一样的，两段代码功能也一样。

**但是**如果此时有另一个线程里执行了一个 X=0 的赋值语句的话（两个线程同时运行），那么输出结果就可能与我们想要的不一样了。

优化前情况：第一个线程执行到了第3次print X 后，第二个线程执行了X=0,把X 的值进行了修改，结果就有可能是1110 1111（在线程执行第5次时，重新执行了X=1，而且之后一直都是 1）。这就有问题了。

优化后情况：按上面的逻辑来执行的话，结果就是11100000…, 后面全是0，再也没有机会将X贬值为1了

由此我们可以得出一个结果，在多线程下，是无法保证编译前后的代码功能是“待价”的。

所以要开发时，这一点我们一定要小心。搞不好在单线程运行没有问题的程序在多线程下会出现各种各样的问题。

每当我们提到内存重排的时候，其实真实在讨论的主题是 内存屏障Memory barrier。指令重排无法逾越内存屏障。

## CPU 重排 

参考： [https://www.w3xue.com/exp/article/20196/40758.html](https://www.w3xue.com/exp/article/20196/40758.html)

## CPU 架构 ![f008c348d5fc364b8298416f3d5cf0fe](https://blogstatic.haohtml.com/uploads/2020/12/0c2d6bdf548e65cbc476df9c2d648e7e.png)_Figure 1. CPU Architecture_![83debca9ea0b15b933075dfe219dee2d](https://blogstatic.haohtml.com/uploads/2020/12/bed268f3590ebf6986a1af64a67e1c3a.png)_Figure 2. Store Buffer_![9ffa8e1bdbe7de39c2ac27c8a90e3a52](https://blogstatic.haohtml.com/uploads/2020/12/c5494abacb226b5d8f0dfee1d1c72ace.jpg)_Figure 3. MESI Protocol_

MESI 可参考 [https://mp.weixin.qq.com/s/vnm9yztpfYA4w-IM6XqyIA](https://mp.weixin.qq.com/s/vnm9yztpfYA4w-IM6XqyIA)



## 相关话题 

* [锁、内存屏障与缓存一致性](https://gocode.cc/project/9/article/128) 

* [memory barrier](https://github.com/cch123/golang-notes/blob/master/memory_barrier.md)

* [[译] 什么是缓存 false sharing 以及如何解决(Golang 示例) ](https://juejin.cn/post/6844903866270482445#comment)

* [CPU缓存体系对Go程序的影响](https://mp.weixin.qq.com/s/vnm9yztpfYA4w-IM6XqyIA)

## 参考 

* [从 Memory Reordering 说起](https://cch123.github.io/ooo)
* [CPU指令重排/内存乱序](https://my.oschina.net/chuqq/blog/3020080)
* [谈谈Go语言中的内存重排](https://www.w3xue.com/exp/article/20196/40758.html)

 