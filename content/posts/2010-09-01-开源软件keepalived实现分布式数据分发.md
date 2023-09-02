---
title: 开源软件keepalived实现分布式数据分发
author: admin
type: post
date: 2010-09-01T01:14:21+00:00
url: /archives/5403
IM_contentdowned:
 - 1
categories:
 - 数据库
tags:
 - keepalived
 - 分布式

---
大家都看到过在支付宝架构图里面一个分布式数据分发中心（Gara系统），这个分布中心为了完成每天的数据抽取和向多个Oracle Rac集群和Greenplum集群分布数据的心脏，数据仓库系统是一切系统数据来源。其中功能是为了完成异构数据抽取和装载。

为了使Gara实现高效性和线性扩展能力，现在alipay dw是用4台高性能PC Dell R900（4*4core，128GB memory）来实现，但是Gara原来开发的程序不能实现分布式，只能通过调度系统来控制，灵活性不够强。

最近发现一个开源软件 Keepalived，能很好来实现Gara高效性和线性扩展能力，很多人用来 Keepalived网站的负载均衡，Keepalived还有一个重要的特性就是实现高可用性，就利用这个特性来实现Gara分布式管理

Keepalived可以提供IP层的高可用性, 一旦某一台机器的网络出现问题, 另一台服务器会立即(几秒或者更少的时间)使用出故障的服务器的IP进行工作。

keepalived体系架构图：

[![](http://blog.haohtml.com/wp-content/uploads/2010/09/1252109344_11666ae6.jpg)][1]
keepalived体系架构很清晰，都模块化的，跟我们目前开发的调度系统架构非常相像，大家可以去仔细研究一下

keepalived工作原理：keepalived是一个类似于IP/TCP协议栈3, 4 & 5层交换机制的软件，也就是我们说的第3层、第4层和第5层交换

1.Keepalived使用Layer3的方式工作式时，Keepalived会定期向服务器群中的服务器。也就是说发送一个ICMP的数据包（类似用的Ping程序）,如果发现某台服务的IP地址没有激活，Keepalived便报告这台服务器失效，并将它从服务器群中剔除。

2.Keepalived使用Layer4的方式工作式时，要以TCP端口的状态来决定服务器工作正常与否。如server的服务端口一般是20000，如果Keepalived检测到20000端口没有启动，则Keepalived将把这台服务器从服务器群中剔除。

3.Keepalived使用Layer5的方式工作式时，工作在具体的应用层，如何我们Gara系统故障，就将应用切换其他机器

 [1]: http://blog.haohtml.com/wp-content/uploads/2010/09/1252109344_11666ae6.jpg