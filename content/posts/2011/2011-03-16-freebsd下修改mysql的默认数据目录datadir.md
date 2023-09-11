---
title: freebsd下修改mysql的默认数据目录datadir
author: admin
type: post
date: 2011-03-16T08:40:00+00:00
url: /archives/7989
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - mysql

---
以前用windows的时候,发现直接修改my.ini文件里的datadir变量就可以了,现在发现在FreeBSD下直接修改这个变量值不行的,进到mysql后,用命令mysql>show variables like ‘datadir’ 查看的时候还是默认的/var/db/mysql这个路径.在网上查了一些资料,正解方法如下.

数据目录为：/usr/local/mysql

> 运行/usr/local/bin/mysql\_install\_db –datadir=/usr/local/mysql –user=mysql

vi /etc/rc.conf,加入 mysql_enable=”YES”

> mysql_dbdir=”/usr/local/mysql”

启动myql,现在原来账户的信息全部丢失.(不知道如果把原来的数据全部复制到这里行不行的,没有测试!)

> /usr/local/bin/mysqladmin -u root password ‘密码’

#mysql -u root -p

进入看一下

> show variables like “datadir”

OK, 出现datadir=/usr/local/mysql