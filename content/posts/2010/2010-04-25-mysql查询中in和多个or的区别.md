---
title: mysql查询中in和多个or的区别
author: admin
type: post
date: 2010-04-25T13:51:33+00:00
url: /archives/3486
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql
 - 查询优化

---
**比较IN()里面的数据**
许多数据库服务器都只把IN()看作多个OR的同义词，因为它们在逻辑上是相等的。MYSQL不是这样的，它会对IN()里面的数据进行排序，然后用二分法查找个是否在列表中，这个算法的效率是Ｏ（Ｌogn),而等同的OR子句的查找效率是Ｏ(n)。在列表很大的时候，OR子句就会变得慢得多。

这里的语句和Oracle数据库里是一样的。