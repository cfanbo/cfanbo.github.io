---
title: device 0 has different MAC address than expected 的解决办法
author: admin
type: post
date: 2011-10-09T07:29:35+00:00
url: /archives/11641
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux

---
今天克隆了一份vm(centos),发现重启网卡的时候提示”device 0 has different MAC address than expected…”之类的错误,手动修改mac地址也不行.后来找到一种解决办法如下:

删除 HWADDR 一行,然后执行ifconfig和service network restart命令.然后用ifconfig命令查看就会发现已经可以正常使用了.

不过在eth0文件里HWADDR这一行系统并没有自动添加上的.