---
title: 从MyISAM转到InnoDB需要注意什么
author: admin
type: post
date: 2016-01-13T03:31:08+00:00
url: /archives/16472
categories:
 - MySQL
tags:
 - mysql

---
当前，绝大多数业务场景用InnoDB已经完全能搞定了，越来越多的业务从MyISAM转向InnoDB引擎，那么有哪些注意事项呢？ 分析 当了解完两种引擎的不同之处，很轻松的就能知道有哪些关键点了。

总的来说，从MyISAM转向InnoDB的注意事项有：

```
1、MyISAM的主键索引中，可以在非第一列(非第一个字段)使用自增列，而InnoDB的主键索引中包含自增列时，必须在最前面；这个特性在discuz论坛中，被设计用于“抢楼”功能，因此，若有类似的业务，则无法将该表从MyISAM转成InnoDB，需要自行变通实现(我们则是将其改到Redis中实现)；
2、不带条件频繁统计全表总记录数时(SELECT COUNT(*) FROM TAB)，InnoDB相对较慢，而MyISAM则飞快；不过，如果是基于索引条件的统计，则二者相差不大；
3、InnoDB在5.6以前不支持全文索引，不过这个相信无所谓，没什么人会在MySQL里直接跑全文索引，尤其是对中文的全文索引(前阵子有开发同学提需求直接被我否了)，确实有需要的话，可以采用Sphinx、Lucene等其他方案实现；
4、一次性导入大量数据并且后续还要进行加工处理的，可以先导入到MyISAM引擎表中，经过一通加工处理完后，再导入InnoDB表(我曾经在业务中用此方法提高数据批量导入及处理效率)；
5、InnoDB不支持LOAD TABLE FROM MASTER语法(不过应该也很少人使用吧)；
```

从MyISAM转成InnoDB可以享受的好处则有：

```
1、完整事务特性支持，以及更高的数据并发存取效率，即更高的TPS；
2、数据库实例异常重启后，InnoDB表能自动修复，而且速度相对更快，而MyISAM需要被触发才能修复，且相对耗时可能多4~5倍甚至更多；
3、更高的数据读取性能，因为InnoDB把数据及索引同时缓存在内存中，而MyISAM只缓存了索引；
4、InnoDB支持外键(不过在MySQL中，应该很少人用到外键)；
```

两个引擎间的重要区别详情见下：

**MyISAM引擎的特点：**

```
1、堆组织表；
2、不支持事务；
3、数据文件和索引文件分开存储；
4、支持全文索引；
5、主键索引和二级索引完全一样都是B+树的数据结构，只有是否唯一的区别(主键和唯一索引有唯一属性，其他普通索引没有唯一属性。B+树叶子节点存储的都是指向行记录的row pointer)；
6、有特殊计数器记录当前记录数；
7、不支持Crash recovery;
8、索引文件很容易损坏；
```

**InnoDB引擎的特点**

```
1、索引组织表；
2、支持事务；
3、数据文件和索引文件存储在同一个表空间中；
4、在5.6以前，不支持全文索引；
5、主键和二级索引数据结构一样都是B+树，但叶子节点存储的键值不一样(主键的叶子节点存储整行数据，因此也称为聚集索引；而二级索引的叶子节点存储的是主键的键值)
5、支持Crash recovery；
6、相同数据量时，InnoDB表空间文件大小约为MyISAM引擎的1.5~2倍；
```

关于InnoDB、MyISAM两种引擎的对比测试，可以参考Percona的这个对比： [http://www.percona.com/blog/2007/01/08/innodb-vs-myisam-vs-falcon-benchmarks-part-1/](http://www.percona.com/blog/2007/01/08/innodb-vs-myisam-vs-falcon-benchmarks-part-1/)

转： [http://ourmysql.com/archives/1387](http://ourmysql.com/archives/1387)