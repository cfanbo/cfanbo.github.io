---
title: 'libiconv.so.2: cannot open shared object file解决办法'
author: admin
type: post
date: 2011-04-11T04:23:48+00:00
url: /archives/9211
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux

---
**解决办法如下：**

1.在/etc/ld.so.conf中加一行/usr/local/lib，

2.然后运行/sbin/ldconfig，文件解决，没有报错了~~