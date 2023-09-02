---
title: 详解MyISAM Key Cache(前篇)
author: admin
type: post
date: 2010-06-26T16:56:54+00:00
url: /archives/4102
IM_data:
 - 'a:1:{s:67:"http://www.taobaodba.com/wp-content/dbauploads/2010/02/Keycache.png";s:68:"http://blog.haohtml.com/wp-content/uploads/2011/04/25cf_Keycache.png";}'
IM_contentdowned:
 - 1
categories:
 - MySQL

---
本文将分为前、中、后三篇，分别介绍MyISAM Key Cache的一般机制、Mid-point strategy、状态、参数和命令。

“Cache为王”，[无所不在][1]。 为了最小化磁盘I/O，MyISAM将最频繁访问的索引块（“index block”）都放在内存中，这样的内存缓冲区我们称之为Key Cache，它的大小可以通过参数key\_buffer\_size来控制。在MyISAM的索引文件中（MYI），连续的单元（contiguous unit）组成一个Block，Index block的大小等于该BTree索引节点的大小。Key Cache就是以Block为单位的。

1. MyISAM如何使用Key Cache

当MySQL请求(读或写)MyISAM索引文件中某个Index Block时，首先会看Key Cache队列中是否已经缓存了对应block。如果有，就直接在Key Cache队列中进行读写了，不再需要请求磁盘。如果是写请求，那么Key Cache中的对应Block就会被标记为Dirty（和磁盘不一致）。在MyISAM在Key Cache成功请求（读写）某个Block后，会将该Block放到Key Cache队列的头部。

如果Key Cache中没有待请求（读或写）的Block，MyISAM会向磁盘请求对应的Block，并将其放到Key Cache的队列头部。队列如果满了，会将队列尾部的Block删除，该Block如果是Dirty的，会将其Flush到磁盘上。我们看到MyISAM 维护了一个LRU（Least Recently Used）的Key Cache队列。队列中的Dirty Block会在Block被踢出队列时Flush到磁盘上。

2. 图解

下图展示了访问Index Block的过程：（黑色部分为磁盘中的Index文件）

![Keycache](http://www.taobaodba.com/wp-content/dbauploads/2010/02/Keycache.png)3. 并发访问

Key Cache中的index Block是可以被并发访问的（Shared access ），下面是一些规则：

 1. 多个没有更新操作的session可以并发同一个block buffer
 2. 多个session同时访问某一个block buffer，如果某个session是update操作，则优先访问
 3. 多个session如果都需要进行block replacement，是可以并发操作。（从index file中读取block更新到key cache，但是key cache已满，需要删除一些block buffer的操作叫做block replacement）

4. 补充说明

Key cache中的Block大小可能和索引文件中的Index Block大小不同，可能是大于、小于、等于中的任何一种，但是一般都是成倍数关系的。Key Cache的block大小由参数[Key\_cache\_block_size][2]控 制。

（未完待续）

 [1]: http://www.orczhou.com/index.php/2009/08/query-cache-1/
 [2]: http://dev.mysql.com/doc/refman/5.0/en/server-system-variables.html#sysvar_key_cache_block_size