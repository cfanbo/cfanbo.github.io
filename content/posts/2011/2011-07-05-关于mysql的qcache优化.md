---
title: 关于Mysql的Qcache优化
author: admin
type: post
date: 2011-07-05T00:53:41+00:00
url: /archives/10231
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql
 - mysql优化
 - Qcache

---

```
生产环境下建议关闭此功能，因绝大部分场景下此选项会产生效率低下问题。
```

**query\_cache\_size = 64M**

指定MySQL查询缓冲区的大小。可以通过在MySQL控制台执行以下命令观察：

> \# > SHOW VARIABLES LIKE ‘%query_cache%’;
> \# > SHOW STATUS LIKE ‘Qcache%’;

\# 如果Qcache\_lowmem\_prunes的值非常大，则表明经常出现缓冲不够的情况;
如果Qcache_hits的值非常大，则表明查询缓冲使用非常频繁，如果该值较小反而会影响效率，那么可以考虑不用查询缓冲;

**Qcache\_free\_blocks**，如果该值非常大，则表明缓冲区中碎片很多。

“Qcache_free_blocks”：Query Cache 中目前还有多少剩余的blocks。如果该值显示较大，则说明Query Cache 中的内存碎片较多了，可能需要寻找合适的机会进行整理。
● “Qcache_free_memory”：Query Cache 中目前剩余的内存大小。通过这个参数我们可以较为准确的观察出当前系统中的Query Cache 内存大小是否足够，是需要增加还是过多了；


● “Qcache_hits”：多少次命中。通过这个参数我们可以查看到Query Cache 的基本效果；
● “Qcache_inserts”：多少次未命中然后插入。通过“Qcache\_hits”和“Qcache\_inserts”两个参数我们就可以算出Query Cache 的命中率了：
Query Cache 命中率= Qcache\_hits / ( Qcache\_hits + Qcache_inserts )；
● “Qcache_lowmem_prunes”：多少条Query 因为内存不足而被清除出Query Cache。通过“Qcache\_lowmem\_prunes”和“Qcache\_free\_memory”相互结合，能够更清楚的了解到我们系统中Query Cache 的内存大小是否真的足够，是否非常频繁的出现因为内存不足而有Query 被换出
● “Qcache_not_cached”：因为query\_cache\_type 的设置或者不能被cache 的Query 的数量；
● “Qcache_queries_in_cache”：当前Query Cache 中cache 的Query 数量；
● “Qcache_total_blocks”：当前Query Cache 中的block 数量；

**Query Cache 的限制**
Query Cache 由于存放的都是逻辑结构的Result Set，而不是物理的数据页，所以在性能提升的同时，也会受到一些特定的限制。
a) 5.1.17 之前的版本不能Cache 帮定变量的Query，但是从5.1.17 版本开始，Query Cache 已经开始支持帮定变量的Query 了；
b) 所有子查询中的外部查询SQL 不能被Cache；
c) 在Procedure，Function 以及Trigger 中的Query 不能被Cache；
d) 包含其他很多每次执行可能得到不一样结果的函数的Query 不能被Cache。
鉴于上面的这些限制，在使用Query Cache 的过程中，建议通过精确设置的方式来使用，仅仅让合适的表的数据可以进入Query Cache，仅仅让某些Query 的查询结果被Cache。