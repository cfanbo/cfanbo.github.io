---
title: 三招解决MongoDB的磁盘IO问题
author: admin
type: post
date: 2012-05-03T19:12:43+00:00
url: /archives/12837
IM_contentdowned:
 - 1
categories:
 - nosql
tags:
 - MongoDB

---
有点标题党的意思，不过下面三招确实比较实用，内容来自Conversocial公司的VP Colin Howe在London MongoDB用户组的一个分享。
申请：下面几点并非放四海皆准的法则，具体是否能够使用，还需要根据自己的应用场景和数据特点来决定。

**1.使用组合式的大文档**
我们知道MongoDB是一个文档数据库，其每一条记录都是一个JSON格式的文档。比如像下面的例子，每一天会生成一条这样的统计数据：

> { metric: “content_count”, client: 5, value: 51, date: ISODate(“2012-04-01 13:00”) }
> { metric: “content_count”, client: 5, value: 49, date: ISODate(“2012-04-02 13:00”) }

而如果采用组合式大文档的话，就可以这样将一个月的数据全部存到一条记录里：

> { metric: “content_count”, client: 5, month: “2012-04”, 1: 51, 2: 49, … }

通过上面两种方式存储，预先一共存储大约7GB的数据（机器只有1.7GB的内存），测试读取一年信息，这二者的读性能差别很明显：


第一种: 1.6秒
第二种: 0.3秒
那么问题在哪里呢？
实际上原因是组合式的存储在读取数据的时候，可以读取更少的文档数量。而读取文档如果不能完全在内存中的话，其代价主要是被花在磁盘seek上，第一种存储方式在获取一年数据时，需要读取的文档数更多，所以磁盘seek的数量也越多。所以更慢。
实际上MongoDB的知名使用者foursquare就大量采用这种方式来提升读性能。见此
****

**2.采用特殊的索引结构**
我们知道，MongoDB和传统数据库一样，都是采用B树作为索引的数据结构。对于树形的索引来说，保存热数据使用到的索引在存储上越集中，索引浪费掉的内存也越小。所以我们对比下面两种索引结构：

> db.metrics.ensureIndex({ metric: 1, client: 1, date: 1})

与

> db.metrics.ensureIndex({ date: 1, metric: 1, client: 1 })

采用这两种不同的结构，在插入性能上的差别也很明显。
当采用第一种结构时，数据量在2千万以下时，能够基本保持10k/s 的插入速度，而当数据量再增大，其插入速度就会慢慢降低到2.5k/s，当数据量再增大时，其性能可能会更低。
而采用第二种结构时，插入速度能够基本稳定在10k/s。
其原因是第二种结构将date字段放在了索引的第一位，这样在构建索引时，新数据更新索引时，不是在中间去更新的，只是在索引的尾巴处进行修改。那些插入时间过早的索引在后续的插入操作中几乎不需要进行修改。而第一种情况下，由于date字段不在最前面，所以其索引更新经常是发生在树结构的中间，导致索引结构会经常进行大规模的变化。

**3.预留空间**
与第1点相同，这一点同样是考虑到传统机械硬盘的主要操作时间是花在磁盘seek操作上。
比如还是拿第1点中的例子来说，我们在插入数据的时候，预先将这一年的数据需要的空间都一次性插入。这能保证我们这一年12个月的数据是在一条记录中，是顺序存储在磁盘上的，那么在读取的时候，我们可能只需要一次对磁盘的顺序读操作就能够读到一年的数据，相比前面的12次读取来说，磁盘seek也只有一次。

> db.metrics.insert([
> { metric: ‘content_count’, client: 3, date: ‘2012-01’, 0: 0, 1: 0, 2: 0, … }
> { ……………………………., date: ‘2012-02’, … })
> { ……………………………., date: ‘2012-03’, … })
> { ……………………………., date: ‘2012-04’, … })
> { ……………………………., date: ‘2012-05’, … })
> { ……………………………., date: ‘2012-06’, … })
> { ……………………………., date: ‘2012-07’, … })
> { ……………………………., date: ‘2012-08’, … })
> { ……………………………., date: ‘2012-09’, … })
> { ……………………………., date: ‘2012-10’, … })
> { ……………………………., date: ‘2012-11’, … })
> { ……………………………., date: ‘2012-12’, … })
> ])

**结果：**
如果不采用预留空间的方式，读取一年的记录需要62ms
如果采用预留空间的方式，读取一年的记录只需要6.6ms
来源：www.colinhowe.co.uk

转载：