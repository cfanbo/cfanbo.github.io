---
title: linux下启动mysql提示”mysql deamon failed to start”错误的解决办法
author: admin
type: post
date: 2013-03-09T10:50:44+00:00
url: /archives/13718
categories:
 - 服务器
tags:
 - selinux

---
有台linux服务器，系统为centos系统.

描述：

网站突然连接不上数据库,于是朋友直接重启了一下服务器。进到cli模式下，执行 service myqsld start 发现还是提示”mysql deamon failed to start”错误信息.

>

> # /etc/init.d/mysqld start
>

>
>

> MySQL Daemon failed to start.
>

>
>

> Starting mysqld:                                           [FAILED]
>

查看mysqld的log文件

>

> #less /var/log/mysqld.log
>

>
>

> /usr/libexec/mysqld: Can’t change dir to ‘XXX’ (Errcode: 13)
>

首先是查看数据库日志

mysqld started


> ### [Warning] Can’t create test file xxx.lower-test   [Warning] Can’t create test file xxx.lower-test   /usr/libexec/mysqld: Can’t change dir to ‘/xxx’ (Errcode: 13)
>
> [ERROR] Aborting

首先检查数据目录和日志目录的权限和所属用户，权限和所属用户都没问题，那应该是SELINUX的权限限制了。


先查看当前配置信息.

> \# getenforce
>
> Enforcing

就表明SELinux已经启用.只需要关闭即可。

**关闭方法：**

> #setenforce 0       （0|1  开|关）

或者

> setsebool ftpd\_disable\_trans 1

命令也可以.