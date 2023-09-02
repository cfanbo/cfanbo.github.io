---
title: 用mysql中的join来优化查询
author: admin
type: post
date: 2010-01-24T09:07:41+00:00
excerpt: |
 Mysql4.1开始支持SQL的子查询。这个技术可以使用SELECT语句来创建一个单列的查询结果，然后把这个结果作为过滤条件用在另一个查询中。使用子查询可以一次性的完成很多逻辑上需要多个步骤才能完成的SQL操作，同时也可以避免事务或者表锁死，并且写起来也很容易。但是，有些情况下，子查询可以被更有效率的连接(JOIN).. 替代。
 连接(JOIN)..之所以更有效率一些，是因为MySQL不需要在内存中创建临时表来完成这个逻辑上的需要两个步骤的查询工作。
url: /archives/2880
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql
 - mysql优化

---
Mysql4.1开始支持SQL的子查询。这个技术可以使用SELECT语句来创建一个单列的查询结果，然后把这个结果作为过滤条件用在另一个查询中。使用子查询可以一次性的完成很多逻辑上需要多个步骤才能完成的SQL操作，同时也可以避免事务或者表锁死，并且写起来也很容易。但是，有些情况下，子查询可以被更有效率的连接(JOIN).. 替代。

假设我们要将所有没有订单记录的用户取出来，可以用下面这个查询完成：

SELECT * FROM customerinfo WHERE CustomerID NOT in (SELECT CustomerID  FROM salesinfo)

如果使用连接(JOIN)..来完成这个查询工作，速度将会快很多。尤其是当salesinfo 表中对CustomerID建有索引的话，性能将会更好，查询如下：

SELECT * FROM customerinfo
LEFT JOIN salesinfo ON customerinfo.CustomerID=salesinfo.CustomerID
WHERE salesinfo.CustomerID IS NULL

连接(JOIN)..之所以更有效率一些，是因为MySQL不需要在内存中创建临时表来完成这个逻辑上的需要两个步骤的查询工作。