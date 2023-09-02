---
title: mysql explain 中key_len的计算方法
author: admin
type: post
date: 2018-08-04T06:12:42+00:00
url: /archives/18018
categories:
 - MySQL
tags:
 - mysql
 - mysql优化

---
建议先阅读这篇文章： [http://hidba.org/?p=404](http://hidba.org/?p=404)

下面我们只对其中提到的做一个验证。

> (1).索引字段的附加信息：可以分为变长和定长数据类型讨论，当索引字段为定长数据类型，比如**char**，**int**，**datetime**，需要有是否为空的标记，这个标记需要占用1个字节；对于变长数据类型，比如：varchar，除了是否为空的标记外，还需要有长度信息，需要占用2个字节；
>
> (备注：当字段定义为非空的时候，是否为空的标记将不占用字节)
>
> (2).同时还需要考虑表所使用的字符集，不同的字符集，gbk编码的为一个字符2个字节，utf8编码的一个字符3个字节， utf8mb4 编码则是4个字节;

每种MySQL数据类型的定义参考：

下面我们以定长数据类型准，变长数据类型请自行测试。

**一、数据索引类型允许为null的情况：**

表结构：

```
CREATE TABLE `tb` (
`id` int(10) unsigned NOT NULL AUTO_INCREMENT,
`sid` smallint(5) DEFAULT NULL,
`gid` smallint(5) DEFAULT NULL,
PRIMARY KEY (`id`),
KEY `idx_common` (`sid`,`gid`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
```

执行分析语句：

```
mysql> EXPLAIN select * from tb where sid=1 and gid=5;
+----+-------------+-------+------------+------+---------------+------------+---------+-------------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key        | key_len | ref         | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------------+---------+-------------+------+----------+-------------+
|  1 | SIMPLE      | tb    | NULL       | ref  | idx_common    | idx_common | 6       | const,const |    1 |   100.00 | Using index |
+----+-------------+-------+------------+------+---------------+------------+---------+-------------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

```

发现用到了复合索引idx\_common,这时复合索引的两个字段全部用到了，而由于 smallint 数据类型占用字节为两个字节, 属于定长类型，且允许为null,所以key\_len长度计算公式为 **(2 + 1) + (2 + 1) = 6**
下面我们将两个字段全部禁止null看一下计算值

**二、数据索引类型不允许为null的情况**
表结构

```
CREATE TABLE `tb` (
`id` int(10) unsigned NOT NULL AUTO_INCREMENT,
`sid` smallint(5) NOT NULL,
`gid` smallint(5) NOT NULL,
PRIMARY KEY (`id`),
KEY `idx_common` (`sid`,`gid`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
```

```
mysql> EXPLAIN select * from tb where sid=1 and gid=5;
+----+-------------+-------+------------+------+---------------+------------+---------+-------------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key        | key_len | ref         | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------------+---------+-------------+------+----------+-------------+
|  1 | SIMPLE      | tb    | NULL       | ref  | idx_common    | idx_common | 4       | const,const |    1 |   100.00 | Using index |
+----+-------------+-------+------------+------+---------------+------------+---------+-------------+------+----------+-------------+

```

可以看到key_len的长度为4,即2 + 2 = 4

这里同样是复合索引中的字段全部用到，我们可以先测试一下用到一个字段的情况，依据左前缀索引原则

```
mysql> EXPLAIN select * from tb where sid=1;
+----+-------------+-------+------------+------+---------------+------------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key        | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | tb    | NULL       | ref  | idx_common    | idx_common | 2       | const |    2 |   100.00 | Using index |
+----+-------------+-------+------------+------+---------------+------------+---------+-------+------+----------+-------------+

```

发现key_len的值为2，就是说明只用到了一个复合索引字段，这里指的是sid字段。

**说明：** 一般情况下如果key的值越大越好，说明了充分利用到了我们创建的索引。

对于char或者varchar类型则可能自行测试！

推荐阅读： [http://imysql.com/2017/08/08/quick-deep-into-mysql-index.shtml](http://imysql.com/2017/08/08/quick-deep-into-mysql-index.shtml)