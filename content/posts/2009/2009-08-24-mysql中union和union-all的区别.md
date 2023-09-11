---
title: MySQL中UNION和UNION ALL的区别
author: admin
type: post
date: 2009-08-24T07:49:45+00:00
excerpt: |
 在数据库中，UNION和UNION ALL关键字都是将两个结果集合并为一个，但这两者从使用和效率上来说都有所不同。

 MySQL中的UNION
 UNION在进行表链接后会筛选掉重复的记录，所以在表链接后会对所产生的结果集进行排序运算，删除重复的记录再返回结果。实际大部分应用中是不会产生重复的记录，最常见的是过程表与历史表UNION。如：
 select * from gc_dfys union select * from ls_jg_dfys
url: /archives/2274
IM_contentdowned:
 - 1
 - 1
categories:
 - MySQL
tags:
 - mysql

---
在数据库中，UNION和UNION ALL关键字都是将两个结果集合并为一个，但这两者从使用和效率上来说都有所不同。

**MySQL中的UNION**
UNION在进行表链接后会筛选掉重复的记录，所以在表链接后会对所产生的结果集进行排序运算，删除重复的记录再返回结果。实际大部分应用中是不会产生重复的记录，最常见的是过程表与历史表UNION。如：

>

> `select * from gc_dfys union select * from ls_jg_dfys`
>

这个SQL在运行时先取出两个表的结果，再用排序空间进行排序删除重复的记录，最后返回结果集，如果表数据量大的话可能会导致用磁盘进行排序。

**MySQL中的UNION ALL**
而UNION ALL只是简单的将两个结果合并后就返回。这样，如果返回的两个结果集中有重复的数据，那么返回的结果集就会包含重复的数据了。
从效率上说，UNION ALL 要比UNION快很多，所以，如果可以确认合并的两个结果集中不包含重复的数据的话，那么就使用UNION ALL，如下：

> `select * from gc_dfys union all select * from ls_jg_dfys`