---
title: MySQL中group_concat函数详解
author: admin
type: post
date: 2018-12-24T08:27:25+00:00
url: /archives/18665
categories:
 - MySQL
tags:
 - mysql

---
**函数语法：
**

**group_concat( [DISTINCT]  要连接的字段   [Order BY ****排序字段 ****ASC/DESC]   [Separator ‘分隔符’] )**

* * *

**下面举例说明：**

select * from goods;

+——+——+
| id| price|
+——+——+
|1 | 10|
|1 | 20|
|1 | 20|
|2 | 20|
|3 | 200 |
|3 | 500 |
+——+——+
6 rows in set (0.00 sec)

* * *

 1. 以id分组，把price字段的值在同一行打印出来，逗号分隔(默认)


select id, group_concat(price) from goods group by id;

+——+——————–+
| id| group_concat(price) |
+——+——————–+
|1 | 10,20,20|
|2 | 20 |
|3 | 200,500|
+——+——————–+
3 rows in set (0.00 sec)

* * *

2. 以id分组，把price字段的值在一行打印出来，分号分隔
select id,group_concat(price separator ‘;’) from goods group by id;

+——+———————————-+
| id| group_concat(price separator ‘;’) |
+——+———————————-+
|1 | 10;20;20 |
|2 | 20|
|3 | 200;500 |
+——+———————————-+
3 rows in set (0.00 sec)

* * *

3. 以id分组，把去除重复冗余的price字段的值打印在一行，逗号分隔
select id,group_concat(distinct price) from goods group by id;

+——+—————————–+
| id| group_concat(distinct price) |
+——+—————————–+
|1 | 10,20|
|2 | 20 |
|3 | 200,500 |
+——+—————————–+
3 rows in set (0.00 sec)

* * *

4. 以id分组，把price字段的值打印在一行，逗号分隔，按照price倒序排列
select id,group_concat(price order by price desc) from goods group by id;

+——+—————————————+
| id| group_concat(price order by price desc) |
+——+—————————————+
|1 | 20,20,10 |
|2 | 20|
|3 | 500,200|
+——+—————————————+
3 rows in set (0.00 sec)