---
title: 详解MyISAM Key Cache(后篇)
author: admin
type: post
date: 2010-06-26T16:58:31+00:00
url: /archives/4109
IM_contentdowned:
 - 1
categories:
 - MySQL

---
在前两篇（[前篇][1]、[中篇][2]） 中，分别介绍了Key Cache的基本原理（LRU和Midpoint Insertion Strategy）。最后，将介绍一些相关的参数、状态参数和命令。

Key Cache的配置很灵活，可以针对全局配置，还可以针对某个单独数据表分配Key Cache的大小；如果一个数据表某部分的索引块被访问的非常频繁（较之其他索引块），那么可以配置Midpoint Insertion Strategy达到最大的利用率([参考][2])。

1. 如何配置Key Cache的大小

```

  #配置文件my.cnf
  key_buffer_size=50*1024*1024

```

另外，Key Cache的大小可以动态的改变2. 给数据表划分单独的Key Cache

例如：划分一块128K的Key buffer空间，指定数据表t1的Key cache放在里面。最后演示了如何删除这个特定的Key buffer空间。

```

  SET GLOBAL hot_cache.key_buffer_size=128*1024;
  CACHE INDEX t1 IN hot_cache;
  SET GLOBAL  hot_cache.key_buffer_size=0;

```

3. 预先载入某些数据表的索引

```

  LOAD INDEX INTO CACHE t1, t2

```

4. 关于Key Cache的使用情况观察 Flush现象

```

  mysql> show status like "key%";
  +------------------------+----------+
  | Variable_name          | Value    |
  +------------------------+----------+
  | Key_blocks_not_flushed | 14468    |
  | Key_blocks_unused      | 0        |
  | Key_blocks_used        | 14497    |
  | Key_read_requests      | 30586575 |
  | Key_reads              | 157      |
  | Key_write_requests     | 7100408  |
  | Key_writes             | 1199800  |
  +------------------------+----------+
  mysql> flush tables;             （注意，请不要在业务高峰期执行）
  +------------------------+----------+
  | Variable_name          | Value    |
  +------------------------+----------+
  | Key_blocks_not_flushed | 0        |   #所有修改的block都已经被flush了
  | Key_blocks_unused      | 0        |
  | Key_blocks_used        | 14497    |
  | Key_read_requests      | 38333936 |
  | Key_reads              | 207      |
  | Key_write_requests     | 8819898  |
  | Key_writes             | 1255245  |
  +------------------------+----------+

```

5. 需要注意的事项

内存中缓存的索引块（Key Cache），**有时候并不会及时刷新**到磁盘上，所以对于正在运行的数据表的索引文 件（MYI）一般都是不完整的。如果此时拷贝或者移动这些索引文件。多半会出现索引文件损坏的情况。

可以通过Flush table命令来将Key Cache中的block都flush到磁盘上。所以，一般要动态移动MyISAM表需要执行以下步骤：

首先，刷新数据表，并锁住数据表：（**注意，请不要在业务高峰期执行**）

```

  FLUSH TABLES WITH READ LOCK;

```

可以通过下面的命令来查看没有被Flush的索引块数量

```

  mysql> show status like "Key_blocks_not_flushed";
  +------------------------+----------+
  | Variable_name          | Value    |
  +------------------------+----------+
  | Key_blocks_not_flushed | 0        |
  +------------------------+----------+

```

最后，移动对应的文件（MYI MYD FRM）。

参考

 1. [MySQL Manual about Key cache][3]

 [1]: http://www.orczhou.com/index.php/2010/01/myisam-key-buffer-1/
 [2]: http://www.orczhou.com/index.php/2010/01/myisam-key-buffer-2/
 [3]: http://dev.mysql.com/doc/refman/5.0/en/myisam-key-cache.html