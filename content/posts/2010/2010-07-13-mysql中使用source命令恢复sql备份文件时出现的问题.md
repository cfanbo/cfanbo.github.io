---
title: mysql中使用source命令恢复sql备份文件时出现的问题
author: admin
type: post
date: 2010-07-13T05:00:38+00:00
url: /archives/4609
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---
当使用mysql做数据库还原的时候，由于有些数据很大，总是失败并决 MySQL server has gone away之类的信息，并自动重新连接数据库且自动继续执行恢复操作，此时没有办法重新指定字符集，容易出现乱码，导致数据库恢复失败，只需要修改max\_allowed\_packet 参数的值即可.

mysql根据配置文件会限制server接受的数据包大小。


有时候大的插入和更新会被max_allowed_packet 参数限制掉，导致失败。


## 1） 方法1

可以编辑my.cnf来修改（windows下my.ini）,在[mysqld]段或者mysql的server配置段进行修改。


```
max_allowed_packet = 20M
```

如果找不到my.cnf可以通过


```
mysql --help | grep my.cnf
```

去寻找my.cnf文件。


## 2） 方法2

**（很妥协，很纠结的办法）**

进入mysql server


```
mysql -h 主机 -u 账号 -p密码
set global max_allowed_packet = 2*1024*1024*10
```

然后关闭掉这此mysql server链接，再进入。


```
show variables like 'max_%'
```

查看下max_allowed_packet是否编辑成功。