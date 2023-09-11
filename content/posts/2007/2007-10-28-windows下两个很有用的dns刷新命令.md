---
title: windows下两个很有用的dns刷新命令
author: admin
type: post
date: 2007-10-28T11:51:50+00:00
url: /archives/193
IM_contentdowned:
 - 1
categories:
 - 服务器

---
首先过往command提示符下:

先运行:ipconfig/displaydns这个命令,查看一下本机已经缓存了那些的dns信息的,然后输入下面的命令

ipconfig/flushdns

这时本机的dns缓存信息已经清空了,我们可以再次输入第一次输入的命令来看一下,

ipconfig/displaydns