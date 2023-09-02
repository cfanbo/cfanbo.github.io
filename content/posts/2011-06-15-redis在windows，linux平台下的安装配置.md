---
title: Redis在Windows，linux平台下的安装配置
author: admin
type: post
date: 2011-06-15T07:35:49+00:00
url: /archives/9831
IM_data:
 - 'a:1:{s:68:"http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif";s:78:"http://blog.haohtml.com/wp-content/uploads/2011/06/8117_ExpandedBlockStart.gif";}'
IM_contentdowned:
 - 1
categories:
 - 数据库
tags:
 - Linux
 - redis

---
为了方便查阅资料，特将网上搜索到的Redis相关安装配置进行归档整理：

**window平台Redis安装**

下载地址: [http://code.google.com/p/servicestack/wiki/RedisWindowsDownload](http://code.google.com/p/servicestack/wiki/RedisWindowsDownload )

> Redis文件夹有以下几个文件

> redis-server.exe：服务程序
>
>
> redis-check-dump.exe：本地数据库检查
>
>
> redis-check-aof.exe：更新日志检查
>
>
> redis-benchmark.exe：性能测试，用以模拟同时由N个客户端发送M个 SETs/GETs 查询 (类似于 Apache 的ab 工具).

指定redis的配置文件，如没有指定，则使用默认设置

解压目录:

> d:\>redis-server.exe

redis-cli.exe：命令行客户端，测试用.windows下没有redis.conf配置文件.

解压目录:


> d:\>redis-cli.exe -h 127.0.0.1 -p 6379

使用方法有两种:一种是直接使用redis-cli.exe 后面加操作,另一种是直接输入redis-cli.exe进入管理界面,然后直接在redis提示符下输入命令即可.

设置一个Key并获取返回的值:

> $ ./redis-cli set mykey somevalue
>
>
> OK
>
>
> $ ./redis-cli get mykey
>
>
> Somevalue

如何添加值到list:

> $ ./redis-cli lpush mylist firstvalue
>
>
> OK
>
>
> $ ./redis-cli lpush mylist secondvalue
>
>
> OK
>
>
> $ ./redis-cli lpush mylist thirdvalue
>
>
> OK
>
>
> $ ./redis-cli lrange mylist 0 -1
>
>
> 1. thirdvalue
>
>
> 2. secondvalue
>
>
> 3. firstvalue
>
>
> $ ./redis-cli rpop mylist
>
>
> firstvalue
>
>
> $ ./redis-cli lrange mylist 0 -1
>
>
> 1. thirdvalue
>
>
> 2. secondvalue

**redis-benchmark.exe：**性能测试，用以模拟同时由N个客户端发送M个 SETs/GETs 查询 (类似于 Apache 的 ab 工具).

> ./redis-benchmark -n 100000 –c 50
>
>
> ====== SET ======
>
>
> 100007 requests completed in 0.88 seconds （译者注：100004 查询完成于 1.14 秒 ）
>
>
> 50 parallel clients （译者注：50个并发客户端）
>
>
> 3 bytes payload （译者注：3字节有效载荷)
>
>
> keep alive: 1 （译者注：保持1个连接)
>
>
> 58.50% <= 0 milliseconds（译者注：毫秒）
>
>
> 99.17% <= 1 milliseconds
>
>
> 99.58% <= 2 milliseconds
>
>
> 99.85% <= 3 milliseconds
>
>
> 99.90% <= 6 milliseconds
>
>
> 100.00% <= 9 milliseconds
>
>
> 114293.71 requests per second（译者注：每秒 114293.71 次查询）
>
>
> ……

Windows下测试并发客户端极限为60



**linux平台Redis安装：**

> wget [http://code.google.com/p/redis/downloads/detail?name=redis-2.0.4.tar.gz](http://code.google.com/p/redis/downloads/detail?name=redis-2.0.4.tar.gz)
>
> tar xvzf redis-2.0.4.tar.gz
>
> cd  redis-2.0.4
>
> make
>
> mkdir /home/redis
>
> cp redis-server  /home/redis
>
> cp redis-benchmark  /home/redis
>
> cp redis-cli  /home/redis
>
> cp redis.conf  /home/redis
>
> cd  /home/redis

启动


> ./redis-server redis.conf

进入命令交互模式，两种：


1:   ./redis-cli


2:   telnet 127.0.0.1 6379       (ip接端口)


在安装的时候要注意系统时间要正确,不然可能会提示”make[1]: warning: Clock skew detected. Your build may be incomplete”错误!


=============================================================

**配置文件参数说明**:

1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程


**daemonize no**

2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定


**pidfile /var/run/redis.pid**

3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字


**port 6379**

4. 绑定的主机地址


**bind 127.0.0.1**

5.当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能


**timeout 300**

6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose


**loglevel verbose**

7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null


**logfile stdout**

8. 设置数据库的数量，默认数据库为0，可以使用SELECT 命令在连接上指定数据库id


**databases 16**

9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合


**save**

Redis默认配置文件中提供了三个条件：


**save 900 1**

**save 300 10**

**save 60 10000**

分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。


10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大


**rdbcompression yes**

11. 指定本地数据库文件名，默认值为dump.rdb


**dbfilename dump.rdb**

12. 指定本地数据库存放目录


**dir ./**

13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步


**slaveof**

14. 当master服务设置了密码保护时，slav服务连接master的密码


**masterauth**

15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH 命令提供密码，默认关闭


**requirepass foobared**

16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息


**maxclients 128**

17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区


**maxmemory**

18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no


**appendonly no**

19. 指定更新日志文件名，默认为appendonly.aof


**appendfilename appendonly.aof**

20. 指定更新日志条件，共有3个可选值：


**no**：表示等操作系统进行数据缓存同步到磁盘（快）


**always**：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）


**everysec**：表示每秒同步一次（折衷，默认值）


**appendfsync everysec**

21. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）


**vm-enabled no**

22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享


**vm-swap-file /tmp/redis.swap**

23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0


**vm-max-memory 0**

24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值


**vm-page-size 32**

25. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。


**vm-pages 134217728**

26. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4


**vm-max-threads 4**

27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启


**glueoutputbuf yes**

28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法


**hash-max-zipmap-entries 64**

**hash-max-zipmap-value 512**

29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）


**activerehashing yes**

30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件


**include /path/to/local.conf**

=============================================================


**问题讨论:**
1. Redis官方文档对VM的使用提出了一些建议:
当你的key很小而value很大时,使用VM的效果会比较好.因为这样节约的内存比较大.
当你的key不小时,可以考虑使用一些非常方法将很大的key变成很大的value,比如你可以考虑将key,value组合成一个新的value.
最好使用linux ext3 等对稀疏文件支持比较好的文件系统保存你的swap文件.
vm-max-threads这个参数,可以设置访问swap文件的线程数,设置最好不要超过机器的核数.如果设置为0,那么所有对swap文件的操作都是串行的.可能会造成比较长时间的延迟,但是对数据完整性有很好的保证.
2. 关于Redis新的存储模式diskstore（http://timyang.net/data/redis-diskstore），
节选：
适合Web 2.0数据访问最佳的方式就是完全基于内存，比如用Memcached或者Redis snapshot方式。但是更多的业务场景是数据规模会超过RAM容量，因此有几种不同的设计模式。

**VM方式**: 将数据分页存放，由应用(如Redis)或者操作系统(如Varnish)将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中。应用实现VM缺点是代码逻辑复杂，如果业务上冷热数据边界并不分明，则换入换出代价太高，系统整体性能低。不少抢鲜的网友在微博上也反馈过使用VM种种不稳定情况。 [操作系统实现缺点在于](http://timyang.net/data/redis-misunderstanding/) 主要OS的VM换入换出是基于Page概念，比如OS VM1个Page是4K, 4K中只要还有一个元素即使只有1个字节被访问，这个页也不会被SWAP,换入也同样道理，读到一个字节可能会换入4K无用的内存。而Redis自己实现则可以达到控制换入的粒度。另外访问操作系统SWAP内存区域时block进程，也是导致Redis要自己实现VM原因之一。   **磁盘方式:** 所有的数据读写访问都是基于磁盘，由操作系统来只能的缓存访问的数据。由于现代操作系统都非常聪明，会将频繁访问的数据加入到内存中，因此应用并不需要过多特殊逻辑。MongoDB就是这种设计方式。这种方式也有一些已知的缺点，比如操作MMap写入磁盘由操作系统控制，操作系统先写哪里后写哪里应用并不知情，如果写入过程中发生了crash则数据一致性会存在问题。这个也是MongoDB饱受争议的单机 [Durability](http://blog.mongodb.org/post/381927266/what-about-durability) 问题


**硬盘存储+cache方式:** 实际原理和mysql+memcache方式类似，只不过将两者功能合二为一到一个底层服务中，简化了调用。


在上面几种方式中，除去VM，antirez觉得MongoDB方式也不太适合，因此选择了disktore方式来实现新的磁盘存储，具体细节是


1) 读操作，使用read through以及LRU方式。内存中不存在的数据从磁盘拉取并放入内存，内存中放不下的数据采用LRU淘汰。


2) 写操作，采用另外spawn一个线程单独处理，写线程通常是异步的，当然也可以把cache-flush-delay配置设成0，Redis尽量保证即时写入。但是在很多场合延迟写会有更好的性能，比如一些计数器用Redis存储，在短时间如果某个计数反复被修改，Redis只需要将最终的结果写入磁盘。这种做法作者叫per key persistence。由于写入会按key合并，因此和snapshot还是有差异，disk store并不能保证时间一致性。由于写操作是单线程，即使cache-flush-delay设成0，多个client同时写则需要排队等待，如果队列容量超过cache-max-memory,Redis设计会进入等待状态，造成调用方卡住。Google Group上有热心网友迅速完成了压力测试，当内存用完之后，set每秒处理速度从25k下降到10k再到后来几乎卡住。虽然通过增加cache-flush-delay可以提高相同key重复写入性能；通过增加cache-max-memory可以应对临时峰值写入。但是diskstore写入瓶颈最终还是在IO。

3) rdb 和新 diskstore 格式关系

rdb是传统Redis内存方式的存储格式，diskstore是另外一种格式，那两者关系如何？1.通过BGSAVE可以随时将diskstore格式另存为rdb格式，而且rdb格式还用于Redis复制以及不同存储方式之间的中间格式。2.通过工具可以将rdb格式转换成diskstore格式。

**相关链接：**

国内： [Redis几个认识误区](http://timyang.net/data/redis-misunderstanding/)

[Redis新的存储模式diskstore](http://timyang.net/data/redis-diskstore/)

[Redis使用系列：配置文件篇](http://terrylee.me/blog/post/2011/01/24/redis-internal-part1.aspx)

[Redis在Windows下的使用](http://www.madcn.net/?p=686)

[深入Redis，读redis-from-the-ground-up有感](http://www.alidw.com/?p=1611)

国外：


[http://code.google.com/p/redis/](http://code.google.com/p/redis/)

[http://blog.mjrusso.com/2010/10/17/redis-from-the-ground-up.html](http://blog.mjrusso.com/2010/10/17/redis-from-the-ground-up.html)

[http://antirez.com/post/redis-virtual-memory-story.html](http://antirez.com/post/redis-virtual-memory-story.html)

[http://code.google.com/p/redis/wiki/VirtualMemorySpecification](http://code.google.com/p/redis/wiki/VirtualMemorySpecification)