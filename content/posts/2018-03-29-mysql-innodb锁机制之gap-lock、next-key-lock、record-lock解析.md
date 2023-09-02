---
title: MySQL InnoDB锁机制之Gap Lock、Next-Key Lock、Record Lock解析
author: admin
type: post
date: 2018-03-29T06:03:55+00:00
url: /archives/17758
categories:
 - MySQL
tags:
 - innodb

---
InnoDB是一个支持行锁的存储引擎，锁的类型有：共享锁（S）、排他锁（X）、意向共享（IS）、意向排他（IX）。为了提供更好的并发，InnoDB提供了非锁定读：不需要等待访问行上的锁释放，读取行的一个快照。该方法是通过InnoDB的一个特性：MVCC来实现的。

# **InnoDB有三种行锁的算法**

1，Record Lock：单个行记录上的锁。

2，Gap Lock：间隙锁，锁定一个范围，但不包括记录本身。GAP锁的目的，是为了防止同一事务的两次当前读，出现幻读的情况。

3，Next-Key Lock：1+2，锁定一个范围，并且锁定记录本身。对于行的查询，都是采用该方法，主要目的是解决幻读的问题。

**锁的是索引，并不是记录。**
记录锁（Record Lock): 单个索引行记录上的锁
间隙锁（Gap Lock）:一般是针对非唯一索引而言的.
后码锁（Next-Key Lock）:记录锁和间隙锁的结合，对于InnoDB中，更新**非唯一索引**对应的记录，会加上Next-Key Lock。在RR下如果where未使用索引会使用全表扫描，这个时候会有Next-Key  Lock。如果更新记录为空，就不能加记录锁，只能加间隙锁。

**Next-Key Lock是行锁与间隙锁的组合，这样，当InnoDB扫描索引记录的时候，会首先对选中的索引记录加上行锁（Record Lock），再对索引记录两边的间隙加上间隙锁（Gap Lock）。如果一个间隙被事务T1加了锁，其它事务是不能在这个间隙插入记录的。**

# **参考资料**

 * （推荐）
 * [https://blog.csdn.net/zhanghongzheng3213/article/details/51721903](https://blog.csdn.net/zhanghongzheng3213/article/details/51721903)
 * [http://blog.sina.com.cn/s/blog_a1e9c7910102vnrj.html](http://blog.sina.com.cn/s/blog_a1e9c7910102vnrj.html)
 * [https://www.cnblogs.com/zhoujinyi/p/3435982.html](https://www.cnblogs.com/zhoujinyi/p/3435982.html)
 * [http://hedengcheng.com/?p=771](http://hedengcheng.com/?p=771)