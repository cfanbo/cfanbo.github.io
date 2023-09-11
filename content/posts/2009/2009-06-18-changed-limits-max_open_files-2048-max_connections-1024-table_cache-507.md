---
title: 'Changed limits: max_open_files: 2048 max_connections: 1024 table_cache: 507'
author: admin
type: post
date: 2009-06-18T17:37:56+00:00
excerpt: |
 Changed limits: max_open_files: 2048 max_connections: 1024 table_cache: 507

 这个问题怎么解决啊！

 在windows下安装Mysql系统日志出现 max_open_files: 2048 max_connections: 510 table_cache: 764 类似错误是因为 max_connections 最大连接数和max_open_files、table_cache 不匹配。适当的降低max_connections 或调整其他两个数值
 解决办法在 mysql bin > 中输入
 mysql-nt --table_cache=764
 mysql-nt --innodb_open_files=2048 即可！！

 table_cache和max_connections 在my.ini 里可调
url: /archives/1864
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---
Changed limits: max\_open\_files: 2048 max\_connections: 1024 table\_cache: 507

这个问题怎么解决啊！

在windows下安装Mysql系统日志出现 max\_open\_files: 2048 max\_connections: 510 table\_cache: 764 类似错误是因为 **max_connections** 最大连接数和**max\_open\_files**、**table_cache** 不匹配。适当的降低max_connections 或调整其他两个数值
解决办法在 mysql bin > 中输入
mysql-nt –table_cache=764
mysql-nt –innodb\_open\_files=2048   即可！！

table\_cache和max\_connections 在my.ini 里可调

Changed limits:
max\_open\_files: 2048
max_connections: 1024
table_cache: 507

max_connections=1024

table_cache=500

OK,问题不一定解决,只要MYSQL运行正常就行.

按以上的方法处理后还是有啊！