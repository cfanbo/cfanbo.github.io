---
title: mysql limit查询优化
author: admin
type: post
date: 2008-12-25T10:04:02+00:00
excerpt: |
 MYSQL的优化是非常重要的。其他最常用也最需要优化的就是limit。mysql的limit给分页带来了极大的方便，但数据量一大的时候，limit的性能就急剧下降。

 同样是取10条数据

 select * from yanxue8_visit limit 10000,10 和
 select * from yanxue8_visit limit 0,10
 就不是一个数量级别的。
url: /archives/743
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql
 - 查询优化

---

MYSQL的优化是非常重要的。其他最常用也最需要优化的就是limit。mysql的limit给分页带来了极大的方便，但数据量一大的时候，limit的性能就急剧下降。

同样是取10条数据


select * from yanxue8_visit limit 10000,10 和

select * from yanxue8_visit limit 0,10

就不是一个数量级别的。


网上也很多关于limit的五条优化准则，都是翻译自mysql手册，虽然正确但不实用。今天发现一篇文章写了些关于limit优化的，很不错。


文中不是直接使用limit，而是首先获取到offset的id然后直接使用limit size来获取数据。根据他的数据，明显要好于直接使用limit。这里我具体使用数据分两种情况进行测试。（测试环境win2033+p4双核 (3GHZ) +4G内存 mysql 5.0.19）


**1、offset比较小的时候。**

select * from yanxue8_visit limit 10,10


多次运行，时间保持在0.0004-0.0005之间

Select * From yanxue8_visit Where vid ＞=(

Select vid From yanxue8_visit Order By vid limit 10,1

) limit 10


多次运行，时间保持在0.0005-0.0006之间，主要是0.0006

结论：偏移offset较小的时候，直接使用limit较优。这个显然是子查询的原因。


**2、offset大的时候。**

select * from yanxue8_visit limit 10000,10


多次运行，时间保持在0.0187左右

Select * From yanxue8_visit Where vid ＞=(

Select vid From yanxue8_visit Order By vid limit 10000,1

) limit 10


多次运行，时间保持在0.0061左右，只有前者的1/3。可以预计offset越大，后者越优。


以后要注意改正自己的limit语句，优化一下mysql了.


注意这里的条件是没有where条件的情况下,对于有where条件的情况下请参考另一篇文章: [http://blog.haohtml.com/archives/3747](http://blog.haohtml.com/archives/3747)