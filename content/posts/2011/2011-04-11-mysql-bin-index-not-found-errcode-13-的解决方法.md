---
title: '‘./mysql-bin.index’ not found (Errcode: 13) 的解决方法'
author: admin
type: post
date: 2011-04-11T02:02:04+00:00
url: /archives/9202
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - mysql

---
今天突然收到消息机房的一台服务器的mysql无法启动了，首先检查了一下mysql的错误日志，发现最后出现以下错误：

> 020101 00:42:21  mysqld started
> /usr/local/mysql/libexec/mysqld: File ‘./mysql-bin.index’ not found (Errcode: 13)
> 020101  0:42:21 [ERROR] Aborting
>
> 020101  0:42:21 [Note] /usr/local/mysql/libexec/mysqld: Shutdown complete

提示./mysql-bin.index无法找到（由于mysql开启了bin日志功能），到数据库根目录查看该文件是存在的，可能是文件权限的问题，查看了数据库根目录的权限是700，所有者和用户组都是root，可能是上次转移数据库的时候不小心修改了文件夹的权限。

**解决方法：**

> chgrp -R mysql ./var && chown -R mysql ./var  （这里数据库根目录为/\*****/var）

重新启动mysql  [OK]