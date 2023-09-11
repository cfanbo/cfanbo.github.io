---
title: MySQL Proxy工作机制浅析
author: admin
type: post
date: 2009-05-16T04:13:51+00:00
excerpt: MySQL Proxy处于客户端应用程序和MySQL服务器之间，通过截断、改变并转发客户端和后端数据库之间的通信来实现其功能，这和WinGate之类的网络代理服务器的基本思想是一样的。代理服务器是和TCP/IP协议打交道，而要理解MySQL Proxy的工作机制，同样要清楚MySQL客户端和服务器之间的通信协议，MySQL Protocol包括认证和查询两个基本过程：
url: /archives/1391
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---
MySQL Proxy处于客户端应用程序和MySQL服务器之间，通过截断、改变并转发客户端和后端数据库之间的通信来实现其功能，这和**WinGate**之类的网络代理服务器的基本思想是一样的。代理服务器是和TCP/IP协议打交道，而要理解MySQL Proxy的工作机制，同样要清楚MySQL客户端和服务器之间的通信协议，**MySQL Protocol**包括认证和查询两个基本过程：

认证过程包括：

 1. 客户端向服务器发起连接请求
 2. 服务器向客户端发送握手信息
 3. 客户端向服务器发送认证请求
 4. 服务器向客户端发送认证结果

如果认证通过，则进入查询过程：

 1. 客户端向服务器发起查询请求
 2. 服务器向客户端返回查询结果

当然，这只是一个粗略的描述，每个过程中发送的包都是有固定格式的，想详细了解MySQL Protocol的同学，可以去 [这里](http://forge.mysql.com/wiki/MySQL_Internals_ClientServer_Protocol) 看看。MySQL Proxy要做的，就是介入协议的各个过程。首先MySQL Proxy以服务器的身份接受客户端请求，根据配置对这些请求进行分析处理，然后以客户端的身份转发给相应的后端数据库服务器，再接受服务器的信息，返回给客户端。所以MySQL Proxy需要同时实现客户端和服务器的协议。由于要对客户端发送过来的SQL语句进行分析，还需要包含一个SQL解析器。可以说MySQL Proxy相当于一个轻量级的MySQL了，实际上，MySQL Proxy的admin server是可以接受SQL来查询状态信息的。

MySQL Proxy通过**lua**脚本来控制连接转发的机制。主要的函数都是配合MySQL Protocol各个过程的，这一点从函数名上就能看出来：

 * connect_server()
 * read_handshake()
 * read_auth()
 * read\_auth\_result()
 * read_query()
 * read\_query\_result()

至于为什么采用**lua**脚本语言，我想这是因为MySQL Proxy中采用了 [**wormhole** 存储引擎](http://jan.kneschke.de/projects/mysql/wormhole-storage-engine) 的关系吧，这个虫洞存储引擎很有意思，数据的存储格式就是一段lua脚本，真是创意无限啊。