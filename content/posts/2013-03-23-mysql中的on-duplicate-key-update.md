---
title: mysql中的ON DUPLICATE KEY UPDATE
author: admin
type: post
date: 2013-03-23T08:01:48+00:00
url: /archives/13730
categories:
 - MySQL
tags:
 - mysql

---
INSERT INTO ON DUPLICATE KEY UPDATE 与 REPLACE INTO，两个命令可以处理重复键值问题，在实际上它之间有什么区别呢？
前提条件是这个表必须有一个唯一索引或主键。

1、REPLACE发现重复的先删除再插入，如果记录有多个字段，在插入的时候如果有的字段没有赋值，那么新插入的记录这些字段为空。
2、INSERT发现重复的是更新操作。在原有记录基础上，更新指定字段内容，其它字段内容保留。

这样REPLACE的操作成本要大于 insert  ON DUPLICATE KEY UPDATE>.  按照 odds 的 代码 ，按道理应该选用insert  ON DUPLICATE KEY UPDATE

部分测试如下
2个 都是 影响的数据栏: 2
时间: 0.000ms
mysql insert的几点操作(DELAYED 、IGNORE、ON DUPLICATE KEY UPDATE )

INSERT语法

INSERT \[LOW\_PRIORITY | DELAYED | HIGH\_PRIORITY\] \[IGNORE\]

[INTO] tbl\_name [(col\_name,…)]

VALUES ({expr | DEFAULT},…),(…),…

[ ON DUPLICATE KEY UPDATE col_name=expr, … ]

或：

INSERT \[LOW\_PRIORITY | DELAYED | HIGH\_PRIORITY\] \[IGNORE\]

[INTO] tbl_name

SET col_name={expr | DEFAULT}, …

[ ON DUPLICATE KEY UPDATE col_name=expr, … ]

或：

INSERT \[LOW\_PRIORITY | HIGH\_PRIORITY\] \[IGNORE\]

[INTO] tbl\_name [(col\_name,…)]

SELECT …

[ ON DUPLICATE KEY UPDATE col_name=expr, … ]

一、DELAYED 的使用

使用延迟插入操作

DELAYED调节符应用于INSERT和REPLACE语句。当DELAYED插入操作到达的时候，

服务器把数据行放入一个队列中，并立即给客户端返回一个状态信息，这样客户

端就可以在数据表被真正地插入记录之前继续进行操作了。如果读取者从该数据

表中读取数据，队列中的数据就会被保持着，直到没有读取者为止。接着服务器

开始插入延迟数据行（delayed-row）队列中的数据行。在插入操作的同时，服务器

还要检查是否有新的读取请求到达和等待。如果有，延迟数据行队列就被挂起，

允许读取者继续操作。当没有读取者的时候，服务器再次开始插入延迟的数据行。

这个过程一直进行，直到队列空了为止。

几点要注意事项：

· INSERT DELAYED应该仅用于指定值清单的INSERT语句。服务器忽略用于INSERT DELAYED…SELECT语句的DELAYED。

· 服务器忽略用于INSERT DELAYED…ON DUPLICATE UPDATE语句的DELAYED。

· 因为在行被插入前，语句立刻返回，所以您不能使用LAST\_INSERT\_ID()来获取AUTO\_INCREMENT值。AUTO\_INCREMENT值可能由语句生成。

· 对于SELECT语句，DELAYED行不可见，直到这些行确实被插入了为止。

· DELAYED在从属复制服务器中被忽略了，因为DELAYED不会在从属服务器中产生与主服务器不一样的数据。

注意，目前在队列中的各行只保存在存储器中，直到它们被插入到表中为止。这意味着，如果您强行中止了mysqld（例如，使用kill -9）

或者如果mysqld意外停止，则所有没有被写入磁盘的行都会丢失。

二、IGNORE的使用

IGNORE是MySQL相对于标准SQL的扩展。如果在新表中有重复关键字，

或者当STRICT模式启动后出现警告，则使用IGNORE控制ALTER TABLE的运行。

如果没有指定IGNORE，当重复关键字错误发生时，复制操作被放弃，返回前一步骤。

如果指定了IGNORE，则对于有重复关键字的行，只使用第一行，其它有冲突的行被删除。

并且，对错误值进行修正，使之尽量接近正确值。

insert ignore into tb(…) value(…)

这样不用校验是否存在了，有则忽略，无则添加

三、ON DUPLICATE KEY UPDATE的使用

如果您指定了ON DUPLICATE KEY UPDATE，并且插入行后会导致在一个UNIQUE索引或PRIMARY KEY中出现重复值，则执行旧行UPDATE。例如，如果列a被定义为UNIQUE，并且包含值1，则以下两个语句具有相同的效果：

mysql> INSERT INTO table (a,b,c) VALUES (1,2,3)

-> ON DUPLICATE KEY UPDATE c=c+1;

mysql> UPDATE table SET c=c+1 WHERE a=1;

如果行作为新记录被插入，则受影响行的值为1；如果原有的记录被更新，则受影响行的值为2。

注释：如果列b也是唯一列，则INSERT与此UPDATE语句相当：

mysql> UPDATE table SET c=c+1 WHERE a=1 OR b=2 LIMIT 1;

如果a=1 OR b=2与多个行向匹配，则只有一个行被更新。通常，您应该尽量避免对带有多个唯一关键字的表使用ON DUPLICATE KEY子句。

您可以在UPDATE子句中使用VALUES(col\_name)函数从INSERT…UPDATE语句的INSERT部分引用列值。换句话说，如果 没有发生重复关键字冲突，则UPDATE子句中的VALUES(col\_name)可以引用被插入的col_name的值。本函数特别适用于多行插入。 VALUES()函数只在INSERT…UPDATE语句中有意义，其它时候会返回NULL。

示例：

mysql> INSERT INTO table (a,b,c) VALUES (1,2,3),(4,5,6)

-> ON DUPLICATE KEY UPDATE c=VALUES(a)+VALUES(b);

本语句与以下两个语句作用相同：

mysql> INSERT INTO table (a,b,c) VALUES (1,2,3)

-> ON DUPLICATE KEY UPDATE c=3;

mysql> INSERT INTO table (a,b,c) VALUES (4,5,6)

-> ON DUPLICATE KEY UPDATE c=9;

当您使用ON DUPLICATE KEY UPDATE时，DELAYED选项被忽略。

总结：DELAYED 做为快速插入，并不是很关心失效性，提高插入性能。

ignore     只关注主键对应记录是不存在，无则添加，有则忽略。

ON DUPLICATE KEY UPDATE 在添加时操作，关注非主键列，注意与ignore的区别。有则更新指定列，无则添加。
posted on 2008-08-22 14:17 鱼有所思 阅读(3997) 评论(1)  编辑 收藏 引用 网摘 所属分类: MySQL

评论:
\# MySQL INSERT … ON DUPLICATE KEY UPDATE 2008-08-22 14:26 | 鱼有所思
INSERT … ON DUPLICATE KEY UPDATE，当插入的记录会引发主键冲突或者违反唯一约束时，则使用UPDATE更新旧的记录，否则插入新记录。

mysql> desc test;
+——-+————-+——+—–+———+——-+
| Field | Type | Null | Key | Default | Extra |
+——-+————-+——+—–+———+——-+
| uid | int(11) | NO | PRI | | |
| uname | varchar(20) | YES | | NULL | |
+——-+————-+——+—–+———+——-+
2 rows in set (0.00 sec)

mysql> select * from test;
+—–+——–+
| uid | uname |
+—–+——–+
| 1 | uname1 |
| 2 | uname2 |
| 3 | me |
+—–+——–+
3 rows in set (0.00 sec)

mysql> INSERT INTO test values ( 3,’insertName’ )
-> ON DUPLICATE KEY UPDATE uname=’updateName’;
Query OK, 2 rows affected (0.03 sec)

mysql> select * from test;+—–+————+
| uid | uname |
+—–+————+
| 1 | uname1 |
| 2 | uname2 |
| 3 | updateName |
+—–+————+
3 rows in set (0.00 sec)

mysql> create index i\_test\_uname on test(uname);
Query OK, 3 rows affected (0.20 sec)
Records: 3 Duplicates: 0 Warnings: 0

mysql> INSERT INTO test VALUES ( 1 , ‘uname2′) -> ON DUPLICATE KEY UPDATE uname=’update2records’;Query OK, 2 rows affected (0.00 sec)

mysql> select * from test;+—–+—————-+
| uid | uname |
+—–+—————-+
| 2 | uname2 |
| 1 | update2records |
| 3 | updateName |
+—–+—————-+
3 rows in set (0.00 sec)

插入时会与两条记录发生冲突，分别由主键和唯一索引引起。但最终只UPDATE了其中一条。这在手册中也说明了，有多个唯一索引（或者有键也有唯一索引）的情况下，不建议使用该语句。

转自：http://www.cnblogs.com/rockee/archive/2012/06/11/2544903.html