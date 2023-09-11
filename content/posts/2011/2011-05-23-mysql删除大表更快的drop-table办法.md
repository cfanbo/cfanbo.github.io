---
title: mysql删除大表更快的drop table办法
author: admin
type: post
date: 2011-05-23T05:46:23+00:00
url: /archives/9571
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---
曾经发文介绍过，DROP table XXX ,特别是碰到大表时，

在DROP TABLE 过程中，所有操作都会被HANG住。
这是因为INNODB会维护一个全局独占锁（在table cache上面），直到DROP TABLE完成才释放。
在我们常用的ext3,ext4，ntfs文件系统，要删除一个大文件（几十G，甚至几百G）还是需要点时间的。
下面我们介绍一个快速DROP table 的方法； 不管多大的表,INNODB 都可以很快返回，表删除完成；
实现：巧用LINK（硬链接）

实测：

 : test 21:38:00> show table status like ‘tt’ \G
\***\***\***\***\***\***\***\***\*\\*\* 1. row \*\*\***\***\***\***\***\***\***\****
Name: tt
Engine: InnoDB
Version: 10
Row_format: Compact
Rows: 151789128
Avg\_row\_length: 72
Data_length: 11011096576
Max\_data\_length: 0
Index_length: 5206179840
Data_free: 7340032
Auto_increment: NULL
Create_time: 2011-05-18 14:55:08
Update_time: NULL
Check_time: NULL
Collation: utf8\_general\_ci
Checksum: NULL
Create_options:
Comment:
1 row in set (0.22 sec)

 : test 21:39:34> drop table tt ;
Query OK, 0 rows affected (25.01 sec)

删除一个11G的表用时25秒左右（硬件不同，时间不同）；

下面我们来对另一个更大的表进行删除；
但之前，我们需要对这个表的数据文件做一个硬连接：

root@ # ln stock.ibd stock.id.hdlk
root@ # ls stock.* -l
-rw-rw—- 1 mysql mysql        9196 Apr 14 23:03 stock.frm
-rw-r–r– 2 mysql mysql 19096666112 Apr 15 09:55 stock.ibd
-rw-r–r– 2 mysql mysql 19096666112 Apr 15 09:55 stock.id.hdlk

你会发现stock.ibd的INODES属性变成了2；

下面我们继续来删表。

 : test 21:44:37> show table status like ‘stock’ \G
\***\***\***\***\***\***\***\***\*\\*\* 1. row \*\*\***\***\***\***\***\***\***\****
Name: stock
Engine: InnoDB
Version: 10
Row_format: Compact
Rows: 49916863
Avg\_row\_length: 356
Data_length: 17799577600
Max\_data\_length: 0
Index_length: 1025507328
Data_free: 4194304
Auto_increment: NULL
Create_time: 2011-05-18 14:55:08
Update_time: NULL
Check_time: NULL
Collation: utf8\_general\_ci
Checksum: NULL
Create_options:
Comment:
1 row in set (0.23 sec)

 : test 21:39:34> drop table stock ;
Query OK, 0 rows affected (0.99 sec)

1秒不到就删除完成； 也就是DROP TABLE不用再HANG这么久了。
但table是删除了，数据文件还在，所以你还需要最后数据文件给删除。

root # ll
total 19096666112
-rw-r–r– 2 mysql mysql 19096666112 Apr 15 09:55 stock.id.hdlk
root # rm stock.id.hdlk
虽然DROP TABLE 多绕了几步。(如果你有一个比较可靠的自运行程序（自动为大表建立硬链接，并会自动删除过期的硬链接文件），就会显得不那么繁琐。)
这样做能大大减少MYSQL HANG住的时间； 相信还是值得的。

至于原理: 就是利用OS HARD LINK的原理,
当多个文件名同时指向同一个INODE时,这个INODE的引用数N>1, 删除其中任何一个文件名都会很快.
因为其直接的物理文件块没有被删除.只是删除了一个指针而已;
当INODE的引用数N=1时, 删除文件需要去把这个文件相关的所有数据块清除,所以会比较耗时;

好了. 大家试试吧.

来源: [http://www.mysqlops.com/2011/05/18/mysql删除大表更快的drop-table办法.html](http://www.mysqlops.com/2011/05/18/mysql删除大表更快的drop-table办法.html)