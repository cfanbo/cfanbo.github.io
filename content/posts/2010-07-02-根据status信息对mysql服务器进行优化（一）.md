---
title: '根据status信息对MySQL服务器进行优化[精典]'
author: admin
type: post
date: 2010-07-02T00:50:40+00:00
url: /archives/4248
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql优化
 - 查询优化

---
对于SQL查询语句对于服务器系统资源的使用情况见：发现瓶颈 – Profiling(程序剖析) -MySQL Profiling

网上有很多的文章教怎么配置MySQL服务器，但考虑到服务器硬件配置的不同，具体应用的差别，那些文章的做法只能作为初步设置参考，我们需要根据自己的 情况进行配置优化，好的做法是MySQL服务器稳定运行了一段时间后运行，根据服务器的”状态”进行优化。

mysql> show global status;

可以列出MySQL服务器运行各种状态值，另外，查询MySQL服务器配置信息语句：

mysql> show variables;

**一、慢查询**

mysql> show variables like ‘%slow%’;
+——————+——-+
| Variable_name    | Value |
+——————+——-+
| log\_slow\_queries | ON    |
| slow\_launch\_time | 2     |
+——————+——-+

mysql> show global status like ‘%slow%’;
+———————+——-+
| Variable_name       | Value |
+———————+——-+
| Slow\_launch\_threads | 0     |
| Slow_queries        | 4148 |
+———————+——-+

配置中打开了记录慢查询，执行时间超过2秒的即为慢查询，系统显示有4148个 慢查询，你可以分析慢查询日志，找出有问题的SQL语句，慢查询时间不宜设置过长，否则意义不大，最好在5秒以内，如果你需要微秒级别的慢查询，可以考虑 给MySQL打补丁： [http://www.percona.com/docs/wiki/release:start](http://www.percona.com/docs/wiki/release:start)，记得找对应的版本。

打开慢查询日志可能会对系统性能有一点点影响，如果你的MySQL是主－从结构，可以考虑打开其中一台从服务器的慢查询日志，这样既可以监控慢查 询，对系统性能影响又小。

**二、连接数**

经常会遇见”MySQL: ERROR 1040: Too many connections”的情况，一种是访问量确实很高，MySQL服务器抗不住，这个时候就要考虑增加从服务器分散读压力，另外一种情况是MySQL配 置文件中max_connections值过小：

mysql> show variables like ‘max_connections’;
+—————–+——-+
| Variable_name   | Value |
+—————–+——-+
| max_connections | 256   |
+—————–+——-+

这台MySQL服务器最大连接数是256，然后查询一下服务器响应的最大连接数：

mysql> show global status like ‘Max\_used\_connections’;
+———————-+——-+
| Variable_name        | Value |
+———————-+——-+
| Max\_used\_connections | 245   |
+———————-+——-+

MySQL服务器过去的最大连接数是245，没有达到服务器连接数上限 256，应该没有出现1040错误，比较理想的设置是：

Max_used_connections / max_connections  * 100% ≈ 85%

最大连接数占上限连接数的85％左右，如果发现比例在10%以下，MySQL服务器连接数上限设置的过高了。

**三、Key\_buffer\_size**

key\_buffer\_size是对MyISAM表性能影响最大的一个参数，下面一台以MyISAM为主要存储引擎服务器的配置：

mysql> show variables like ‘key\_buffer\_size’;
+—————–+————+
| Variable_name   | Value      |
+—————–+————+
| key\_buffer\_size | 536870912 |
+—————–+————+

分配了512MB内存给key\_buffer\_size，我们再看一下 key\_buffer\_size的使用情况：

mysql> show global status like ‘key_read%’;
+————————+————-+
| Variable_name          | Value       |
+————————+————-+
| Key\_read\_requests      | 27813678764 |
| Key_reads              | 6798830     |
+————————+————-+

一共有27813678764个索引读取请求，有 6798830个请求在内存中没有找到直接从硬盘读取索引，计算索引未命中缓存的概率：

key_cache_miss_rate ＝ Key_reads / Key_read_requests * 100%

比如上面的数据，key\_cache\_miss\_rate为0.0244%，4000个索引读取请求才有一个直接读硬盘，已经很BT 了，key\_cache\_miss\_rate在0.1%以下都很好（每1000个请求有一个直接读硬盘），如果key\_cache\_miss\_rate在 0.01%以下的话，key\_buffer_size分配的过多，可以适当减少。

MySQL服务器还提供了key\_blocks\_*参数：

mysql> show global status like ‘key\_blocks\_u%’;
+————————+————-+
| Variable_name          | Value       |
+————————+————-+
| Key\_blocks\_unused      | 0           |
| Key\_blocks\_used        | 413543      |
+————————+————-+

Key\_blocks\_unused　表示未使用的缓存簇 (blocks)数
Key\_blocks\_used  表示曾经用到的最大的blocks数

比如这台服务器，所有的缓存都用到了，要么增加 key\_buffer\_size，要么就是过渡索引了，把缓存占满了。比较理想的设置：

Key_blocks_used / (Key_blocks_unused + Key_blocks_used) * 100% ≈ 80%

**四、临时表**

mysql> show global status like ‘created_tmp%’;
+————————-+———+
| Variable_name           | Value   |
+————————-+———+
| Created\_tmp\_disk_tables | 21197   |
| Created\_tmp\_files       | 58      |
| Created\_tmp\_tables      | 1771587 |
+————————-+———+

每次创建临时表，Created\_tmp\_tables增加，如果是在磁盘上创建临时表，Created\_tmp\_disk\_tables也增加,Created\_tmp_files表示MySQL服务创建的临时文件文 件数，比较理想的配置是：

Created_tmp_disk_tables / Created_tmp_tables * 100% <= 25%
比如上面的服务器Created\_tmp\_disk\_tables / Created\_tmp_tables * 100% ＝ 1.20%，应该相当好了。我们再看一下MySQL服务器对临时表的配置：

mysql> show variables where Variable\_name in (‘tmp\_table\_size’, ‘max\_heap\_table\_size’);
+———————+———–+
| Variable_name       | Value     |
+———————+———–+
| max\_heap\_table_size | 268435456 |
| tmp\_table\_size      | 536870912 |
+———————+———–+

tmp\_table\_size的值为17M大小(my.conf/my.ini),只有256MB以下的临时表才能全部放内存，超过的就会用到硬盘临时 表。

**五、Open Table情况**

mysql> show global status like ‘open%tables%’;
+—————+——-+
| Variable_name | Value |
+—————+——-+
| Open_tables   | 919   |
| Opened_tables | 1951  |
+—————+——-+

Open\_tables表示打开表的数量，Opened\_tables表示打开过的表数量， 如果Opened\_tables数量过大，说明配置中table\_cache(**5.1.3之后这个值变量名叫做table\_open\_cache**)值可能太小， 我们查询一下服务器table_cache值：

mysql> show variables like ‘table_cache’;
+—————+——-+
| Variable_name | Value |
+—————+——-+
| table_cache   | 2048  |
+—————+——-+

比较合适的值为：

Open_tables / Opened_tables  * 100% >= 85%

 Open_tables / table_cache * 100% <= 95%

待续，本文参考以下网页：

1. [http://dev.mysql.com/doc/refman/5.1/en/server-status-variables.htm](http://dev.mysql.com/doc/refman/5.1/en/server-status-variables.htm)

2. [http://dev.mysql.com/doc/refman/5.1/en/server-system-variables.html](http://dev.mysql.com/doc/refman/5.1/en/server-system-variables.html)

3. [http://www.ibm.com/developerworks/cn/linux/l-tune-lamp-3.html](http://www.ibm.com/developerworks/cn/linux/l-tune-lamp-3.html)

4. [http://www.day32.com/MySQL/tuning-primer.sh](http://www.day32.com/MySQL/tuning-primer.sh) 具体数值主要参考此工具

**六、进程使用情况**

mysql> show global status like ‘Thread%’;
+——————-+——-+
| Variable_name     | Value |
+——————-+——-+
| Threads_cached    | 46    |
| Threads_connected | 2     |
| Threads_created   | 570   |
| Threads_running   | 1     |
+——————-+——-+

如果我们在MySQL服务器配置文件中设置了thread\_cache\_size， 当客户端断开之后，服务器处理此客户的线程将会缓存起来以响应下一个客户而不是销毁（前提是缓存数未达上限）。Threads\_created表示创建过 的线程数，如果发现Threads\_created值过大的话，表明MySQL服务器一直在创建线程，这也是比较耗资源，可以适当增加配置文件中 thread\_cache\_size值，查询服务器thread\_cache\_size配置：

mysql> show variables like ‘thread\_cache\_size’;
+——————-+——-+
| Variable_name     | Value |
+——————-+——-+
| thread\_cache\_size | 64    |
+——————-+——-+

示例中的服务器还是挺健康的。

**七、查询缓存(query cache)**

mysql> show global status like ‘qcache%’;
+————————-+———–+
| Variable_name           | Value     |
+————————-+———–+
| Qcache\_free\_blocks      | 22756     |
| Qcache\_free\_memory      | 76764704  |
| Qcache_hits             | 213028692 |
| Qcache_inserts          | 208894227 |
| Qcache\_lowmem\_prunes    | 4010916   |
| Qcache\_not\_cached       | 13385031  |
| Qcache\_queries\_in_cache | 43560     |
| Qcache\_total\_blocks     | 111212    |
+————————-+———–+

MySQL查询缓存变量解释：

Qcache\_free\_blocks：缓存中相邻内存块的个数。数目大说明可能有碎片。FLUSH QUERY CACHE会对缓存中的碎片进行整理，从而得到一个空闲块。
Qcache\_free\_memory：缓存中的空闲内存。
Qcache_hits：每次查询在缓存中命中时就增大
Qcache_inserts：每次插入一个查询时就增大。命中次数除以插入次数就是不中比率。
Qcache\_lowmem\_prunes：缓存出现内存不足并且必须要进行清理以便为更多查询提供空间的次数。这个数字最好长时间来看；如果这 个数字在不断增长，就表示可能碎片非常严重，或者内存很少。（上面的 free\_blocks和free\_memory可以告诉您属于哪种情况）
Qcache\_not\_cached：不适合进行缓存的查询的数量，通常是由于这些查询不是 SELECT 语句或者用了now()之类的函数。
Qcache\_queries\_in_cache：当前缓存的查询（和响应）的数量。
Qcache\_total\_blocks：缓存中块的数量。

我们再查询一下服务器关于query_cache的配置：

mysql> show variables like ‘query_cache%’;
+——————————+———–+
| Variable_name                | Value     |
+——————————+———–+
| query\_cache\_limit            | 2097152   |
| query\_cache\_min\_res\_unit     | 4096      |
| query\_cache\_size             | 203423744 |
| query\_cache\_type             | ON        |
| query\_cache\_wlock_invalidate | OFF       |
+——————————+———–+

各字段的解释：

query\_cache\_limit：超过此大小的查询将不缓存
query\_cache\_min\_res\_unit：缓存块的最小大小
query\_cache\_size：查询缓存大小
query\_cache\_type：缓存类型，决定缓存什么样的查询，示例中表示不缓存 select sql\_no\_cache 查询
query\_cache\_wlock_invalidate：当有其他客户端正在对MyISAM表进行写操作时，如果查询在query cache中，是否返回cache结果还是等写操作完成再读表获取结果。

query\_cache\_min\_res\_unit的配置是一柄”双刃剑”，默认是4KB，设置值大对大数据查询有好处，但如果你的查询都是小数 据查询，就容易造成内存碎片和浪费。

查询缓存碎片率 = Qcache\_free\_blocks / Qcache\_total\_blocks * 100%

如果查询缓存碎片率超过20%，可以用FLUSH QUERY CACHE整理缓存碎片，或者试试减小query\_cache\_min\_res\_unit，如果你的查询都是小数据量的话。

查询缓存利用率 = (query_cache_size – Qcache_free_memory) / query_cache_size * 100%

查询缓存利用率在25%以下的话说明query\_cache\_size设置的过大，可适当减小；查询缓存利用率在80％以上而且 Qcache\_lowmem\_prunes > 50的话说明query\_cache\_size可能有点小，要不就是碎片太多。

查询缓存命中率 = (Qcache_hits – Qcache_inserts) / Qcache_hits * 100%

示例服务器 查询缓存碎片率 ＝ 20.46％，查询缓存利用率 ＝ 62.26％，查询缓存命中率 ＝ 1.94％，命中率很差，可能写操作比较频繁吧，而且可能有些碎片。

**八、排序使用情况**

mysql> show global status like ‘sort%’;
+——————-+————+
| Variable_name     | Value      |
+——————-+————+
| Sort\_merge\_passes | 29         |
| Sort_range        | 37432840   |
| Sort_rows         | 9178691532 |
| Sort_scan         | 1860569    |
+——————-+————+

Sort\_merge\_passes 包括两步。MySQL 首先会尝试在内存中做排序，使用的内存大小由系统变量 Sort\_buffer\_size 决定，如果它的大小不够把所有的记录都读到内存中，MySQL 就会把每次在内存中排序的结果存到临时文件中，等 MySQL 找到所有记录之后，再把临时文件中的记录做一次排序。这再次排序就会增加 Sort\_merge\_passes。实际上，MySQL 会用另一个临时文件来存再次排序的结果，所以通常会看到 Sort\_merge\_passes 增加的数值是建临时文件数的两倍。因为用到了临时文件，所以速度可能会比较慢，增加 Sort\_buffer\_size 会减少 Sort\_merge\_passes 和 创建临时文件的次数。但盲目的增加 Sort\_buffer\_size 并不一定能提高速度，见 How fast can you sort data with MySQL?（引自 [http://qroom.blogspot.com/2007/09/mysql-select-sort.html](http://qroom.blogspot.com/2007/09/mysql-select-sort.html)，貌似被墙）

另外，增加read\_rnd\_buffer\_size(3.2.3是record\_rnd\_buffer\_size)的值对排序的操作也有一点的 好处，参见： [http://www.mysqlperformanceblog.com/2007/07/24/what-exactly-is- read_rnd_buffer_size/](http://www.mysqlperformanceblog.com/2007/07/24/what-exactly-is-read_rnd_buffer_size/)

**九、文件打开数(open_files)**

mysql> show global status like ‘open_files’;
+—————+——-+
| Variable_name | Value |
+—————+——-+
| Open_files    | 1410  |
+—————+——-+

mysql> show variables like ‘open\_files\_limit’;
+——————+——-+
| Variable_name    | Value |
+——————+——-+
| open\_files\_limit | 4590  |
+——————+——-+

比较合适的设置：Open_files / open_files_limit * 100% <= 75％

**十、表锁情况**

mysql> show global status like ‘table_locks%’;
+———————–+———–+
| Variable_name         | Value     |
+———————–+———–+
| Table\_locks\_immediate | 490206328 |
| Table\_locks\_waited    | 2084912   |
+———————–+———–+

Table\_locks\_immediate表示立即释放表锁 数，Table\_locks\_waited表示需要等待的表锁数，如果Table\_locks\_immediate / Table\_locks\_waited > 5000，最好采用InnoDB引擎，因为InnoDB是行锁而MyISAM是表锁，对于高并发写入的应用InnoDB效果会好些。示例中的服务器 Table\_locks\_immediate / Table\_locks\_waited ＝ 235，MyISAM就足够了。

**十一、表扫描情况**

mysql> show global status like ‘handler_read%’;
+———————–+————-+
| Variable_name         | Value       |
+———————–+————-+
| Handler\_read\_first    | 5803750     |
| Handler\_read\_key      | 6049319850  |
| Handler\_read\_next     | 94440908210 |
| Handler\_read\_prev     | 34822001724 |
| Handler\_read\_rnd      | 405482605   |
| Handler\_read\_rnd_next | 18912877839 |
+———————–+————-+

各字段解释参见 [http://blog.haohtml.com/index.php/archives/4262](http://blog.haohtml.com/index.php/archives/4262)，调出服务器完成的查询请求次数：

mysql> show global status like ‘com_select’;
+—————+———–+
| Variable_name | Value     |
+—————+———–+
| Com_select    | 222693559 |
+—————+———–+

计算表扫描率：

表扫描率 ＝ Handler_read_rnd_next / Com_select

如果表扫描率超过4000，说明进行了太多表扫描，很有可能索引没有建好，增加read\_buffer\_size值会有一些好处，但最好不要超过 8MB。

**后记：**

文中提到一些数字都是参考值，了解基本原理就可以，除了MySQL提供的各种status值外，操作系统的一些性能指标也很重要，比如常用的 top,iostat等，尤其是iostat，现在的系统瓶颈一般都在磁盘IO上，关于iostat的使用，可以参考： [http://www.php-oa.com/2009/02/03/iostat.html](http://www.php-oa.com/2009/02/03/iostat.html)

修改mysql全局变量的一些变量见: [http://blog.haohtml.com/index.php/archives/4284](http://blog.haohtml.com/index.php/archives/4284)