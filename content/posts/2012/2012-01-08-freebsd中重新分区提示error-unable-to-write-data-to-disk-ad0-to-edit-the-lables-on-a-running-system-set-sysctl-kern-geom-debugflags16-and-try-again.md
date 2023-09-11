---
title: 'FreeBSD中重新分区提示”ERROR: Unable to write data to disk ad0! To edit the lables on a running system set sysctl kern.geom.debugflags=16 and try again.”的解决办法'
author: admin
type: post
date: 2012-01-08T05:09:23+00:00
url: /archives/12399
IM_contentdowned:
 - 1
categories:
 - 前端设计

---
今天将FreeBSD系统重新安装系统的时候.将原来的分区全部删除.进行重新分区,而按下W进行分区保存的时候.提示以下错误:

> ERROR: Unable to write data to disk ad0! To edit the lables on a running system set sysctl kern.geom.debugflags=16 and try again.

解决办法如下:

用root权限运行以下任何一条命令：

**#sysctl -w kern.geom.debugflags=16**

 或者

**#sysctl  kern.geom.debugflags=16**

 你可以用sysctl -a查询你系统的所有内核子系统的配置参数，在具备权限的情况下，你可以修改配置变量，其中有一些只读的属性无法修改，有一些属性只能在开机时设定而不是运行时动态修改的也不能改（这些参数/属性在/boot/loader.conf中调整和修改）