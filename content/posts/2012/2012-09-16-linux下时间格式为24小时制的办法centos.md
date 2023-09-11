---
title: linux下时间格式为24小时制的办法(centos)
author: admin
type: post
date: 2012-09-16T05:31:31+00:00
url: /archives/13403
categories:
 - 服务器
tags:
 - centos

---
下面是自己解决的方法

> tzselect

根据提示选择

> 5 –> 9–>1–>1–>ok
> rm /etc/localtime
> ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

这时就可以看到时间已经修改成为国内的时间了。时间也对的。时间为24小时制。