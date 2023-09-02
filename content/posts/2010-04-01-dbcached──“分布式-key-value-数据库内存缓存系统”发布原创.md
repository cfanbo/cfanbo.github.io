---
title: 'dbcached──“分布式 key-value 数据库内存缓存系统”发布[原创]'
author: admin
type: post
date: 2010-04-01T14:35:42+00:00
url: /archives/3183
IM_data:
 - 'a:1:{s:51:"http://blog.s135.com/attachment/200802/dbcached.gif";s:68:"http://blog.haohtml.com/wp-content/uploads/2011/03/a725_dbcached.gif";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - dbcached

---

前言：dbcached 1.0 beta* 在 Memcached 1.2.4 的基础上编写而成，也是我的第一个开源C项目。编写 dbcached 的目的是为了最大限度的发挥 Memcached 内存缓存的优势，便捷地维护 Memcached 服务器节点哈希列表，智能地支持 Memcached 故障转移，同时保证数据的持久化存储。

dbcached

协议： [New BSD License](http://www.opensource.org/licenses/bsd-license.php)
作者：张宴
网址： [http://code.google.com/p/dbcached/](http://code.google.com/p/dbcached/) **dbcached 是什么?**

● dbcached 是一款基于 Memcached 和 NMDB 的分布式 key-value 数据库内存缓存系统。

● **dbcached = Memcached + 持久化存储管理器 + NMDB 客户端接口**

● Memcached 是一款高性能的，分布式的内存对象缓存系统，用于在动态应用中减少数据库负载，提升访问速度。

● NMDB 是一款多协议网络数据库(dbm类)管理器，它由内存缓存和磁盘存储两部分构成，使用 QDBM 或 Berkeley DB 作为后端数据库。

● QDBM 是一个管理数据库的例程库，它参照 GDBM 为了下述三点而被开发：更高的处理速度，更小的数据库文件大小，和更简单的API。QDBM 读写速度比 Berkeley DB 要快，详细速度比较见《 [Report of Benchmark Test](http://qdbm.sourceforge.net/benchmark.pdf)》。 [![dbcached](http://blog.haohtml.com/wp-content/uploads/2010/04/dbcached.gif)](http://blog.haohtml.com/wp-content/uploads/2010/04/dbcached.gif) ![](http://blog.s135.com/attachment/200802/dbcached.gif)**Memcached 和 dbcached 在功能上一样吗?**

● 兼容：Memcached 能做的，dbcached 都能做。除此之外，dbcached 还将“Memcached、持久化存储管理器、NMDB 客户端接口”在一个程序中结合起来，对任何原有 Memcached 客户端来讲，dbcached 仍旧是个 Memcached 内存对象缓存系统，但是，它的数据可以持久存储到本机或其它服务器上的 QDBM 或 Berkeley DB 数据库中。

● 性能：前端 dbcached 的并发处理能力跟 Memcached 相同；后端 NMDB 跟 Memcached 一样，采用了libevent 进行网络IO处理，拥有自己的内存缓存机制，性能不相上下。

● 写入：当“dbcached 的 Memcached 部分”接收到一个 set(add/replace/…) 请求并储存 key-value 数据到内存中后，“dbcached 持久化存储管理器”能够将 key-value 数据通过“NMDB 客户端接口”保存到 QDBM 或 Berkeley DB 数据库中。

● 速度：如果加上“-z”参数，采用 UDP 协议“只发送不接收”模式将 set(add/replace/…) 命令写入的数据传递给 NMDB 服务器端，对 Memcache 客户端写速度的影响几乎可以忽略不计。在千兆网卡、同一交换机下服务器之间的 UDP 传输丢包率微乎其微。在命中的情况下，读取数据的速度跟普通的 Memcached 无差别，速度一样快。

● 读取：当“dbcached 的 Memcached 部分”接收到一个 get(incr/decr/…) 请求后，如果“dbcached 的 Memcached 部分”查询自身的内存缓存未命中，则“dbcached 持久化存储管理器”会通过“NMDB 客户端接口”从 QDBM 或 Berkeley DB 数据库中取出数据，返回给用户，然后储存到 Memcached 内存中。如果有用户再次请求这个 key，则会直接从 Memcached 内存中返回 Value 值。

● 持久：使用 dbcached，不用担心 Memcached 服务器死机、重启而导致数据丢失。

● 变更：使用 dbcached，即使因为故障转移，添加、减少 Memcached 服务器节点而破坏了“key 信息”与对应“Memcached 服务器”的映射关系也不怕。

● 分布：dbcached 和 NMDB 既可以安装在同一台服务器上，也可以安装在不同的服务器上，多台 dbcached 服务器可以对应一台 NMDB 服务器。

● 特长：dbcached 对于“读”大于“写”的应用尤其适用。

**1. dbcached**

**Installation (安装)**

引用


wget [http://www.monkey.org/~provos/libevent-1.3e.tar.gz](http://www.monkey.org/%7Eprovos/libevent-1.3e.tar.gz)

tar zxvf libevent-1.3e.tar.gz

cd libevent-1.3e/

./configure –prefix=/usr

make && make install

cd ../

wget [http://dbcached.googlecode.com/files/dbcached-1.0.beta2.tar.gz](http://dbcached.googlecode.com/files/dbcached-1.0.beta2.tar.gz)

tar zxvf dbcached-1.0.beta2.tar.gz

cd dbcached-1.0.beta2/

./configure –prefix=/usr/local/dbcached –with-libevent=/usr

make && make install

cd ../

**Run as a daemon (作为守护进程运行)**

引用


/usr/local/dbcached/bin/memcached -d -m 256 -p 11211 -c 51200 -u nobody -x 192.168.0.2 -y 26010 -z 26010


● -x nmdb 服务器的域名或者IP地址，推荐使用IP地址


● -y <端口号> nmdb 服务器的TCP端口号 (默认: 26010) 支持 set/delete/… 等写命令 和 get 等读命令


● -z <端口号> nmdb 服务器的UDP端口号 (默认: 26010) 只支持 get 等都命令, 当使用 -z 参数时，将使用 UDP 协议代替 TCP 协议执行 set 操作，执行 get 操作时仍然使用 TCP 协议。强烈推荐加上 -z 参数。

● 其他参数跟 memcached 1.2.4 完全一样，就不再详细说明。


● 如果想让 dbcached 通过 NMDB 保存数据时采用 TCP 协议，去掉 -z 参数即可，例如：(除非因防火墙、NAT穿透等问题导致 UDP 协议不可用，否则不建议使用 TCP 协议)


引用


/usr/local/dbcached/bin/memcached -d -m 256 -p 11211 -c 51200 -u nobody -x 192.168.0.2 -y 26010


● 如果想让 dbcached 作为普通的 Memcached 运行，去掉 -x、-y、-z 参数即可，例如：


引用


/usr/local/dbcached/bin/memcached -d -m 256 -p 11211 -c 51200 -u nobody


**2. QDBM & NMDB**

QDBM 和 NMDB 均为原版，可以从它们的官方网站下载最新版本。


**QDBM Installation (安装)**

引用


wget [http://qdbm.sourceforge.net/qdbm-1.8.77.tar.gz](http://qdbm.sourceforge.net/qdbm-1.8.77.tar.gz)

tar zxvf qdbm-1.8.77.tar.gz

cd qdbm-1.8.77/

./configure –prefix=/usr

make

make install

cd ../


**NMDB Installation (安装)**

引用


wget [http://www.monkey.org/~provos/libevent-1.3e.tar.gz](http://www.monkey.org/%7Eprovos/libevent-1.3e.tar.gz)

tar zxvf libevent-1.3e.tar.gz

cd libevent-1.3e/

./configure –prefix=/usr

make && make install

cd ../

wget [http://auriga.wearlab.de/~alb/nmdb/files/0.21/nmdb-0.21.tar.gz](http://auriga.wearlab.de/%7Ealb/nmdb/files/0.21/nmdb-0.21.tar.gz)

tar zxvf nmdb-0.21.tar.gz

cd nmdb-0.21/

make BACKEND=qdbm ENABLE_TIPC=0 ENABLE_SCTP=0 install

cd ../

**Run as a daemon (作为守护进程运行)**

引用


/usr/local/bin/nmdb -d /var/dbcached.db -t 26010 -T 192.168.0.2 -u 26010 -U 192.168.0.2 -c 1024


● -d 数据库路径(这里使用比 Berkeley DB 更快的 QDBM 数据库)，例如 /var/dbcached.db


● -t TCP 监听端口 (默认：26010)


● -T TCP 监听地址 (默认：任何地址)


● -u UDP 监听端口 (默认：26010)


● -U UDP 监听地址 (默认：任何地址)


● -c 最大的缓存对象数目，单位为千 (默认：128)