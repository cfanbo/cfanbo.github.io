---
title: mysql-Innodb事务隔离级别-repeatable read详解(转)
author: admin
type: post
date: 2015-03-31T00:14:15+00:00
url: /archives/15549
categories:
 - MySQL
tags:
 - innodb

---
[https://www.cnblogs.com/zhoujinyi/p/3437475.html](https://www.cnblogs.com/zhoujinyi/p/3437475.html)

其中对“幻读”进行了详情说明，以前对这个词的理解有些偏差。

**④ 幻读:**第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样。

```
mysql>CREATE TABLE `t_bitfly` (
`id` bigint(20) NOT NULL default '0',
`value` varchar(32) default NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB

mysql> select @@global.tx_isolation, @@tx_isolation;
+-----------------------+-----------------+
| @@global.tx_isolation | @@tx_isolation  |
+-----------------------+-----------------+
| REPEATABLE-READ       | REPEATABLE-READ |
+-----------------------+-----------------+

实验一：

t Session A                   Session B
|
| START TRANSACTION;          START TRANSACTION;
|
| SELECT * FROM t_bitfly;
| empty set
|                             INSERT INTO t_bitfly
|                             VALUES (1, 'a');
|
| SELECT * FROM t_bitfly;
| empty set
|                             COMMIT;
|
| SELECT * FROM t_bitfly;
| empty set
|
| INSERT INTO t_bitfly VALUES (1, 'a');
| ERROR 1062 (23000):
| Duplicate entry '1' for key 1
v (shit, 刚刚明明告诉我没有这条记录的)

如此就出现了幻读，以为表里没有数据，其实数据已经存在了，傻乎乎的提交后，才发现数据冲突了。

实验二：

t Session A                  Session B
|
| START TRANSACTION;         START TRANSACTION;
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
|                            INSERT INTO t_bitfly
|                            VALUES (2, 'b');
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
|                            COMMIT;
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
|
| UPDATE t_bitfly SET value='z';
| Rows matched: 2  Changed: 2  Warnings: 0
| (怎么多出来一行)
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | z     |
| |    2 | z     |
| +------+-------+

```

本事务中第一次读取出一行，做了一次更新后，另一个事务里提交的数据就出现了。也可以看做是一种幻读。
当隔离级别是可重复读，且禁用innodb_locks_unsafe_for_binlog的情况下，在搜索和扫描index的时候使用的next-key locks可以避免幻读。