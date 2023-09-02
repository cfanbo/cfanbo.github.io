---
title: '“mysql 错误log InnoDB: Error: pthread_create returned 11″的解决办法'
author: admin
type: post
date: 2014-11-06T01:50:21+00:00
url: /archives/15369
categories:
 - 服务器
tags:
 - mysql

---
最近mysql总是死掉,登录服务器查看，mysql没有启动，启动mysql数据库。

[shell]service mysqld start[/shell]

网站访问正常。

查看服务器没有重启记录，也没有其他用户登录服务器停止mysql数据库。

查看mysql log，有如下提示：

140116 19:38:39 mysqld_safe Number of processes running now: 0

140116 19:38:39 mysqld_safe mysqld restarted

140116 19:38:45  InnoDB: Initializing buffer pool, size = 8.0M

140116 19:38:45  InnoDB: Completed initialization of buffer pool

InnoDB: Error: pthread_create returned 11

140116 19:38:46 mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended

上网google了下。

临时解决方法：

[shell]# ulimit -s unlimited[/shell]

该方法在系统重启，或者重新登录后将失效。

另外一种方法:

修改 /etc/profile 文件，添加

[shell]ulimit -s unlimited[/shell]