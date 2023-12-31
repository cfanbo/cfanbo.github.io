---
title: MySQL Cluster的常见问题
author: admin
type: post
date: 2010-06-26T14:09:08+00:00
url: /archives/3959
IM_contentdowned:
 - 1
categories:
 - MySQL
 - 前端设计
tags:
 - mysql

---

MySQL Cluster是MySQL适合于分布式计算环境的高实用、高冗余版本。它采用了NDB Cluster存储引擎，允许在1个Cluster中运行多个MySQL服务器。


MySQL Cluster是一种技术，该技术允许在无共享的系统中部署“内存中”数据库的Cluster。通过无共享体系结构，系统能够使用廉价的硬件，而且对软硬 件无特殊要求。此外，由于每个组件有自己的内存和磁盘，不存在单点故障。

总结了些移植到MySQL Cluster要注意的常见问题。

**关于连接**

MySQL集群适合用于高速带宽的环境中，采用TCP/IP方式 连接。它的性能跟主机间的连接速率有直接关系。集群中的最小速率要求是常规的100Mb以太网或者等同的网络。我们建议可能的话就采用G级网络。


**关于内存**

MySQL集群可以运行在任何启用NDB的平台上。显然，CPU 越快，内存越大，对集群性能提升越明显，64位的CPU也可能比32位的处理器更快。每个作为数据节点的机器都必须有足够的内存来保存共享数据库。


在MySQL 5.0中，集群只能基于内存。意思是所有表的数据(包括索引)都保存在内存中。如果你的数据有1GB那么大，你想要复制一份到集群中的话，那么就必须要 2GB的内存才行(每份复制占用1GB)，这是运行集群的计算机上相对其他操作系统额外要求的内存。


如果一个数据节点上的内存使用超出了可用的范围，则操作系统会使 用交换内存来达到上限值DataMemory。不过这会导致性能严重下降，并且可能导致相应时间变慢。正是由于这个原因，我们不推荐在生产环境中使用磁盘 交换空间。在任何情况下，只要达到DataMemory上限了，那么所有的操作请求(比如插入)都会失败。


在MySQL 5.1中实现了基于磁盘存储的集群，但是5.0中没有这个功能。对于包含主键哈希索引的有索引字段，必须仍保存在RAM中，但可以将所有其他字段保存在磁 盘上。


需要特别注意: 每个MySQL集群表都需要主键。如果没有定义主键，则 NDB 存储引擎会自动创建一个所有的数据节点的内存大小都要一样，由于集群中任何数据节点都不能使用比其他数据节点最小内存还多的内存。换句话说，如果集群中有 4台计算机，如果有3台计算机的内存都是3GB，而另外一台只有1GB，那么每个数据节点最多只能拿出1GB内存用于集群。


**关于安全**

MySQL集群的2个节点之间的通信是不安全的;它们没有经过任 何保护机制加密或者防护。安全的集群是放在防火墙之内的私网中，在外界中无法直接访问数据和管理节点(SQL节点也要和其他MySQL服务器一样注意安全 防护)。


**关于存储引擎**

MySQL集群只支持 NDB 存储引擎。也就是说，想要让一个表在集群节点中共享，就必须指定ENGINE=NDB(或 ENGINE=NDBCLUSTER 也一样)。


MySQL集群中也可以使用MyISAM或InnoDB存储引擎 来创建数据表，但是那些非NDB的表不会存储在集群节点间共享;它们独立于创建的MySQL服务器或者实例中。


**关于导入**

你可以把各种版本的MySQL数据导入到集群中去。唯一的要求就 是要导入的表必须是 NDB 存储引擎，也就是用 ENGINE=NDB 或 ENGINE=NDBCLUSTER方式创建的表。


**关于数据类型**

MySQL集群支持所有常用的数据类型，除了跟MySQL相关的 空间扩展类型(详情请看 Chapter 16， Spatial Extensions)。另外，NDB表的索引也有些不同。


注意: MySQL集群表(即 NDB 或 NDBCLUSTER 类型表)只支持固定长度记录。这也意味着(举例)如果有一条记录包含有 VARCHAR(255) 字段，那么它就会需要用到255个字符的空间(和数据表使用的字符集和校验所要求的空间一样大)，而不管实际存储的字符数。但是在MySQL 5.1中，只保存被记录实际占用的字段部分。


在NDB表中，数据库名称、表名称和属性名称不能与其他表处理程 序中的一样长。属性名称将被截短至31个字符，截短后如果不是唯一的，将导致错误。数据库名称和表名的总最大长度为122个字符（也就是说，NDB簇表名 的最大长度为122个字符减去该表所属的数据库的名称中的字符数）。


**关于事务**

NDB存储引擎的表都支持事务。在MySQL 5.0中，集群只支持READ COMMITTED隔离级别。


**关于外键**

NDB存储引擎不支持外键。跟MyISAM一样，它们都不支持。


**关于FULLTEXT索引**

在MySQL 5.0中，NDB存储引擎不支持FULLTEXT索引，其他除了MyISAM存储引擎外也不支持。