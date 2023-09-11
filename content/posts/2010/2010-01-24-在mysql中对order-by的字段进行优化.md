---
title: 在mysql中对order by的字段进行优化
author: admin
type: post
date: 2010-01-24T09:02:56+00:00
excerpt: |
 |
 在某些情况中，MySQL可以使用一个索引来满足ORDERBY子句，而不需要额外的排序。where条件和orderby使用相同的索引，并且orderby的顺序和索引顺序相同， 并且orderby的字段都是升序或者都是降序。

 例如：下列sql可以使用索引。

 SELECT * FROM t1 ORDER BY key_part1,key_part2,...;

 SELECT * FROM t1 WHERE key_part1=1 ORDER BY key_part1 DESC,key_part2 DESC;

 SELECT * FROM t1 ORDER BY key_part1 DESC,key_part2 DESC;
url: /archives/2877
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql
 - mysql优化

---
在某些情况中，MySQL可以使用一个索引来满足ORDER BY子句，而不需要额外的排序。where条件和order by使用相同的索引，并且order by的顺序和索引顺序相同， 并且order by的字段都是升序或者都是降序。

例如：下列sql可以使用索引。

SELECT * FROM t1 ORDER BY key\_part1,key\_part2,…;

SELECT * FROM t1 WHERE key\_part1=1 ORDER BY key\_part1 DESC,key_part2 DESC;

SELECT * FROM t1 ORDER BY key\_part1 DESC,key\_part2 DESC;

**但是以下情况不使用索引： **

SELECT * FROM t1 ORDER BY key\_part1 DESC,key\_part2 ASC；

**–orderby的字段混合ASC和DESC**

**
**

SELECT * FROM t1 WHERE key2=constant ORDER BY key1；

 **–用于查询行的关键字与ORDERBY中所使用的不相同**

**
**

SELECT * FROM t1 ORDER BY key1,key2；

**–对不同的关键字使用ORDERBY**