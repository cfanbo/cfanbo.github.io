---
title: MySQL8.0中的跳跃范围扫描优化Skip Scan Range Access Method介绍
author: admin
type: post
date: 2019-10-25T04:49:22+00:00
url: /archives/19291
categories:
 - MySQL
tags:
 - mysql优化

---
在MySQL8.0以前，索引使用规则有一项是索引左前缀，假如说有一个索引idx\_abc(a,b,c)，能用到索引的情况只有查询条件为a、ab、abc、ac这四种，对于只有字段b的where条件是无法用到这个idx\_abcf索引的。这里再强调一下，这里的顺序并不是在where中字段出现的顺序，where b=2 and 1=1 也是可以利用到索引的，只是用到了(a,b)这两个字段

针对这一点， 从MySQL 8.0.13开始引入了一种新的优化方案，叫做 **Skip Scan Range**，翻译过来的话是**跳跃范围扫描**。如何理解这个概念呢？我们可以拿官方的SQL示例具体讲一下（）

```
CREATE TABLE t1 (f1 INT NOT NULL, f2 INT NOT NULL, PRIMARY KEY(f1, f2));
INSERT INTO t1 VALUES
  (1,1), (1,2), (1,3), (1,4), (1,5),
  (2,1), (2,2), (2,3), (2,4), (2,5);
INSERT INTO t1 SELECT f1, f2 + 5 FROM t1;
INSERT INTO t1 SELECT f1, f2 + 10 FROM t1;
INSERT INTO t1 SELECT f1, f2 + 20 FROM t1;
INSERT INTO t1 SELECT f1, f2 + 40 FROM t1;
ANALYZE TABLE t1;

EXPLAIN SELECT f1, f2 FROM t1 WHERE f2 > 40;
```

我们这里创建了一个t1表，其中主键为(f1,f2)，这里是两个字段。执行完这个sql语句后表里有160条记录，执行计划为

```
mysql> EXPLAIN SELECT f1, f2 FROM t1 WHERE f2 > 40;
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+----------------------------------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra                                  |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+----------------------------------------+
|  1 | SIMPLE      | t1    | NULL       | range | PRIMARY       | PRIMARY | 8       | NULL |   53 |   100.00 | Using where; Using index for skip scan |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+----------------------------------------+
1 row in set, 1 warning (0.00 sec)
```

这里可以看到 type 为 rang,说明用到了范围查询，key为 PRIMARY, Extra中 Using where; **Using index for skip scan**。

说明确实用到了新特性 skip scan。

那么在MySQL内部这个 skip scan 它又是如何执行的呢，我们可以理解以下几步

 1. 先统计一下索引前缀字段 f1 字段值有几个唯一值，这里一共有1 和2
 2. 对其余索引部分上的f2> 40条件的每个不同的前缀值执行子范围扫描

对于详细的执行流程如下：

 1. 获取f1的第一个唯一值（f1=1)
 2. 组合能用到索引的sql语句(f1=1 AND f2>40)
 3. 执行组合后的sql语句，进行范围扫描，并将结果放入记录集
 4. 重复上面的步骤，获取f1的第二个唯一值(f1=2)
 5. 组合能用到索引的sql语句(f1=2 AND f2>40)
 6. 执行组合后的sql语句，进行范围扫描，并将结果放入记录集
 7. 全部执行完毕，返回记录集给客户端

不错，原理很简单，就是将f1字段拆分成不同的值，将每个值带入到适合左前缀索引的SQL语句中，最后再合并记录集并返回即可，类似UNION操作。够简单吧！

但有同学可能会问，是所有的查询都不会执行这个优化吗？答案是否定的，主要还要看左前缀有字段值的分散情况，如果值过多的话，性能还是比较差的。系统会进行全表扫描，这里就需要单独为这个字段创建一个单独的索引。

skip scan特性虽好，但也有一些使用条件。

**skip scan触发条件**

(1)必须是联合索引

(2)只能是一个表

(3)不能使用distinct或group by ;

(4)SQL不能回表，即select列和where条件列都要包含在一个索引中

(5)默认optimizer\_switch=’skip\_scan=on’开启；