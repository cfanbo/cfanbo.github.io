---
title: MySQL中的 InnoDB Buffer Pool
author: admin
type: post
date: 2020-01-04T03:05:54+00:00
url: /archives/19383
categories:
 - MySQL
tags:
 - mysql

---
## 一、InnoDB Buffer Pool简介 

Buffer Pool是InnoDB引擎内存中的一块区域，主要用来缓存表和索引数据使用。我们知道从内存读取数据要比磁盘读取效率要高的多，这也正是buffer pool发挥的主要作用。一般配置值都比较大，在专用数据库服务器上，大小为物理内存的80%左右。

## 二、Buffer Pool LRU 算法 

Buffer Pool 链表使用**优化改良后LRU**(最近最少使用)算法进行管理。

整个LRU链表可分为两个子链表，一个是New Sublist，也称为Young列表或新生代，另一个是Old Sublist ，称为Old 列表或老生代。每个子链表都有一个Head和Tail，中间部分是存储Page数据的地方。

当新的Page放入 Buffer Pool 缓存池的时候，会交其Page插入就是两个子链表的交界处，称为midpoint，同时就会有旧的Page被淘汰，整个操作过程都需要对链接进行维护。

**Young 链表区存放的数据是经常访问的数据；
Old 链表区存放是即将被淘汰的数据；**

一个新数据先被插入到midpoint 位置，根据LRU算法访问频率高的Page，会慢慢向Young区Head方向移动。同时访问频率低的Page会从Young区慢慢转到Old 链表的Tail，至到被淘汰。![](https://blog.haohtml.com/wp-content/uploads/2020/01/innodb-buffer-pool-list.png)**Figure 14.2 Buffer Pool List**

算法操作流程如下：

Old区占 Buffer Pool大小的 3/8，此值可以通过 **innodb\_old\_blocks_pct** 调整，单位为百分比。

midpoint是young和Old的相交边界，新页插入的位置。

当执行一个read操作时，如select查询语句，这时InnoDB会将页读入到 buffer pool中，最初时会放在midpoint位置(放在Old区的head部分)。（**这一点和传统的LRU 算法不一样**，并没有放在Old的Tail位置。

如果访问的是Old区的页，会使此页被标记为”young”, 并将其向到 Young区的头部。如果由于用户操作引起的将page放在buffer pool中，当插入到midpoint位置后，立即进行访问操作，会将此page标识为”young”。而如果一个page是由于**预读**操作而访问的话，则不会产生立即访问这个行为，并且有可能在此page被淘汰之前一次也没有访问过。

随着数据库地运行，新page不断的插入midpoint位置，并不断地被访问到，其page慢慢向Young区的Tail位置移动，同时未被访问的page也会慢慢向Old区的Tail方向移动，最终将其从Old区的尾部被淘汰。

**什么是预读？**

磁盘读写，并不是按需读取，而是按页读取，一次至少读一页数据（一般是4K），如果未来要读取的数据就在页中，就能够省去后续的磁盘IO，提高效率。在InnoDB中一个page一般为14K，是操作系统的4倍，当然这个值是可以通过参数innodb\_page\_size调整的。

**预读有哪些好处？**

数据访问，通常都遵循“集中读写”的原则，使用一些数据，大概率会使用附近的数据，这就是所谓的“局部性原理”，它表明提前加载是有效的，确实能够**减少磁盘IO**。

## 三、Page 移动策略 

上面我们提到过这里使用的LRU算法和传统的算法有些区别，这里我们主要介绍一下不同点。

如果使用传统算法的话，发现当我们执行一个访问频率很小的 query 时，如一个全表扫描，会把大量的数据插入到midpoint位置，短时间内多次访问的话，会导致这些数据被移动到young区（尽管这些数据以后不再访问了），并多次进行链表修改，不但大量占用了young区的空间，同时还造成了频繁修改young链表的成本，这时就需要一种方法来解决此问题。

```
参数：innodb_old_blocks_time
单位：毫秒
默认值：1000
```

此参数主要用来控制Old区的数据移动策略，如果Page在Old区停留innodb\_old\_blocks_time时间后，再次被访问才会向Young区移动，否则保存位置不变。这样就避免了频繁向Young区移动数据，减少维护LRU链表的成本。例如原来1秒内访问了10次，会修改Young链表10次，现在只修改一次就可以了，大大节省了LRU链表维护成本。

可以看到将此值调整大的话，将直接影响old数据向young移动的难度，从而加速old区淘汰的频繁，保护了Young区内被频繁访问的数据不被淘汰。

## 四、监控 

在SHOW ENGINE INNODB STATUS 里面提供了Buffer Pool一些监控指标，有几个我们需要关注一下：

 1. youngs/s：该指标表示的是每秒访问Old 链表中页面，使其移动到Young链表的次数。如果MySQL实例都是一些小事务，没有大表全扫描，且该指标很小，就需要调大innodb\_old\_blocks\_pct 或者减小innodb\_old\_blocks\_time，这样会使得Old List 的长度更长，Old页面被移动到Old List 的尾部消耗的时间会更久，那么就提升了下一次访问到Old List里面的页面的可能性。如果该指标很大，可以调小innodb\_old\_blocks\_pct，同时调大innodb\_old\_blocks\_time，保护热数据。
 2. non-youngs/s：该指标表示的是每秒访问Old 链表中页面，没有移动到Young链表的次数，因为其不符合innodb\_old\_blocks\_time。如果该指标很大，一般情况下是MySQL存在大量的全表扫描。如果MySQL存在大量全表扫描，且这个指标又不大的时候，需要调大innodb\_old\_blocks\_time，因为这个指标不大意味着全表扫描的页面被移动到Young 链表了，调大innodb\_old\_blocks_time时间会使得这些短时间频繁访问的页面保留在Old 链表里面。
 3. Buffer pool hit rate: 表示缓冲池命中率，最好是100%全部命中的情况，如果低于95%的话，用户需要观察是否由于全表扫描引起的LRU列表被污染的问题了。

## 五、总结 

缓冲池(buffer pool)是一种常见的降低磁盘读取加速数据访问的机制，数据以Page为单位，默认页大小为16K，配置参数为 innodb\_page\_size 。使用的是优化后改良后的LRU算法。

主要三个参数

**参数**：innodb\_buffer\_pool_size

**介绍**：配置缓冲池的大小，在内存允许的情况下，DBA往往会建议调大这个参数，越多数据和索引放到内存里，数据库的性能会越好。一般对于专业的数据库服务器，此值大小为物理内存大小的80%左右。

**参数**：innodb\_old\_blocks_pct

**介绍**：老生代Old区占整个LRU链长度的比例，默认是37，即整个LRU中新生代与老生代长度比例是63:37。

_画外音：如果把这个参数设为100，就退化为普通LRU了。_

**参数**：innodb\_old\_blocks_time

**介绍**：老生代停留时间窗口，单位是毫秒，默认是1000，即同时满足“被访问”与“在老生代停留时间超过1秒”两个条件，才会被插入到新生代头部。

**参考资料：**

 * [https://dev.mysql.com/doc/refman/5.7/en/innodb-buffer-pool.html](https://dev.mysql.com/doc/refman/5.7/en/innodb-buffer-pool.html)
 * [https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651962450&](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651962450&idx=1&sn=ce17c4da8d20ce275f75d0f2ef5e40c9&chksm=bd2d098e8a5a809834aaa07da0d7546555385543fb6d687a7cf94d183ab061cd301a76547411&scene=21#wechat_redirect)
 * [https://www.cnblogs.com/warcraft/p/12012462.html](https://www.cnblogs.com/warcraft/p/12012462.html)