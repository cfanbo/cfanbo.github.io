---
title: mysql中innodb表的count优化
author: admin
type: post
date: 2013-07-10T03:33:57+00:00
url: /archives/14096
categories:
 - MySQL
tags:
 - innodb
 - 查询优化

---

作/译者：叶金荣（imysql#imysql.com>），来源： [http://imysql.com](http://imysql.com/)，欢迎转载。


起因：在innodb表上做count(*)统计实在是太慢了，因此想办法看能不能再快点。

现象：先来看几个测试案例，如下

**一、 sbtest 表上的测试**

```
show create table sbtest\G
*************************** 1. row ***************************
Table: sbtest
Create Table: CREATE TABLE `sbtest` (
`aid` bigint(20) unsigned NOT NULL auto_increment,
`id` int(10) unsigned NOT NULL default '0',
`k` int(10) unsigned NOT NULL default '0',
`c` char(120) NOT NULL default '',
`pad` char(60) NOT NULL default '',
PRIMARY KEY  (`aid`),
KEY `k` (`k`),
KEY `id` (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1000001 DEFAULT CHARSET=latin1

show index from sbtest;
+--------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+
| Table  | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment |
+--------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+
| sbtest |          0 | PRIMARY  |            1 | aid         | A         |     1000099 |     NULL | NULL   |      | BTREE      |         |
| sbtest |          1 | k        |            1 | k           | A         |          18 |     NULL | NULL   |      | BTREE      |         |
| sbtest |          1 | id       |            1 | id          | A         |     1000099 |     NULL | NULL   |      | BTREE      |         |
+--------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+
```

填充了 100万条 记录。


show index from 表名介绍： [http://blog.haohtml.com/archives/1889](http://blog.haohtml.com/archives/1889)

**1、 直接 count(*)**

```
explain SELECT COUNT(*) FROM sbtest;
+----+-------------+--------+-------+---------------+---------+---------+------+---------+-------------+
| id | select_type | table  | type  | possible_keys | key     | key_len | ref  | rows    | Extra       |
+----+-------------+--------+-------+---------------+---------+---------+------+---------+-------------+
|  1 | SIMPLE      | sbtest | index | NULL          | PRIMARY | 8       | NULL | 1000099 | Using index |
+----+-------------+--------+-------+---------------+---------+---------+------+---------+-------------+
SELECT COUNT(*) FROM sbtest;
+----------+
| COUNT(*) |
+----------+
|  1000000 |
+----------+
1 row in set (1.42 sec)
```

可以看到，如果不加任何条件，那么优化器优先采用 primary key 来进行扫描。

**2、count(*) 使用 primary key 字段做条件**

```
explain SELECT COUNT(*) FROM sbtest WHERE aid>=0;
+----+-------------+--------+-------+---------------+---------+---------+------+--------+--------------------------+
| id | select_type | table  | type  | possible_keys | key     | key_len | ref  | rows   | Extra                    |
+----+-------------+--------+-------+---------------+---------+---------+------+--------+--------------------------+
|  1 | SIMPLE      | sbtest | range | PRIMARY       | PRIMARY | 8       | NULL | 485600 | Using where; Using index |
+----+-------------+--------+-------+---------------+---------+---------+------+--------+--------------------------+
SELECT COUNT(*) FROM sbtest WHERE aid>=0;
+----------+
| COUNT(*) |
+----------+
|  1000000 |
+----------+
1 row in set (1.39 sec)
```

可以看到，尽管优化器认为只需要扫描 485600 条记录(其实是索引)，比刚才少多了，但其实仍然要做全表(索引)扫描。因此耗时和第一种相当。


**3、 count(*) 使用 辅助索引secondary index 字段做条件**

```
explain SELECT COUNT(*) FROM sbtest WHERE id>=0;
+----+-------------+--------+-------+---------------+------+---------+------+--------+--------------------------+
| id | select_type | table  | type  | possible_keys | key  | key_len | ref  | rows   | Extra                    |
+----+-------------+--------+-------+---------------+------+---------+------+--------+--------------------------+
|  1 | SIMPLE      | sbtest | range | id            | id   | 4       | NULL | 500049 | Using where; Using index |
+----+-------------+--------+-------+---------------+------+---------+------+--------+--------------------------+
SELECT COUNT(*) FROM sbtest WHERE id>=0;
+----------+
| COUNT(*) |
+----------+
|  1000000 |
+----------+
1 row in set (0.43 sec)
```

可以看到，采用这种方式查询会非常快。

有人也许会问了，会不会是因为 id 字段的长度比 aid 字段的长度来的小，导致它扫描起来比较快呢？先不着急下结论，咱们来看看下面的测试例子。

**二、 sbtest1 表上的测试**

```
show create table sbtest1\G
*************************** 1. row ***************************
Table: sbtest1
Create Table: CREATE TABLE `sbtest1` (
`aid` int(10) unsigned NOT NULL AUTO_INCREMENT,
`id` bigint(20) unsigned NOT NULL DEFAULT '0',
`k` int(10) unsigned NOT NULL DEFAULT '0',
`c` char(120) NOT NULL DEFAULT '',
`pad` char(60) NOT NULL DEFAULT '',
PRIMARY KEY (`aid`),
KEY `k` (`k`),
KEY `id` (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1000001 DEFAULT CHARSET=latin1
show index from sbtest1;
+---------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+
| Table   | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment |
+---------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+
| sbtest1 |          0 | PRIMARY  |            1 | aid         | A         |     1000099 |     NULL | NULL   |      | BTREE      |         |
| sbtest1 |          1 | k        |            1 | k           | A         |          18 |     NULL | NULL   |      | BTREE      |         |
| sbtest1 |          1 | id       |            1 | id          | A         |     1000099 |     NULL | NULL   |      | BTREE      |         |
+---------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+
```

这个表里，把 aid 和 id 的字段长度调换了一下，也填充了 100万条 记录。

**1、 直接 count(*)**

```
explain SELECT COUNT(*) FROM sbtest1;
+----+-------------+---------+-------+---------------+---------+---------+------+---------+-------------+
| id | select_type | table   | type  | possible_keys | key     | key_len | ref  | rows    | Extra       |
+----+-------------+---------+-------+---------------+---------+---------+------+---------+-------------+
|  1 | SIMPLE      | sbtest1 | index | NULL          | PRIMARY | 4       | NULL | 1000099 | Using index |
+----+-------------+---------+-------+---------------+---------+---------+------+---------+-------------+
SELECT COUNT(*) FROM sbtest1;
+----------+
| COUNT(*) |
+----------+
|  1000000 |
+----------+
1 row in set (1.42 sec)
```

可以看到，如果不加任何条件，那么优化器优先采用 primary key 来进行扫描。

**2、count(*) 使用 primary key 字段做条件**

```
explain SELECT COUNT(*) FROM sbtest1 WHERE aid>=0;
+----+-------------+---------+-------+---------------+---------+---------+------+--------+--------------------------+
| id | select_type | table   | type  | possible_keys | key     | key_len | ref  | rows   | Extra                    |
+----+-------------+---------+-------+---------------+---------+---------+------+--------+--------------------------+
|  1 | SIMPLE      | sbtest1 | range | PRIMARY       | PRIMARY | 4       | NULL | 316200 | Using where; Using index |
+----+-------------+---------+-------+---------------+---------+---------+------+--------+--------------------------+
1 row in set (0.00 sec)
SELECT COUNT(*) FROM sbtest1 WHERE aid>=0;
+----------+
| COUNT(*) |
+----------+
|  1000000 |
+----------+
1 row in set (1.42 sec)
```

可以看到，尽管优化器认为只需要扫描 485600 条记录(其实是索引)，比刚才少多了，但其实仍然要做全表(索引)扫描。因此耗时和第一种相当。


**3、 count(*) 使用 secondary index 字段做条件**

```
explain SELECT COUNT(*) FROM sbtest1 WHERE id>=0;
+----+-------------+---------+-------+---------------+------+---------+------+--------+--------------------------+
| id | select_type | table   | type  | possible_keys | key  | key_len | ref  | rows   | Extra                    |
+----+-------------+---------+-------+---------------+------+---------+------+--------+--------------------------+
|  1 | SIMPLE      | sbtest1 | range | id            | id   | 8       | NULL | 500049 | Using where; Using index |
+----+-------------+---------+-------+---------------+------+---------+------+--------+--------------------------+
1 row in set (0.00 sec)
SELECT COUNT(*) FROM sbtest1 WHERE id>=0;
+----------+
| COUNT(*) |
+----------+
|  1000000 |
+----------+
1 row in set (0.45 sec)
```

可以看到，采用这种方式查询会非常快。

上面的所有测试，均在 mysql 5.1.24 环境下通过，并且每次查询前都重启了 mysqld。

可以看到，把 aid 和 id 的长度调换之后，采用 secondary index 查询仍然是要比用 primary key 查询来的快很多。看来主要不是字段长度引起的索引扫描快慢，而是采用 primary key 以及 secondary index 引起的区别。那么，为什么用 secondary index 扫描反而比 primary key 扫描来的要快呢？我们就需要了解innodb的 _clustered index_ 和 _secondary index_ 之间的区别了。

innodb 的 clustered index 是把 primary key 以及 row data 保存在一起的，而 secondary index 则是单独存放，然后有个指针指向 primary key。因此，需要进行 count(*) 统计表记录总数时，利用 secondary index 扫描起来，显然更快。而primary key则主要在扫描索引，同时要返回结果记录时的作用较大，例如：


```
SELECT * FROM sbtest WHERE aid = xxx;
```

那既然是使用 secondary index 会比 primary key 更快，为何优化器却优先选择 primary key 来扫描呢， **Heikki Tuuri** 的回答是：


```
in the example table, the secondary index is inserted into in a perfect order! That is
very unusual. Normally the secondary index would be fragmented, causing random disk I/O,
and the scan would be slower than in the primary index.
I am changing this to a feature request: keep 'clustering ratio' statistics on a secondary
index and do the scan there if the order is almost the same as in the primary index. I
doubt this feature will ever be implemented, though.
```

详情请看： [这个 bug](http://bugs.mysql.com/bug.php?id=19271)，以及这个文章： [InnoDB Row Counting using Indexes](http://optimmysql.blogspot.com/2007/07/innodb-row-counting-using-indexes.html)。

最后感谢 [老杨](http://blog.csdn.net/whyangwanfu/) 的帮助。


**技术相关:**

[MySQL优化](http://imysql.com/taxonomy/term/1)

[InnoDB](http://imysql.com/taxonomy/term/21)