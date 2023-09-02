---
title: Mysql Master/Slave模式实战
author: admin
type: post
date: 2009-07-02T16:10:26+00:00
excerpt: |
 1.master上授权给slave
 mysql>grant all on *.* to repadmin@'218.6.67.75' identified by 'backup';
 mysql>flush privileges;
 mysql>use abs;
 mysql>create table mysqlslave (status char(8));
 mysql>insert into mysqlslave values ('aaaa');

 2.shutdown master
 mysqladmin -u root shutdown
url: /archives/1947
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - mysql

---
**1.master上授权给slave
** mysql>grant all on \*.\* to repadmin@’218.6.67.75′ identified by ‘backup’;
mysql>flush privileges;
mysql>use abs;
mysql>create table mysqlslave (status char(8));
mysql>insert into mysqlslave values (‘aaaa’);

**2.shutdown master**
mysqladmin -u root shutdown

**3.拷贝数据文件**
直接把数据文件夹打包拷贝到slave去。

**4.修改Master的my.cnf文件，在[mysqld]处增加**
master /etc/my.cnf:
log-bin
server-id = 1
sql-bin-update-same
binlog-do-db = abs

**5.修改Slave的my.cnf文件**
server-id       = 2
master-host     = 218.6.67.68
master-user     = backup
master-password = backup
master-port     = 3306
master-connect-retry    = 60
replicate-wild-do-table= ads.%

**6.启动slave**

**7.启动master**

**8.测试**
向其中的测试表里插入一条记录,如
use ads;
insert into mysqlslave values  ((CURDATE() + 0));
再在slave里查看是否有此记录

**9.问题**
a)ERROR1062  Duplicate entry
mysql> slave stop;
mysql> set GLOBAL SQL\_SLAVE\_SKIP_COUNTER=1;
mysql> slave start;
Use the value 1 for any SQL statement that does not use AUTO\_INCREMENT or LAST\_INSERT\_ID(), otherwise you will need to use the value 2. Statements that use AUTO\_INCREMENT or LAST\_INSERT\_ID() take up 2 events in the binary log.

b)调试命令
show processlist;
slave stop;
show slave status;
show master status;
flush master;
flush slave;
reset slave;
reset master;
slave start;
set global sql\_slave\_skip_counter=1;

**参考资料**
介绍几个管理Replication的命令：

 **1. PURGE MASTER LOG**
Replication需要生成大量的二进制文件，用以记录Client在Master上的操作，日积月累，这些文件会占据相当大的空间，可以用PURGE MASTER LOG命令来删除它们。

mysql> SHOW MASTER LOGS;
+—————-+
| Log_name |
+—————-+
| binary-log.001 |
| binary-log.002 |
| binary-log.003 |
| binary-log.004 |
+—————-+
4 rows in set (0.02 sec)

mysql> PURGE MASTER LOGS TO ‘binary-log.004’;

之后binary-log.001至binary-log.003三个文件都将被删除。

**2. SQL\_SLAVE\_SKIP_COUNTER**
如果Replication在Slave上出现错误而停止，一般都期望Slave能忽略这个错误，继续进行同步，而不是重新启动Slave。

> In MySQL 3.23.xx:
> mysql> SET SQL\_SLAVE\_SKIP_COUNTER=1
> mysql> SLAVE START
>
> In Versions 4.0.0-4.0.2:
> mysql> SET SQL\_SLAVE\_SKIP_COUNTER=1
> mysql> SLAVE START SQL_THREAD
>
> In Version 4.0.3 and beyond:
> mysql> SET GLOBAL SQL\_SLAVE\_SKIP_COUNTER=1
> mysql> SLAVE START SQL_THREAD