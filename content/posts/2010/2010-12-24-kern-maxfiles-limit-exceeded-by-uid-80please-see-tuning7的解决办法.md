---
title: kern.maxfiles limit exceeded by uid 80,please see tuning(7)的解决办法
author: admin
type: post
date: 2010-12-24T08:36:16+00:00
url: /archives/7136
IM_contentdowned:
 - 1
categories:
 - 服务器

---
\# sysctl kern.maxfiles
kern.maxfiles: 3912

这个值太小了,需要修改一下

通过#sysctl 命令可以查看所有内核配置的信息

[配置FreeBSD的内核 http://www.51docs.net/FreeBSD-Manual/kernelconfig.html](http://www.51docs.net/FreeBSD-Manual/kernelconfig.html)