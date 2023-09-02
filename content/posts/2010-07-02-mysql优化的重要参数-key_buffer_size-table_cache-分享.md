---
title: mysql优化的重要参数 key_buffer_size table_cache 分享
author: admin
type: post
date: 2010-07-02T00:49:01+00:00
url: /archives/4246
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql优化

---
MySQL服务器端的参数有很多，但是对于大多数初学者来说，众多的参数往往使得我们不知所措，但是哪些参数是需要我们调整的，哪些对服务器的性能影响最大呢？对于使用Myisam存储引擎来说，主要有key\_buffer\_size和table\_cache两个参数。对于InnoDB引擎来说主要还是以innodb\_开始的参数，也很好辨认。
查看MySQL参数，可以使用**show variables**和**show status**命令查看，前者查看服务器静态参数，即在数据库启动后不会动态更改的值，比如缓冲区、字符集等。后者查看服务器的动态运行状态信息，即数据库运行期间动态变化的信息，比如锁，当前连接数等。


key\_buffer\_size这个参数是用来设置索引块（index blocks）缓存的大小，它被所有线程共享，严格说是它决定了数据库索引处理的速度，尤其是索引读的速度。那我们怎么才能知道key\_buffer\_size的设置是否合理呢，一般可以检查状态值Key\_read\_requests和Key\_reads，比例key\_reads / key\_read\_requests 应该尽可能的低，比如1:100，1:1000 ，1:10000。其值可以用以i下命令查得：

> mysql> show status like ‘key_read%’;
> +——————-+————+
> | Variable_name | Value |
> +——————-+————+
> | Key\_read\_requests | 3916880184 |
> | Key_reads | 1014261 |
> +——————-+————+
> 2 rows in set (0.00 sec)

3916880184/1024/1024=?M //单位为兆

我的key\_buffer\_size值为：

> key\_buffer\_size=536870912/1024/1024=512M
> key\_reads / key\_read_requests=1014261: 3916880184≈1:4000

照上面来看，健康状况还行。
table\_cache指定表高速缓存的大小。每当MySQL访问一个表时，如果在表缓冲区中还有空间，该表就被打开并放入其中，这样可以更快地访问表内容。通过检查峰值时间的状态值Open\_tables和Opened\_tables，可以决定是否需要增加table\_cache的值。如果你发现open\_tables等于table\_cache，并且opened\_tables在不断增长，那么你就需要增加table\_cache的值了（上述状态值可以使用SHOW STATUS LIKE ‘Open%tables’获得）。注意，不能盲目地把table_cache设置成很大的值。如果设置得太高，可能会造成文件描述符不足，从而造成性能不稳定或者连接失败。

open_tables表示当前打开的表缓存数，如果执行 flush tables操作，则此系统会关闭一些当前没有使用的表缓存而使得此状态值减小；
opend_tables表示曾经打开的表缓存数，会一直进行累加，如果执行flush tables操作，值不会减小。
在mysql默认安装情况下，table\_cache的值在2G内存以下的机器中的值默认时256到512，如果机器有4G内存,则默认这个值是2048，但这决意味着机器内存越大，这个值应该越大，因为table\_cache加大后，使得mysql对SQL响应的速度更快了，不可避免的会产生更多的死锁（dead lock），这样反而使得数据库整个一套操作慢了下来，严重影响性能。所以平时维护中还是要根据库的实际情况去作出判断，找到最适合你维护的库的 table_cache值。
就是table_cache加大后碰到文件描述符不够用的问题，在mysql的配置文件中有这么一段提示：
引用
“The number of open tables for all threads. Increasing this value increases the number of file descriptors that mysqld requires.
Therefore you have to make sure to set the amount of open files allowed to at least 4096 in the variable “open-files-limit” in” section [mysqld_safe]”
说的就是要注意这个问题，一想到这里，部分兄弟可能会用ulimit -n 作出调整，但是这个调整实际是不对的，换个终端后，这个值又会回到原始值，所以最好用sysctl或者修改/etc/sysctl.conf文件，同时还要在配置文件中把open\_files\_limit这个参数增大，对于4G内存服务器，相信现在购买的服务器都差不多用4G的了，那这个 open\_files\_limit至少要增大到4096，如果没有什么特殊情况，设置成8192就可以了。
innodb\_buffer\_pool\_size 这个参数和MyISAM的key\_buffer_size有相似之处，但也是有差别的。这个参数主要缓存innodb表的索引，数据，插入数据时的缓冲。为 Innodb加速优化首要参数。

该参数分配内存的原则：这个参数默认分配只有8M，可以说是非常小的一个值。如果是一个专用ＤＢ服务器，那么他可以占到内存的70%-80%。这个参数不能动态更改，所以分配需多考虑。分配过大，会使Swap占用过多，致使Mysql的查询特慢。如果你的数据比较小，那么可分配是你的数据大小＋１０％左右做为这个参数的值。