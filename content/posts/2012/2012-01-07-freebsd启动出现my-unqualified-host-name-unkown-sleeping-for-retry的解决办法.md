---
title: FreeBSD启动出现”My unqualified host name unkown…Sleeping for retry”的解决办法
author: admin
type: post
date: 2012-01-07T08:27:39+00:00
url: /archives/12395
IM_contentdowned:
 - 1
categories:
 - 服务器

---
最简单的方法是把/etc/rc.conf里的hostname改成”localhost”。即

> hostname=”localhost”