---
title: MySQL中对MVCC的理解总结
author: admin
type: post
date: 2018-09-18T09:55:26+00:00
url: /archives/18221
categories:
 - MySQL
tags:
 - innodb
 - mvcc
 - mysql

---
# 一、MVCC简介

MVCC (Multiversion Concurrency Control)，即多版本并发控制技术。InnoDB数据库的事务隔离级别就是通过UNDO和MVCC来实现的(ACID特性)，旧数据存储在UNDO中，再通过DB\_ROLL\_PTR 回溯查找历史版本。

# 二、MVCC原理

1、通过DB_ROLL_PT 回溯查找数据历史版本2、通过read view判断行记录是否可见

理解这一块之前，我们必须先了解一下row的内部存储格式

![image-20230904183143992](https://blog--static.oss-cn-shanghai.aliyuncs.com//uploads/2023/09/image-20230904183143992.png)

字段说明：

 * **DB\_ROW\_ID：**长度6个字节。此值由**InnoDB自动生成**，聚集索引时使用。如果用户未显式指定表主键时，表优先使用**第一个非null的唯一索引**作为主键.否则使用DB\_ROW\_ID的值作为主键ID，聚集索引会使用此值。如果指定了表主键的话，则聚集索引使用指定的值。
 * **DB\_TRX\_ID：**6个字节的事务ID。标记了最后**更新**此记录的事务ID，每开起一个新事务，其值自动+1
 * **DB\_ROLL\_PTR：**7字节的回滚指针。指向当前记录项的**undo log**记录，找之前版本的数据需通过此指针。

**MySQL中的MVCC原理**

[![](https://blog--static.oss-cn-shanghai.aliyuncs.com//uploads/2023/09/mysql_mvcc_3.24.44.png)][2]

**首次 `insert`** 记录的`DB_ROLL_PTR`指针为**NULL**。修改新值后，记录的 `DB_ROLL_PTR` 回滚指针指向原始值在`Undo Log` 日志的位置，也就是说将原值在`Unde Log`的物理位置存储到原记录的 `DB_POLL_PTR` 字段。如果事务回滚的话，则从`Undo Log` 中把原始值读取出来再放到记录中去。如果直接commit的话，则直接保存即可。记录格式参考：

[![](https://blog--static.oss-cn-shanghai.aliyuncs.com//uploads/2023/09/mysql_trans_2.jpg)][3]

[![](https://blog--static.oss-cn-shanghai.aliyuncs.com//uploads/2023/09/mysql_redo_log.png)][4]

**InnoDB Undo Log的日志类型**
MySQL数据库InnoDB存储引擎的undo log采用了**逻辑的日志**。
InnoDB undo log的格式可以概括为:<操作类型>++<数据>.  A. 从表中删除一行记录
**TRX\_UNDO\_DEL\_MARK\_REC** (将主键记入日志)
在删除一条记录时，并不是真正的将数据从数据库中删除,只是标记为已删除.这样做的好处是Undo Log中不用记录整行的信息.在undo时操作也变得很简单.
  B. 向表中插入一行记录
**TRX\_UNDO\_INSERT_REC** (仅将主键记入日志)
**TRX\_UNDO\_UPD\_DEL\_REC** (将主键记入日志) 当表中有一条被标记为删除的记录和要插入的数据主键相同时， 实际的操作是更新这个被标记为删除的记录。  C. 更新表中的一条记录
**TRX\_UNDO\_UPD\_EXIST\_REC** (将主键和被更新了的字段内容记入日志)
**TRX\_UNDO\_DEL\_MARK\_REC** 和 **TRX\_UNDO\_INSERT_REC ** 当更新主键字段时，实际执行的过程是删除旧的记录然后，再插入一条新的记录。

**事务隔离级别的区别：**

- RR隔离级别下，在**每个事务开始**的时候，会将当前系统中的所有的活跃事务拷贝到一个列表中(read view)。

- RC隔离级别下，在事务中的 **每个语句开始（select)** 时，会将当前系统中的所有的活跃事务拷贝到一个列表中(read view)

- 然后按照以下逻辑判断事务的可见性

[![](https://blog--static.oss-cn-shanghai.aliyuncs.com//uploads/2023/09/mysql_readview.jpg)][5]

# MVCC解决了什么问题

- MVCC使得数据库读不会对数据加锁，普通的SELECT请求不会加锁，提高了数据库的并发处理能力；

- 借助MVCC，数据库可以实现RC，RR等隔离级别，用户可以查看当前数据的前一个或者前几个历史版本。保证了ACID中的I特性（隔离性）。


**查看当前数据库中的活跃事务**

```
SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX
```

# 参考

 * [MVCC原理探究及MySQL源码实现分析](https://mp.weixin.qq.com/s/tNA_-_MoYt1fJT0icyKbMg)
 * [MySQL数据库InnoDB存储引擎Log漫游(2)](https://mp.weixin.qq.com/s?timestamp=1537701178&src=3&ver=1&signature=WZFE75k0*co7M7wtc2aAbSqkeWqyo5KyqfCkU9KAG6b8KJrGt-vZZaXeLdlkTLKbb1wF1psN5sSlv-qCs0BYlYU9x-AMiLdK2KB5nm7tm1MFCuLFIz94Pjr8VmxQbkoQb6s3*hk-A-XyRG37cEnDnBngxPiZqu2PNeY0uqGtBUQ=)
 * [MySQL多版本并发控制分析](https://www.2cto.com/database/201503/381708.html)

[1]: https://blog.haohtml.com/wp-content/uploads/2018/09/mysql_trans_1.jpg
[2]: https://blog.haohtml.com/wp-content/uploads/2018/09/mysql_mvcc_3.24.44.png
[3]: https://blog.haohtml.com/wp-content/uploads/2018/09/mysql_trans_2.jpg
[4]: https://blog.haohtml.com/wp-content/uploads/2018/09/mysql_redo_log.png
[5]: https://blog.haohtml.com/wp-content/uploads/2018/09/mysql_readview.jpg