---
title: 聚簇索引概念（Myisam与Innodb索引的区别）转推荐
author: admin
type: post
date: 2018-03-29T03:40:23+00:00
url: /archives/17743
categories:
 - MySQL
tags:
 - innodb
 - myisam

---
myisam的主索引和次索引都指向物理行，下面来进行讲解

innodb的主键下存储该行的数据，此索引指向对主键的引用

myisam的索引存储图如下，可以看出，无论是id还是cat_id，**下面都存储有存储物理地址的值。通过主键索引或者次索引来查询数据的时候，都是先查找到数据地址，然后再到物理位置上去寻找数据**。

[![](https://blog--static.oss-cn-shanghai.aliyuncs.com//uploads/2023/09/myisam-index-struct.png)][1]

innodb的索引存储图如下，我们会发现，**主键索引下面直接存储有数据，而次索引下，存储的是主键的id(不同于MyISAM，存储的是内容数据的物理地址）**。通过主键查找数据的时候，就会很快查找到数据，但是通过次索引查找数据的时候，需要先查找到对应的主键id，然后才能查找到对应的数据。

[![](https://blog--static.oss-cn-shanghai.aliyuncs.com//uploads/2023/09/innodb-index-struct.png)][2]

**总结：**
InnoDB的主索引文件上 直接存放该行数据,称为聚簇索引,次索引指向对**主键的引用**.
Myisam中, 主索引和次索引,都指向物理行(**磁盘位置**).

**注意:对InnoDB来说,**
1: 主键索引 既存储索引值,又在叶子中存储行的数据
2: 如果没有主键, 则会Unique key做主键
3: 如果没有unique,则系统生成一个内部的rowid做主键.
4: **像innodb中,主键的索引结构中,既存储了主键值,又存储了行数据,这种结构称为”聚簇索引”**

参考： [https://blog.csdn.net/lisuyibmd/article/details/53004848](https://blog.csdn.net/lisuyibmd/article/details/53004848)

转自： [https://blog.csdn.net/qq_25551295/article/details/48901317
