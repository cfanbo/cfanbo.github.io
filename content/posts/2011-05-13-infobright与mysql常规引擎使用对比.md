---
title: infobright与mysql常规引擎使用对比
author: admin
type: post
date: 2011-05-13T01:05:05+00:00
url: /archives/9445
IM_data:
 - 'a:2:{s:74:"http://60.29.242.49/wp-content/uploads/2010/03/image00103-29-10-45-541.jpg";s:83:"http://blog.haohtml.com/wp-content/uploads/2011/05/0a0e_image00103-29-10-45-541.jpg";s:73:"http://60.29.242.49/wp-content/uploads/2010/03/image00203-29-10-45-54.jpg";s:82:"http://blog.haohtml.com/wp-content/uploads/2011/05/7634_image00203-29-10-45-54.jpg";}'
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - Infobright
 - mysql

---
测试背景介绍 ：两台机器AB,A机器使用常规引擎innodb，B使用infobright，测试数据量10亿，平均分散到两台机器，基于各种因素，A的数据分成了24个表，即每小时一个。

1.infobright和myisampack的压缩性能对比：

数据加载完成后首先alter table XXX engine=myisam使用mysqlchk进行压缩，压缩后每天有45G左右的数据，infobright存储要7~8G，压缩性能差异近80%

2.infrobright和myisam查询效率对比：

两台机器上面执行相同的sql语句：select count(1),type from table_name group by type;

A（innodb）运行情况:![image001(03-29-10-45-54)](http://60.29.242.49/wp-content/uploads/2010/03/image00103-29-10-45-541.jpg)

B（infobright）运行情况:

![image002(03-29-10-45-54)](http://60.29.242.49/wp-content/uploads/2010/03/image00203-29-10-45-54.jpg)

由于innodb存储时需要改成myisam引擎并进行压缩，所以耗费了cpu不少资源，除此之外，mysql本身运行的资源消耗基本无区别。

在执行时间上，infobright耗时(3 min 31.37 sec) ，myisam耗时（1 min 45.38 sec)，但由于A是散成了24个表，所以耗时需要\*24，除去其他相关因素干扰，infobright的查询效率应该比innodb高至少5~6倍 ，考虑仅仅执行一个sql语句误差太大，又执行一个相同语句：select count(1) from table_name where type =’XXXX’; infobright（1 min 12.10 ），innodb(44.33 sec) \*24,结论相同。

**综上所述：**

infobright无论在压缩性能，执行效率还是在消耗系统资源上面都有优势。

个人觉得数据量大的一些存放历史数据的库，infobright基本可以完成查询要求，还可以节省80%的空间，由于免费版本不支持load以外的操作，所以此次测试都是建立在load的基础上进行的

摘自: