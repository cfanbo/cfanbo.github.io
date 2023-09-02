---
title: '[MySQL FAQ]系列 — 为什么InnoDB表要建议用自增列做主键'
author: admin
type: post
date: 2015-08-10T09:06:34+00:00
url: /archives/15919
categories:
 - MySQL
tags:
 - innodb
 - mysql优化

---
我们先了解下InnoDB引擎表的一些关键特征：

- InnoDB引擎表是基于B+树的索引组织表(IOT)；

- 每个表都需要有一个聚集索引(clustered index)；

- 所有的行记录都存储在B+树的叶子节点(leaf pages of the tree)；

- 基于聚集索引的增、删、改、查的效率相对是最高的；

- 如果我们定义了主键(PRIMARY KEY)，那么InnoDB会选择其作为聚集索引；

- 如果没有显式定义主键，则InnoDB会选择第一个不包含有NULL值的唯一索引作为主键索引；

- 如果也没有这样的唯一索引，则InnoDB会选择内置6字节长的ROWID作为隐含的聚集索引(ROWID随着行记录的写入而主键递增，这个ROWID不像ORACLE的ROWID那样可引用，是隐含的)。


综上总结，如果InnoDB表的数据写入顺序能和B+树索引的叶子节点顺序一致的话，这时候存取效率是最高的，也就是下面这几种情况的存取效率最高：

- 使用自增列(INT/BIGINT类型)做主键，这时候写入顺序是自增的，和B+数叶子节点分裂顺序一致；

- 该表不指定自增列做主键，同时也没有可以被选为主键的唯一索引(上面的条件)，这时候InnoDB会选择内置的ROWID作为主键，写入顺序和ROWID增长顺序一致；

- 除此以外，如果一个InnoDB表没有显式主键，但有可以被选择为主键的唯一索引，且该唯一索引可能不是递增关系时(例如字符串、UUID、多字段联合唯一索引的情况)，该表的存取效率就会特别差。



实际情况是如何呢？经过简单TPCC基准测试，修改为使用自增列作为主键与原始表结构分别进行TPCC测试，**前者的TpmC结果比后者高9%倍**，足见使用自增列做InnoDB表主键的明显好处，其他更多不同场景下使用自增列的性能提升可以自行对比测试下。

附图：

**1、B+树典型结构**

[![QQ截图20150810095054](http://blog.haohtml.com/wp-content/uploads/2015/08/QQ截图20150810095054.png)][1]

**2、InnoDB主键逻辑结构**

[![mysql_primarykey_index](http://blog.haohtml.com/wp-content/uploads/2015/08/mysql_primarykey_index.png)][2]

延伸阅读：

1、TPCC-MySQL使用手册, [http://imysql.com/2012/08/04/tpcc-for-mysql-manual.html](http://imysql.com/2012/08/04/tpcc-for-mysql-manual.html)

2、B+Tree index structures in InnoDB, [http://blog.jcole.us/2013/01/10/btree-index-structures-in-innodb/](http://blog.jcole.us/2013/01/10/btree-index-structures-in-innodb/)

3、B+Tree Indexes and InnoDB – Percona, [http://www.percona.com/files/presentations/percona-live/london-2011/PLUK2011-b-tree-indexes-and-innodb.pdf](http://www.percona.com/files/presentations/percona-live/london-2011/PLUK2011-b-tree-indexes-and-innodb.pdf)

4、MySQL官方手册: Clustered and Secondary Indexes,

[https://dev.mysql.com/doc/refman/5.6/en/innodb-index-types.html](https://dev.mysql.com/doc/refman/5.6/en/innodb-index-types.html) 关于MySQL的方方面面大家想了解什么，可以直接留言回复，我会从中选择一些热门话题进行分享。 同时希望大家多多 **转发**，多一些阅读量是老叶继续努力分享的绝佳助力，谢谢大家 🙂

摘自： [http://mp.weixin.qq.com/s?__biz=MjM5NzAzMTY4NQ==&mid=207339713&idx=1&sn=bceacae229cb9c8ab5947c8270b12663&scene=0#rd](http://mp.weixin.qq.com/s?__biz=MjM5NzAzMTY4NQ==&mid=207339713&idx=1&sn=bceacae229cb9c8ab5947c8270b12663&scene=0#rd)

 [1]: http://blog.haohtml.com/wp-content/uploads/2015/08/QQ截图20150810095054.png
 [2]: http://blog.haohtml.com/wp-content/uploads/2015/08/mysql_primarykey_index.png