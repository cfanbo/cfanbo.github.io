---
title: freebsd下启动、停止 MySQL命令
author: admin
type: post
date: 2010-07-07T06:23:34+00:00
url: /archives/4447
IM_contentdowned:
 - 1
categories:
 - MySQL
 - 服务器
tags:
 - mysql

---
**启动、停止 MySQL**

要启动 MySQL 的方法：（以本文将 MySQL 安装在 /usr/local/mysql 为例）

> \# /usr/local/mysql/share/mysql.server start

如果安装目录使用的是默认的话,请使用

>  /usr/local/etc/rc.d/mysql-server start|stop|restart

注意在第一次执行前，须将 mysql.server 设成可执行（chmod 744 mysql.server），另外可将这行指令加在 /etc/rc.d/rc.local 档中，让 MySQL 在开机时自动启动。

**要停止 MySQL 的方法：**

> \# /usr/local/mysql/bin/mysqladmin shutdown

如果你为 MySQL Administrator root 帐号（非作业系统的 root）设了密码，要停止 MySQL 则必须像下列这样做，MySQL 会询问你 root 的密码後才会执行 shutdown 的工作：

> \# /usr/local/mysql/bin/mysqladmin -u root -p shutdown