---
title: freebsd系统top命令中的THR表示的意思
author: admin
type: post
date: 2011-03-22T11:06:09+00:00
url: /archives/8077
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - top

---
mysql 正常情况下THR是17-30之间，有时侯会突然升到100以上

> **PID USERNAME THR PRI NICE SIZE RES STATE C TIME WCPU COMMAND**
> 852 mysql 17 102 0 519M 293M ucond 1 93:42 12.06% mysqld
> 1587 www 1 4 0 65256K 15224K accept 2 0:04 0.05% php-cgi

其中thr表示“该进程所拥有的线程数。”

对于Top命令介绍请参考: