---
title: Keepalived服务器状态监测
author: admin
type: post
date: 2010-08-20T10:17:43+00:00
url: /archives/5186
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - lvs

---
keepalived是一个类似于layer3, 4 & 5交换机制的软件，也就是我们平时说的第3层、第4层和第5层交换。Keepalived的作用是检测web服务器的状态，如果有一台web服务器死机，或工作出现故障，Keepalived将检测到，并将有故障的web服务器从系统中剔除，当web服务器工作正常后Keepalived自动将web服务器加入到服务器群中，这些工作全部自动完成，不需要人工干涉，需要人工做的只是修复故障的web服务器。

Layer3,4&5工作在IP/TCP协议栈的IP层，TCP层，及应用层,原理分别如下：

Layer3：Keepalived使用Layer3的方式工作式时，Keepalived会定期向服务器群中的服务器

发送一个ICMP的数据包（既我们平时用的Ping程序）,如果发现某台服务的IP地址没有激活，Keepalived便报告这台服务器失效，并将它从服务器群中剔除，…

**相关keepalived实现的教程:**

基于LVS的集群配置:

利用LVS+Keepalived 实现高性能高可用负载均衡服务器:

CentOS5.5环境下布署LVS+keepalived :