---
title: MySQL的InnoDB引擎强烈建议使用自增主键的原因
author: admin
type: post
date: 2016-11-24T01:17:18+00:00
url: /archives/17233
categories:
 - MySQL

---
1)InnoDB使用聚集索引，数据记录本身被存于主索引的叶子节点上，这就要求同一个叶子节点内的各条数据记录按主键顺序存放，因此每当一条新的记录插入时，MySQL会根据其主键将其插入适当的节点和位置，如果页面达到装载因子，则开辟一个新的页（节点）。

如果表使用自增主键，那么每次插入新的记录时，记录就会顺序添加到当前索引节点后续位置，当一页写满，就会自动开辟一个新的页。这样就就会形成一个紧凑的索引结构，近似顺序填满，由于每次插入时也不需要移动所有数据，因此效率很高，也不会增加很多额外的开销维护索引。

如果使用非自增主键，由于每次插入主键的值近乎于随机，因此每次新纪录都要被插到现有索引页的中间某个位置，此时MySQL不得不为了将新纪录插到合适位置而移动数据，甚至目标页面可能已经被写到磁盘而从缓存中清除，这增加了很多额外开销，同时频繁的移动，分页造成了大量的碎片，得到不够紧凑的索引结构，后续不得不通过OPTIMIZE TABLE来重建并优化填充页面。

2)由于MySQL从磁盘读取数据时一块一块来读取的，同时，根据局部性原理，MySQL引擎会选择预读一部分和你当前读数据所在内存相邻的数据块，这个时候这些相邻数据块的数据已经存在于内存中。由于数据库大部分是查询操作，这个时候，如果主键是自增的话，数据存储都是紧凑地存储在一起的，那么对于局部性原理利用和避免过多地I/O操作都有着巨大的促进作用