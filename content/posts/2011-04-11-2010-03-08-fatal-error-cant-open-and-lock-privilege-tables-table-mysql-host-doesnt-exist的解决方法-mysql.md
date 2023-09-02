---
title: '2010-03-08 Fatal error: Can’t open and lock privilege tables: Table ‘mysql.host’ doesn’t exist的解决方法 – [MySQL]'
author: admin
type: post
date: 2011-04-11T02:31:13+00:00
url: /archives/9206
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - mysql

---
今天在用一个装好的Mysql时，用safe_mysqldq启动的时候，出现

Fatal error: Can’t open and lock privilege tables: Table ‘mysql.host’ doesn’t exist

最终解决方法如下：

在mysql的安装目录下，我的是/usr/local/mysql

> ./scripts/mysql\_install\_db   –usrer=mysql  –datadir=/usr/local/mysql/data/

原因是重装的时候数据目录不一致导致

然后再次启动，OK