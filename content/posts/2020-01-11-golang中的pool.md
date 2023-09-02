---
title: golang中的sync.Pool对象缓存
author: admin
type: post
date: 2020-01-11T05:03:20+00:00
url: /archives/19552
categories:
 - 程序开发
tags:
 - golang

---
## 参考文章 

 * [Golang 的 协程调度机制 与 GOMAXPROCS 性能调优][1]
 * [深入Golang之sync.Pool详解][2]
 * [golang sync.Pool 分析][3]
 * [[译] Go: 理解 Sync.Pool 的设计][4]
 * 视频 [sync.pool对象缓存](https://time.geekbang.org/course/detail/160-87731)

## 知识点 

 * Pool只是一个缓存，一个缓存，一个缓存。由于生命周期受GC的影响，一定不要用于数据库连接池这类的应用场景，它只是一个缓存。
 * golang1.13版本对 [Pool](https://golang.org/pkg/sync/#Pool) 进行了优化，结构体添加了两个字段 victim 和 victimSize。
 * 适应于通过复用，降低复杂对象的创建和GC代价的场景
 * 因为init()的时候会注册一个PoolCleanup函数，他会在gc时清除掉sync.Pool中的所有的缓存的对象。所以每个sync.Pool的生命周期为两次GC中间时段才有效，可以手动进行gc操作 **runtime.GC()**
 * 由于要保证协程安全，所以会有锁的开销
 * 每个Pool都有一个私有池（协程安全）和共享池(**协程不安全**)，其中私有池只有存放一个值。
 1. 每次Get()时会先从当前P的私有池private中获取（ [类似MPG模型中的G](https://studygolang.com/articles/11825)）
 2. 如果获取失败，再从当前P的共享池share中获取
 3. 如果仍失败，则从其它P中共享池中拿**一个**，需要加锁保证协程安全
 4. 如果还失败，则表示所有P中的池(也有可能只是共享池)都为空，则需要New()一个并直接返回（此时不会被放入池中）
 每次取值出来后，会从原来存储的地方将该值删除。

 [1]: https://juejin.im/post/5b7678f451882533110e8948
 [2]: https://www.cnblogs.com/sunsky303/p/9706210.html
 [3]: https://www.jianshu.com/p/494cda4db297
 [4]: https://juejin.im/post/5d006254e51d45776031afe3