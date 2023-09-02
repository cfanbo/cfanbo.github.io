---
title: Memcache VS Memcached VS MemcacheDB
author: admin
type: post
date: 2011-03-23T05:52:54+00:00
url: /archives/8089
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - memcache
 - memcached
 - memcachedb

---
1、 简单讲Memcache和Memcached都讲的是同一个开源项目http://memcached.org/，只不过Memcached一般指的是后台的cache server（其实也是一个客户端的，参考php手册)．而Memcache指的访问cache server的客户端。Memcached提供了两种访问协议，ASCII和Binary。

2、 MemcacheDB=Memcached+BerkeleyDB组成的轻量的持久数据库，与前两者是不同的两个东西。

3、作为数据库就要讲究consistency，但是Memcached是一种分布式的缓存机制，因此并不严格要求consistency，而且实际上每个memcached server之间本身不通讯也不共享，所谓的分布式是由memcached的客户端程序来决定的。一般分布式算法采用基于server节点数的取余法，这种方法以node数为基础，因此增减服务器就会造成很大hash失效问题。所以改进的算法一般采用consistent hash算法，这种算法取消了以服务器节点数作为基数的理念，而是直接对服务器的节点进行hash，然后散布在0~2^32的圆周上，同样对于数据的key也采用同样的hash进行散布，对于key hash的结果取其所在圆周顺时针方向最近的一个服务器节点进行set，这样当增减服务器时，只会影响散布在该增减的服务器到逆时针方向的上一个服务器节点之间的数据。一下子减少了损失。

4、持久性。Memcached没有持久性，down机数据即丢失，其中的数据采用LRU算法。而Memcachedb是一种持久的数据库存储。也可以参考：

[http://stackoverflow.com/questions/1825256/memcache-vs-memcached](http://stackoverflow.com/questions/1825256/memcache-vs-memcached)

[Memcache的分布式算法](http://tech.idv2.com/2008/07/24/memcached-004/)

Here is a quick backgrounder in naming conventions (for those unfamiliar), which explains the frustration by the question asker: For many *nix applications, the piece that does the backend work is called a “daemon” (think “service” in Windows-land), while the interface or client application is what you use to control or access the daemon. The daemon is most often named the same as the client, with the letter “d” appended to it. For example “imap” would be a client that connects to the “imapd” daemon.

This naming convention is clearly being adhered to by memcache when you read the introduction to the memcache module (notice the distinction between memcache and memcached in this excerpt):


> Memcache module provides handy procedural and object oriented interface to memcached, highly effective caching daemon, which was especially designed to decrease database load in dynamic web applications.
>
>
> The Memcache module also provides a session handler (memcache).
>
>
> More information about memcached can be found at » [http://www.danga.com/memcached/](http://www.danga.com/memcached/).

The frustration here is caused by the author of the PHP extension which was badly named memcached, since it shares the same name as the actual daemon called memcached. Notice also that in the introduction to memcached (the php module), it makes mention of libmemcached, which is the shared library (or API) that is used by the module to access the memcached daemon:


> memcached is a high-performance, distributed memory object caching system, generic in nature, but intended for use in speeding up dynamic web applications by alleviating database load.
>
>
> This extension uses libmemcached library to provide API for communicating with memcached servers. It also provides a session handler (memcached).
>
>
> Information about libmemcached can be found at » [http://tangent.org/552/libmemcached.html](http://tangent.org/552/libmemcached.html).

In summary, both are functionally the same, but they simply have different authors, and the one is simply named more appropriately than the other.