---
title: Unix下重置mysqlroot密码
author: admin
type: post
date: 2010-04-02T10:14:17+00:00
url: /archives/3264
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---

**mysql忘记root 密码如何处理?**

 如果 MySQL 正在运行，首先结束mysql进程： killall mysqld


 启动 MySQL (非正常方式起动)：/usr/local/mysql/bin/mysqld_safe –-skip-grant-tables &


 这样就可以不需要密码进入 MySQL ：/usr/local/mysql/bin/mysql -u root -p　（要求输入密码时直接回车即可）


 mysql> update user mysql.set password=password(”新密码”) where user=”root”;


 mysql> flush privileges;


 mysql> quit;


 重新结束进程：killall mysqld


 用正常方式启动 MySQL ：/usr/local/mysql/bin/mysqld_safe -user=mysql &