---
title: FreeBSD下安装mysql支持GBK字符集
author: admin
type: post
date: 2011-02-24T03:48:31+00:00
url: /archives/7811
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - mysql

---
在FreeBSD下安装mysql支持GBK字符集：

#cd/usr/ports/databases/mysql51-server
\# make WITH\_CHARSET=gbk WITH\_XCHARSET=all WITH\_PROC\_SCOPE\_PTH=yes BUILD\_OPTIMIZED=yes BUILD\_STATIC=yes SKIP\_DNS\_CHECK=yes WITHOUT\_INNODB=yes install clean
\# cp /usr/local/share/mysql/my-small.cnf /etc/my.cnf
mysqld中加入bind_address=127.0.0.1
#rehash
#mysql\_install\_db
#chown –R mysql:mysql /var/db/mysql
#/usr/local/etc/rc.d/mysql-server forcestart
#/usr/local/bin/mysqladmin -u root password ‘new-password’

有的教程说还需要同时指定”WITH_COLLATION=gbk_chinese_ci“这一项,我没有加上,不知道以后会不会有问题出现的.

 **Mysql 支持的字符集简介**

mysql 服务器可以支持多种字符集（可以用 show character set 命令查看所有 mysql 支持的字符集），在同一台服务器、同一个数据库、甚至同一个表的不同字段都可以指定使用不同的字符集，相比 oracle 等其他数据库管理系统，在同一个数据库只能使用相同的字符集，mysql 明显存在更大的灵活性。
mysql 的字符集包括字符集（ CHARACTER ）和校对规则（ COLLATION ）两个概念。字符集是用来定义 mysql 存储字符串的方式，校对规则则是定义了比较字符串的方式。字符集和校对规则是一对多的关系 , MySQL 支持 30 多种字符集的 70 多种校对规则。

每个字符集至少对应一个校对规则。可以用 SHOW COLLATION LIKE ‘utf8%‘; 命令查看 相关字符集的校对规则。

来源: