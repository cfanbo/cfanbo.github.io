---
title: ttl expired in transit
author: admin
type: post
date: 2010-04-02T03:14:23+00:00
url: /archives/3253
IM_contentdowned:
 - 1
categories:
 - 服务器

---

```
1）TTL值太小！TTL值小于你和对方主机之间经过的路由器数目。

2）路由器数量太多，经过路由器的数量大于TTL值

3)网络存在环路

用 TRACERT命令查看所经过的路由

#tracert 域名或者ip
```