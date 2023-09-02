---
title: MySql Query Cache 查询缓存介绍(1)
author: admin
type: post
date: 2009-06-18T16:25:35+00:00
excerpt: 'MySql Query Cache 和 Oracle Query Cache 是不同的， Oracle Query Cache 是缓存执行计划的，而MySql Query Cache 不缓存执行计划而是整个结果集。缓存整个结果集的好处不言而喻，但由于缓存的是结果集因此Query必须是完全一样的，这样带来的后果就是平均 Hit Rate 命中率一般不会太高。 Query Cache 对于一些小型应用程序或者数据表的数据量不大的情况下效果是最为明显的。'
url: /archives/1861
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---
MySql Query Cache 和 Oracle  Query Cache 是不同的， Oracle Query Cache 是缓存执行计划的，而MySql Query Cache 不缓存执行计划而是整个结果集。缓存整个结果集的好处不言而喻，但由于缓存的是结果集因此Query必须是完全一样的，这样带来的后果就是平均 Hit Rate 命中率一般不会太高。 Query Cache 对于一些小型应用程序或者数据表的数据量不大的情况下效果是最为明显的。

作为一个新的特性，MySql Query Cache 有什么特典和局限呢？ 咱一个一个来说：

1、Cache 机制对应用程序是透明的。在应用程序中只是改变查询语句的语义，也能得到缓存中的查询结果集。如果你没有使用 query\_cache\_wlock_invalidate=ON   来提示MySql 锁表将要进行写操作，那么此时的查询即使表在锁Lock状态下或者预备更新的状态下，仍然可以从缓存中获得结果集；

2、只缓存整个查询结果集，即对子查询，内联视图和部分UNION的查询是不缓存的；

3、缓存机制工作在Packet 级别，第二项的只缓存整个查询结果集就是因为局限于这个机制的原因。由于没有额外的转换和处理，所以保证缓存结果集返回能够非常快；

4、缓存处理在解析查询前进行，保证缓存高性能的一个原因就是查询缓存在执行查询解析前先查找是否已经存在缓存，如果已经存在查询缓存，则直接返回结果集。

5、 查询必须绝对完全同，由于在查找缓存是否存在前不进行查询解析( Query Parser )所以查询并没有经过规范化处理（Normalized），因此缓存查找的过程是按字节顺序进行的 ( Byte by byte )。更具体点说吧：在每次查询时包不同的注释、多余的空格以及大小写不同等等，都不会指向同一个缓存结果集。

6、只有 SELECT 语句被缓存。 插入、删除、更新当然不需要进行缓存了，同时 SHOW 命令和 存储过程 stored procedure （包括存储过程中的SELECT）也不会进入缓存结果集。

7、空格和注释不要出现在查询语句的最前面，当查找缓存时第一个字幕如果不是”S” ,就会停止查询缓存结果集了。第5、6项已经解释过了；

8、不支持预备查询 prepared statement 和 游标 cursors 。 （ ？ ）

9、或许不支持事务处理。（？）

10、查询结果必须完全一致，才能进入缓存结果集。比如：查询语句中有 UUID , RAND , CONNECTION_ID 等会动态改变查询结果集的函数，都不会进入缓存结果集的；

11、查询缓存失效的粒度级别的是表，当表被修改时，所有与改表相关的缓存立即失效( invalidation )。

12、过长时间的查询缓存容易造成碎片 fragmentation  ，这一点和Windows的磁盘管理的碎片整理类似，长时间查询缓存产生的碎片对执行效率有一定影响。可以把查询缓存碎片看作是是查询缓存可用内存（Qcache\_free\_memory）的块（Qcache\_free\_blocks ）。FLUSH QUERY CACHE  命令可以削除这种情况。

13、设定适当大小的查询缓存用的内存，由于前面提到的一些原因，一般情况下MySql 的查询缓存机制对内存的需求不可能无限增长，因此设定一个适当的查询缓存内存值是比较经济的做法。可以通过查看 Qcache\_free\_memory 和 Qcache\_lowmem\_prunes 的状态来进行适当设置。

14、查询缓存的运行模式，默认情况下开启缓存后MySql 的缓存机制对全局的有效，如果你只想对特定的查询语句使用缓存，可以通过把 query\_cache\_type  设定为 “DEMAND” 并且在查询语句中加入： SQL\_CACHE  来进行，比如：SELECT SQL\_CACHE [DomoloSeoHelper][1] from domolo where author=’tianchunfeng’ 。

上面为你介绍了 Mysql 查询缓存的一些基本特点，那么如何监控Mysql 查询缓存的运行时状态呢？比如监控查询缓存的命中率，调节查询缓存的内存大小等等数据。

可以使用下面的命令：

mysql> show status like ‘Qcache%’;

输出:

```
+-------------------------+----------+
```

```
| Variable_name           | Value    |
```

```
+-------------------------+----------+
```

```
| Qcache_free_blocks      | 1        |
```

```
| Qcache_free_memory      | 16766912 |
```

```
| Qcache_hits             | 3        |
```

```
| Qcache_inserts          | 1        |
```

```
| Qcache_lowmem_prunes    | 0        |
```

```
| Qcache_not_cached       | 6        |
```

```
| Qcache_queries_in_cache | 1        |
```

```
| Qcache_total_blocks     | 4        |
```

```
+-------------------------+----------+
```

 [1]: http://www.domolo.com/seo_software