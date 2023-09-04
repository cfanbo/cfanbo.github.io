---
title: 关于MySQL explain 中的ID(推荐)
author: admin
type: post
date: 2010-06-27T09:51:27+00:00
url: /archives/4141
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - EXPLAIN
 - mysql
 - 查询优化

---
**Explain ID详解**

含义：select查询的序列号，是一组数字，表示的是查询中执行select子句或者是操作表的顺序。

**id的情况有三种，分别是：**

 * id相同表示加载表的顺序是从上到下。
 * id不同id值越大，优先级越高，越先被执行。
 * id有相同，也有不同，同时存在。id相同的可以认为是一组，从上往下顺序执行；在所有的组中，id的值越大，优先级越高，越先执行。

再看一个查询计划的例子：

[![](https://blogstatic.haohtml.com//uploads/2023/09/mysql_explain.png)][1]

**执行顺序依次为 4 -> 3 -> 2 > 1 > NULL**

第一行：id列为1，表示第一个select，select_type列的primary表示该查询为外层查询，table列被标记为，表示查询结果来自一个衍生表，其中3代表该查询衍生自第三个select查询，即id为3的select。[select d1.name……]

第二行：id为3，表示该查询的执行次序为2（4→3），是整个查询中第三个select的一部分。因查询包含在from中，所以为derived。[select id,name from t1 where other_column=”]

第三行：select列表中的子查询，select_type为subquery，为整个查询中的第二个select。[select id from t3]

第四行：select_type为union，说明第四个select是union里的第二个select，最先执行。[select name,id from t2]

第五行：代表从union的临时表中读取行的阶段，table列的表示用第一个和第四个select的结果进行union操作。[两个结果union操作]

**Extra:包含不适合在其他列中显示但十分重要的额外信息。**

Only index，这意味着信息只用索引树中的信息检索出的，这比扫描整个表要快。

Using where 是使用上了where限制，表示MySQL服务器在**存储引擎**收到记录后进行“后过滤”（Post-filter），如果查询未能使用索引，Using where的作用只是提醒我们MySQL将用where子句来过滤结果集。

impossible where 表示用不着where，一般就是没查出来啥。

Using filesort（MySQL中无法利用索引完成的排序操作称为“文件排序”）当我们试图对一个没有索引的字段进行排序时，就是filesoft。它跟文件没有任何关系，实际上是内部的一个快速排序。

Using temporary（表示MySQL需要使用临时表来存储结果集，常见于排序和分组查询），使用filesort和temporary的话会很吃力，WHERE和ORDER BY的索引经常无法兼顾，如果按照WHERE来确定索引，那么在ORDER BY时，就必然会引起Using filesort，这就要看是先过滤再排序划算，还是先排序再过滤划算。

[https://blog.csdn.net/xifeijian/article/details/19773795](https://blog.csdn.net/xifeijian/article/details/19773795)

[1]: https://blog.haohtml.com/wp-content/uploads/2010/06/mysql_explain.png