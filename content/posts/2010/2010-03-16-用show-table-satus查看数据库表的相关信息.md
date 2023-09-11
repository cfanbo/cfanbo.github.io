---
title: 用show table satus查看数据库表的相关信息
author: admin
type: post
date: 2010-03-16T06:27:01+00:00
excerpt: |
 显然，我之前那个数据库备份程序太菜鸟了。还用select count(*) 来判断数据库的记录集数，其实完全不用那么麻烦。有一个简单的MML语句今天被我无意间发 现，"SHOW TABLE STATUS FROM `dbname`"，执行后会生成一个记录集。

 记录集包含下列字段：

 Name: newdb （表名）
 Engine: MyISAM （表引擎）
 Version: 10 （版本）
 Row_format: Dynamic （行格式）

 Rows: 56496 （表内总行数）
 对于其它存储引擎，比如InnoDB，本值是一个大约的数，与实际值相差可达40到50％。 在这些情况下，使用SELECT COUNT(*)来获得准确的数目。对于在INFORMATION_SCHEMA数 据库中的表，Rows值为NULL
url: /archives/2966
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---
显然，我之前那个数据库备份程序太菜鸟了。还用select count(*) 来判断数据库的记录集数，其实完全不用那么麻烦。有一个简单的MML语句今天被我无意间发 现，”SHOW TABLE STATUS FROM \`dbname\`”，执行后会生成一个记录集。

记录集包含下列字段：

Name: **newdb** （表名）
Engine: **MyISAM** （表引擎）
Version: **10** （版本）
Row_format: **Dynamic** （行格式）

Rows: **56496** （表内总行数）
对于其它存储引擎，比如InnoDB，本值是一个大约的数，与实际值相差可达40到50％。 在这些情况下，使用SELECT COUNT(*)来获得准确的数目。对于在INFORMATION_SCHEMA数 据库中的表，Rows值为NULL

Avg\_row\_length: **4749** （平均每行大小，这里是4.7K）
Data_length: **268352680** （该表总大小，单位字节）
Max\_data\_length: **281474976710655** （该表可存储上限，这个值所有表都一样，换算出来是256TB，应该用不到那么大吧）
如果给定了数据指针的大小，这是可以被存储在表中的数据的字节总数。mysql5以后的版本所能支持的最大存储容量是非常大的，上面的 列子为256TB。这时表的最大存储容量主要受限于OS了。不过到现在，Linux ext3 fs支持单个最大文件尺寸2T了，所以基本这个问题不必过于担心了。存储引擎是innodb的话，这个值在show table status显示的值总是为0，不知道为什么。

Index_length: **2464768** （索引大小）
Data_free: **** （数据多余,即数据碎片）
Auto_increment: **69296** （自动累加ID 6W9，而前面的行数只有5W6，说明我有删掉了1W3笔数据）
Create_time: **2009-10-13 10:46:14** （这个不用说了吧）
Update_time: **2009-10-13 11:08:15**
Check_time:　什么时候表被最后一次检查。不是所有的存储引擎此时都更新，在此情况下，值为NULL。
Collation: **gb2312\_chinese\_ci** （编码）
Checksum: 活性校验和值
Create_options: **row_format=DYNAMIC** （为什么要重复一遍？）
Comment: （表注释）