---
title: SQL 的 MASTER到MASTER的主主循环同步
author: admin
type: post
date: 2011-02-14T01:53:11+00:00
url: /archives/7701
IM_contentdowned:
 - 1
categories:
 - MySQL

---
注意在进行配置前,请确保相应的3306端口可以端口: [http://blog.haohtml.com/archives/7726](http://blog.haohtml.com/archives/7726)

刚刚抽空做了一下MYSQL 的主主同步。
把步骤写下来，至于会出现的什么问题，以后随时更新。这里我同步的数据库是TEST
 **1、环境描述。**
主机：192.168.0.231（A）
主机：192.168.0.232（B）
MYSQL 版本为5.1.21
 **2、授权用户。**
 A：
mysql> grant replication slave,file on \*.\* to ‘repl1’@’192.168.0.232’ identified by ‘123456’;
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
 B：
mysql> grant replication slave,file on \*.\* to ‘repl2’@’192.168.0.231’ identified by ‘123456’;
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
然后都停止MYSQL 服务器。

**3、配置文件。**
在两个机器上的my.cnf里面都开启二进制日志 。
 A：
user = mysql
log-bin=mysql-bin
server-id       = 1
binlog-do-db=test
binlog-ignore-db=mysql
replicate-do-db=test
replicate-ignore-db=mysql
 log-slave-updates
slave-skip-errors=all
 sync_binlog=1

 auto_increment_increment=2

 auto_increment_offset=1B：
user = mysql
log-bin=mysql-bin
server-id       = 2
binlog-do-db=test
binlog-ignore-db=mysql
replicate-do-db=test
replicate-ignore-db=mysql
 log-slave-updates
slave-skip-errors=all
 sync_binlog=1

 auto_increment_increment=2

 auto_increment_offset=2

至于这些参数的说明具体看手册。
参数sync_binlog详解参考: [http://blog.haohtml.com/archives/7704](http://blog.haohtml.com/archives/7704)
红色的部分非常重要，如果一个MASTER 挂掉的话，另外一个马上接管。
紫红色的部分指的是服务器频繁的刷新日志。这个保证了在其中一台挂掉的话，日志刷新到另外一台。从而保证了数据的同步 。
 **4、重新启动MYSQL服务器。**
在A和B上执行相同的步骤

> [root@localhost ~]# /usr/local/mysql/bin/mysqld_safe &
> [1] 4264
> [root@localhost ~]# 071213 14:53:20 mysqld_safe Logging to ‘/usr/local/mysql/data/localhost.localdomain.err’.
> /usr/local/mysql/bin/mysqld_safe: line 366: [: -eq: unary operator expected
> 071213 14:53:20 mysqld_safe Starting mysqld daemon with databases from /usr/local/mysql/data

**5、进入MYSQL的SHELL。**
 A：
mysql> flush tables with read lockG
Query OK, 0 rows affected (0.00 sec)

mysql> show master statusG
\***\***\***\***\***\***\***\***\*\\*\* 1. row \*\*\***\***\***\***\***\***\***\****
File: mysql-bin.000007
Position: 528
Binlog\_Do\_DB: test
Binlog\_Ignore\_DB: mysql
1 row in set (0.00 sec)

B：
mysql> flush tables with read lock;
Query OK, 0 rows affected (0.00 sec)

mysql> show master statusG
\***\***\***\***\***\***\***\***\*\\*\* 1. row \*\*\***\***\***\***\***\***\***\****
File: mysql-bin.000004
Position: 595
Binlog\_Do\_DB: test
Binlog\_Ignore\_DB: mysql
1 row in set (0.00 sec)
然后备份自己的数据，保持两个机器的数据一致。
方法很多。完了后看下一步。
 **6、在各自机器上执行CHANGE MASTER TO命令。**
 A：
mysql> change master to
-> master_host=’192.168.0.232′,
-> master_user=’repl2′,
-> master_password=’123456′,
-> master\_log\_file=’mysql-bin.000004′,
-> master\_log\_pos=595;
Query OK, 0 rows affected (0.01 sec)
mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

B：
mysql> change master to
-> master_host=’192.168.0.231′,
-> master_user=’repl1′,
-> master_password=’123456′,
-> master\_log\_file=’mysql-bin.000007′,
-> master\_log\_pos=528;
Query OK, 0 rows affected (0.01 sec)
mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

**7、查看各自机器上的IO进程和 SLAVE进程是否都开启。**
 A：

mysql> show processlistG
\***\***\***\***\***\***\***\***\*\\*\* 1. row \*\*\***\***\***\***\***\***\***\****
Id: 2
User: repl
Host: 192.168.0.232:54475
db: NULL
Command: Binlog Dump
Time: 1590
 State: Has sent all binlog to slave; waiting for binlog to be updated
Info: NULL
\***\***\***\***\***\***\***\***\*\\*\* 2. row \*\*\***\***\***\***\***\***\***\****
Id: 3
User: system user
Host:
db: NULL
Command: Connect
Time: 1350
 State: Waiting for master to send event
Info: NULL
\***\***\***\***\***\***\***\***\*\\*\* 3. row \*\*\***\***\***\***\***\***\***\****
Id: 4
User: system user
Host:
db: NULL
Command: Connect
Time: 1149
 State: Has read all relay log; waiting for the slave I/O thread to update it
Info: NULL
\***\***\***\***\***\***\***\***\*\\*\* 4. row \*\*\***\***\***\***\***\***\***\****
Id: 5
User: root
Host: localhost
db: test
Command: Query
Time: 0
State: NULL
Info: show processlist
4 rows in set (0.00 sec)

B：

mysql> show processlistG
\***\***\***\***\***\***\***\***\*\\*\* 1. row \*\*\***\***\***\***\***\***\***\****
Id: 1
User: system user
Host:
db: NULL
Command: Connect
Time: 2130
State: Waiting for master to send event
Info: NULL
\***\***\***\***\***\***\***\***\*\\*\* 2. row \*\*\***\***\***\***\***\***\***\****
Id: 2
User: system user
Host:
db: NULL
Command: Connect
Time: 1223
 State: Has read all relay log; waiting for the slave I/O thread to update it
Info: NULL
\***\***\***\***\***\***\***\***\*\\*\* 3. row \*\*\***\***\***\***\***\***\***\****
Id: 4
User: root
Host: localhost
db: test
Command: Query
Time: 0
State: NULL
Info: show processlist
\***\***\***\***\***\***\***\***\*\\*\* 4. row \*\*\***\***\***\***\***\***\***\****
Id: 5
User: repl2
Host: 192.168.0.231:50718
db: NULL
Command: Binlog Dump
Time: 1398
 State: Has sent all binlog to slave; waiting for binlog to be updated
Info: NULL
4 rows in set (0.00 sec)

如果红色部分没有出现，检查DATA目录下的错误文件。

**8、释放掉各自的锁，然后进行插数据测试。**
mysql> unlock tables;
Query OK, 0 rows affected (0.00 sec)

插入之前两个机器表的对比：
 A：

mysql> show tables;
+—————-+
| Tables\_in\_test |
+—————-+
| t11_innodb     |
| t22            |
+—————-+
 B：

mysql> show tables;
+—————-+
| Tables\_in\_test |
+—————-+
| t11_innodb     |
| t22            |
+—————-+
从A机器上进行插入
 A：
mysql> create table t11_replicas
-> (id int not null auto_increment primary key,
-> str varchar(255) not null) engine myisam;
Query OK, 0 rows affected (0.01 sec)

mysql> insert into t11_replicas(str) values
-> (‘This is a master to master test table’);
Query OK, 1 row affected (0.01 sec)

mysql> show tables;
+—————-+
| Tables\_in\_test |
+—————-+
| t11_innodb     |
| t11_replicas   |
| t22            |
+—————-+
3 rows in set (0.00 sec)

mysql> select * from t11_replicas;
+—-+—————————————+
| id | str                                   |
+—-+—————————————+
|  1 | This is a master to master test table |
+—-+—————————————+
1 row in set (0.00 sec)

现在来看B机器：

mysql> show tables;
+—————-+
| Tables\_in\_test |
+—————-+
| t11_innodb     |
| t11_replicas   |
| t22            |
+—————-+
3 rows in set (0.00 sec)

mysql> select * from t11_replicas;
+—-+—————————————+
| id | str                                   |
+—-+—————————————+
|  1 | This is a master to master test table |
+—-+—————————————+
1 row in set (0.00 sec)

现在反过来从B机器上插入数据：
 B：

mysql> insert into t11_replicas(str) values(‘This is a test 2’);
Query OK, 1 row affected (0.00 sec)

mysql> select * from t11_replicas;
+—-+—————————————+
| id | str                                   |
+—-+—————————————+
|  1 | This is a master to master test table |
|  2 | This is a test 2                      |
+—-+—————————————+
2 rows in set (0.00 sec)
我们来看A
 A：
mysql> select * from t11_replicas;
+—-+—————————————+
| id | str                                   |
+—-+—————————————+
|  1 | This is a master to master test table |
|  2 | This is a test 2                      |
+—-+—————————————+
2 rows in set (0.00 sec)

好了。现在两个表互相为MASTER。

多MASTER自增字段冲突的问题。
具体文章见：
[http://dev.mysql.com/tech-resources/articles/advanced-mysql-replication.html](http://dev.mysql.com/tech-resources/articles/advanced-mysql-replication.html)

在邮件列表中看到有人讨论在线同步与忽略库与表的问题，具体看：
[http://dev.mysql.com/doc/refman/5.1/en/replication-rules.html](http://dev.mysql.com/doc/refman/5.1/en/replication-rules.html)

摘自: [http://blogold.chinaunix.net/u/29134/showart_441667.html](http://blogold.chinaunix.net/u/29134/showart_441667.html)