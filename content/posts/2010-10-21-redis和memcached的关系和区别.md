---
title: Redis和memcached的关系和区别
author: admin
type: post
date: 2010-10-21T05:59:27+00:00
url: /archives/6306
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - memcached
 - redis

---
**Q1：什么是Redis？**

一种新型open source NoSQL应用，从协议上看它提供高性能key-value以及集合操作等功能。它不仅仅是个cache产品，同时支持persistent和chain replication。

**Q2：Redis与Memcached类似的地方**

Redis是最类似Memcached的开源项目，也有很多的client库可供选择。

**Q3：Redis与Memcached的区别**

**功能：**

1. 从协议上看，Redis提供更多的数据结构和操作，如set, stack, range操作等。

2. Redis不仅仅是个cache，提供persistent和chain replication，加上#1，可以近似替代Memcached+MySQL

3. Memcached提供binary protocol和sasl协议，ms Redis还没有支持

**实现：**

1. 内存管理方式差别很大，Redis使用独立的VM管理内存，将大部分value进行压缩后放到磁盘上，类似swap out的概念

2. Memcached使用libevent库，而Redis则原生的使用了epoll，kqueue等多个异步通信模型

**性能：**

这个还真不好说，众说纷纭

Memcached快：

Redis快：

[http://systoilet.wordpress.com/2010/08/09/redis-vs-mem][1][cached/][1]

更有人测出来说Redis要快10倍，我觉得这个不大靠谱，不然Memcached直接88好了。我倾向于相信Redis稍微慢一点，但是由于它在相当程度上顶Memcached+MySQL，所以才更值得期待。有空测测看。

**Links：**

协议：

必读Tutorial：

Benchmark：

VM设计和实现：

 [1]: http://systoilet.wordpress.com/2010/08/09/redis-vs-memcached/