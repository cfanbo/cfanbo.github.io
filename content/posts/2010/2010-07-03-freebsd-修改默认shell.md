---
title: FreeBSD 修改默认SHELL
author: admin
type: post
date: 2010-07-03T15:12:19+00:00
url: /archives/4326
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - shell

---
FreeBSD下默认的shell为CSH，可以通过命令 echo $SHELL来查看系统默认的shell是哪一个的。

显示自己所使用的SHEEL命令：
ps或echo $SHELL
修改默认SHELL为csh
name：是指你登陆的名称

> pw usermod -n name -s csh

查看所有支持的shell

> freebsd# cat /etc/shells
> /bin/sh
> /bin/csh
> /bin/tcsh

到于bash的安装请参考: