---
title: memcached与memcache的区别
author: admin
type: post
date: 2008-09-24T12:30:28+00:00
url: /archives/392
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - memcache
 - memcached

---
memcached 像是一个后台服务器(也有客户端的memcached)，memcache是php的一个模块，需要编译，像是一个客户端，memcached 和 memcache 是紧密结合的两个东西。

另外memcached也是一个客户端的．这点可以参考php手册得知．两者的区别也可以参考：

有关linux下memcache和memcached的安装方法请参考:

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝

**说法一：**

两个不同版本的php的memcached的客户端

new memcache是pecl扩展库版本
new memcached是libmemcached版本
功能差不多．

**说法二：**

**Memcache是什么？**

Memcache是一个自由和开放源代码、高性能、分配的内存对象缓存系统。用于加速动态web应用程序，减轻数据库负载。
它可以应对任意多个连接，使用非阻塞的网络IO。由于它的工作机制是在内存中开辟一块空间，然后建立一个HashTable，Memcached自管理这些HashTable。
Memcached是简单而强大的。它简单的设计促进迅速部署，易于发展所面临的问题，解决了很多大型数据缓存。它的API可供最流行的语言。
Memcache的知名用户有：LiveJournal、Wikipedia、Flickr、Bebo、Twitter、Typepad、Yellowbot、Youtube 等。
Memcache官方网站： [http://memcached.org/](http://memcached.org/)

**Memcached又是什么？
** Memcache是该系统的项目名称，Memcached是该系统的主程序文件，以守护程序方式运行于一个或多个服务器中，随时接受客户端的连接操作，使用共享内存存取数据。

**那PHP中的Memcache是什么？**
php中的所讲的memcache是用于连接Memecached的客户端组件。

简单的说一句话：Memcached 是一个服务（运行在服务器上的程序，监听某个端口），Memcache 是 一套访问Memcached的api。

两者缺一不可，不然无法正常运行。**Memcache能在多台服务器上发挥很好的作用，同台服务器上用APC或Xcache效率是很可观的。**

同台服务器上APC的效率是Memcached的7倍，APC效率比Memcached高是肯定的