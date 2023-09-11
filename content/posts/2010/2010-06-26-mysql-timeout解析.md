---
title: MySQL Timeout解析
author: admin
type: post
date: 2010-06-26T16:59:42+00:00
url: /archives/4111
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---
**“And God said, Let there be network: and there was timeout”**
在使用MySQL的过程中，你是否遇到了众多让人百思不得其解的Timeout？
那么这些Timeout之后，到底是代码问题，还是不为人知的匠心独具？
本期Out-man，讲述咱们MySQL DBA自己的Timeout。

先看一下比较常见的Timeout参数和相关解释：
**connect_timeout**
The number of seconds that the mysqld server waits for a connect packet before responding with Bad handshake.
**interactive_timeout**
The number of seconds the server waits for activity on an interactive connection before closing it.
**wait_timeout**
The number of seconds the server waits for activity on a noninteractive connection before closing it.
**net\_read\_timeout**
The number of seconds to wait for more data from a connection before aborting the read.
**net\_write\_timeout**
The number of seconds to wait for a block to be written to a connection before aborting the write.

从以上解释可以看出，connect\_timeout在获取连接阶段（authenticate）起作用，interactive\_timeout 和wait\_timeout在连接空闲阶段（sleep）起作用，而net\_read\_timeout和net\_write_timeout则是在连接繁 忙阶段（query）起作用。

获取MySQL连接是多次握手的结果，除了用户名和密码的匹配校验外，还有IP->HOST->DNS->IP验证，任何一步都 可能因为网络问题导致线程阻塞。为了防止线程浪费在不必要的校验等待上，超过connect_timeout的连接请求将会被拒绝。

即使没有网络问题，也不能允许客户端一直占用连接。对于保持sleep状态超过了wait\_timeout（或 interactive\_timeout，取决于CLIENT_INTERACTIVE标志）的客户端，MySQL会主动断开连接。

即使连接没有处于sleep状态，即客户端忙于计算或者存储数据，MySQL也选择了有条件的等待。在数据包的分发过程中，客户端可能来不及响应 （发送、接收、或者处理数据包太慢）。为了保证连接不被浪费在无尽的等待中，MySQL也会选择有条件（net\_read\_timeout和 net\_write\_timeout）地主动断开连接。

这么多Timeout足以证明MySQL是多么乐于断开连接。而乐于断开连接的背后，主要是为了防止服务端共享资源被某客户端（mysql、 mysqldump、页面程序等）一直占用。