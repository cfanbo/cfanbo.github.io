---
title: mysql中DATETIME、DATE 和 TIMESTAMP 类型数据使用
author: admin
type: post
date: 2007-08-13T13:48:44+00:00
url: /archives/96
IM_contentdowned:
 - 1
categories:
 - 数据库

---

`DATETIME`、`DATE` 和 `TIMESTAMP` 类型是相似的。这个章节描述了它们的特性以及它们的相似点与不同点。

`DATETIME` 类型可用于需要同时包含日期和时间信息的值。MySQL 以 `'YYYY-MM-DD HH:MM:SS'` 格式检索与显示 `DATETIME` 类型。支持的范围是 `'1000-01-01 00:00:00'` 到 `'9999-12-31 23:59:59'`。(“支持”的含义是，尽管更早的值可能工作，但不能保证他们均可以。)

`DATE` 类型可用于需要一个日期值而不需要时间部分时。MySQL 以 `'YYYY-MM-DD'` 格式检索与显示 `DATE` 值。支持的范围是 `'1000-01-01'` 到 `'9999-12-31'`。

`TIMESTAMP` 列类型提供了一种类型，通过它你可以以当前操作的日期和时间自动地标记 `Insert` 或`Update` 操作。如果一张表中有多个 `TIMESTAMP` 列，只有第一个被自动更新。

自动更新第一个 `TIMESTAMP` 列在下列任何条件下发生：

 * 列值没有明确地在一个 `Insert` 或 `LOAD DATA INFILE` 语句中被指定。
 * 列值没有明确地在一个 `Update` 语句中被指定，并且其它的一些列值已发生改变。(注意，当一个 `Update` 设置一个列值为它原有值时，这将不会引起 `TIMESTAMP` 列的更新，因为，如果你设置一个列值为它当前值时，MySQL 为了效率为忽略更新。)
 * 明确地以 `NULL` 设置 `TIMESTAMP` 列。

第一个列以外其它 `TIMESTAMP` 列，可以设置到当前的日期和时间，只要将该列赋值 `NULL` 或 `NOW()`。

任何 `TIMESTAMP` 列均可以被设置一个不同于当前操作日期与时间的值，这通过为该列明确指定一个你所期望的值来实现。这也适用于第一个 `TIMESTAMP` 列。这个选择性是很有用的，举例来说，当你希望 `TIMESTAMP` 列保存该记录行被新添加时的当前的日期和时间，但该值不再发生改变，无论以后是否对该记录行进行过更新：

 * 当该记录行被建立时，让 MySQL 设置该列值。这将初始化该列为当前日期和时间。
 * 以后当你对该记录行的其它列执行更新时，为 `TIMESTAMP` 列值明确地指定为它原来的值。

另一方面，你可能发现更容易的方法，使用 `DATETIME` 列，当新建记录行时以 `NOW()` 初始化该列，以后在对该记录行进行更新时不再处理它。

`示例(译者注)：`

```
mysql> Create TABLE `tA` (
->   `id` int(3) unsigned NOT NULL auto_increment,
->     `date1` timestamp(14) NOT NULL,
->     `date2` timestamp(14) NOT NULL,
->     PRIMARY KEY  (`id`)
-> ) TYPE=MyISAM;
Query OK, 0 rows affected (0.01 sec)
mysql> Insert INTO `tA` SET `id` = 1;
Query OK, 1 row affected (0.02 sec)
# 没有明确地指定第一个 timestamp 列值，该列值被设为插入的当前时刻
# 没有明确地指定其它的 timestamp 列值，MySQL 则认为插入的是一个非法值，而该列值被设为0
mysql> Insert INTO `tA` S (2, NOW(), NULL);
Query OK, 1 row affected (0.01 sec)
mysql> Select * FROM `tA`;
+----+----------------+----------------+
| id | date1          | date2          |
+----+----------------+----------------+
|  1 | 20030503104118 | 00000000000000 |
|  2 | 20030503104254 | 20030503104254 |
+----+----------------+----------------+
2 rows in set (0.00 sec)
mysql> Update `tA` SET `id` = 3 Where `id` = 1;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
# 对某一记录行进行了更新，第一个 timestamp 列值也将被更新
mysql> Update `tA` SET `id` = 2 Where `id` = 2;
Query OK, 0 rows affected (0.00 sec)
Rows matched: 1  Changed: 0  Warnings: 0
# MySQL 忽略了这次操作，第一个 timestamp 列值不会被更新
mysql> Select * FROM `tA`;
+----+----------------+----------------+
| id | date1          | date2          |
+----+----------------+----------------+
|  3 | 20030503104538 | 00000000000000 |
|  2 | 20030503104254 | 20030503104254 |
+----+----------------+----------------+
2 rows in set (0.00 sec)
mysql> Update `tA` SET `id` = 1,`date1`=`date1` Where `id` = 3;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
# 明确地指定了第一个 timestamp 列值为它原有值，该值将不会被更新
mysql> Select * FROM `tA`;
+----+----------------+----------------+
| id | date1          | date2          |
+----+----------------+----------------+
|  1 | 20030503104538 | 00000000000000 |
|  2 | 20030503104254 | 20030503104254 |
+----+----------------+----------------+
2 rows in set (0.00 sec)
* 以上结果在 MySQL 4.0.12 中测试

```