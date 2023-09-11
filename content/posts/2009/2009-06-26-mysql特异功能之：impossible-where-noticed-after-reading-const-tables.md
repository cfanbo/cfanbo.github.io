---
title: MySQL特异功能之：Impossible WHERE noticed after reading const tables
author: admin
type: post
date: 2009-06-26T01:03:36+00:00
excerpt: |
 用EXPLAIN看MySQL的执行计划时经常会看到Impossible WHERE noticed after reading const tables这句话，意思是说MySQL通过读取“const tables”，发现这个查询是不可能有结果输出的。比如对下面的表和数据：

 create table t (a int primary key, b int) engine = innodb;
 insert into t values(1, 1);
 insert into t values(3, 1);

 执行“EXPLAIN select * from t where a = 2”时就会输出“Impossible WHERE noticed after reading const tables”。
url: /archives/1910
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - EXPLAIN
 - mysql

---
用EXPLAIN看MySQL的执行计划时经常会看到Impossible WHERE noticed after reading const tables这句话，意思是说MySQL通过读取“const tables”，发现这个查询是不可能有结果输出的。比如对下面的表和数据：

```
  create table t (a int primary key, b int) engine = innodb;
  insert into t values(1, 1);
  insert into t values(3, 1);
```

执行“EXPLAIN select * from t where a = 2”时就会输出“Impossible WHERE noticed after reading const tables”。

不 明白所谓的“const tables”是什么意思，对MySQL在查询优化时竟然可以发现一个查询不可能输出结果更是感觉不可思议。按数据库中“传统”的做法，查询优化时只会访 问模式定义和统计信息，而据我所知，数据库中使用的各种统计信息如EquiDepth、MaxDiff柱状图，MCV，属性的最大值、最小值等都不可能精 确到能够断言在上述的表中不存在“a = 2”的记录。

今天看MySQL Internal手册时才总算弄明白，原来MySQL并没有什么神奇之处，这个Impossible WHERE noticed after reading const tables的结论并不是通过统计信息做出的，而是真的去实际访问了一遍数据后，发现确实没有“a = 2”的行才得出的。

当查询中对某个表指定了主键或非空唯一索引上的等值条件，从而使得最多只可能产生一条命中结果（只对该表而言）时，MySQL在EXPLAIN之前会优先根据这一条件查找出对应的记录，并用记录的实际值替换查询中所有用到来自该表的属性的地方。一个更复杂的例子如下：

```
  explain select * from t as t1, t as t2 where t1.a = 1 and t2.a = t1.b + 1;

的输出结果为（由于排版关系省略了一些输出内容）：
+----+...+-----------------------------------------------------+
| id | ... | Extra                                               |
+----+...+-----------------------------------------------------+
|  1 | ... | Impossible WHERE noticed after reading const tables |
+----+...+-----------------------------------------------------+
```

MySQL得出上述查询不会输出结果的步骤如下：
1、首先根据t1.a = 1条件找到一条记录（1,1）;
2、将上述记录中b的值1替换查询中的t1.b，即将上述查询转化为等价的“explain select 1, 1,t2.a, t2.b from t as t2 where t2.a = 1 + 1”；
3、优化器计算常量表达式的值，即计算1+1得出结果为2；
4、优化器根据t2.a = 2条件查找，发现没有命中记录；
5、优化器最终打断出上述查询不可能输出结果。

说 白了，这个“Impossible WHERE noticed after reading const tables”就不再神秘了。但从这件事，我更加感觉到MySQL是个“怪怪”的数据库，有很多地方跟惯常的做法不太一样。很多数据库会在联接时将指定了 唯一索引等值条件的表优先执行，作为查询执行的第一步，但据我所知只有MySQL将这一步骤提前到查询优化的第一步来做。这么做到底在什么情况下才有好处 好像是个很微妙的问题，对于本文中给出的这两个例子，在优化时还是执行时做这一步开销都没什么区别。不过这么做好像没什么坏处。

这么会导 致一个“怪怪”的现象，那就是EXPLAIN有时候也会被阻塞。比如“EXPLAIN select * from t where a = 2 lock in share mode”，同时又有另一个事务插入了一条a = 2的记录而没有提交时，EXPLAIN就会在那里等锁。