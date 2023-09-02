---
title: mysql memcached UDF安装使用［教程］
author: admin
type: post
date: 2011-06-27T01:42:03+00:00
url: /archives/10022
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - memcached
 - mysql
 - udf

---
在Centos5.6下通过验证！

官方网站：

很早之前，就看到了通过mysql UDF 更新memcached ，原来也研究过一段时间，只是没有来得及写个文档，导致后来工作中，经常要google，搜索其安装，使用的方法，刹时麻烦，今天总结一下：

**1：mysql memcached UD介绍**

mysql memcached UDF 其实就是通过libmemcached来使用memcache的一系列函数，通过这些函数，你能 对memcache进行get, set, cas, append, prepend, delete, increment, decrement objects操作，如果我们通过mysql trigger来使用这些函数，那么就能通过mysql更好的，更自动的管理memcache!下载地址：

**2:安装方法：**

1）安装memcache和memcached

参考：

2）安装libmemcached（）

> $ wget http://download.tangent.org/libmemcached-0.31.tar.gz
> $ tar -xzvf libmemcached-0.31.tar.gz
> $ cd libmemcached-0.31
> $ ./configure
> $ make

安装的时候，发现新版本的都提示错误的

> wget http://launchpad.net/libmemcached/1.0/0.43/+download/libmemcached-0.43.tar.gz
> tar zxvf libmemcached-0.43.tar.gz
> cd libmemcached-0.43
> ./configure –with-memcached=/usr/local/bin/memcached
> make && make install
> echo “/usr/local/lib” >> /etc/ld.so.conf
> ldconfig

安装完成后，libmemcached 的文件包括：
/usr/local/bin/ 目录下
memcat memcp memdump memerror memflush memrm memslap memstat
都是可执行文件，是一些命令行工具，具体使用，可参考官方文档，或帮助。

/usr/local/include/libmemcached 目录下是该函数库的一些头文件

/usr/local/lib 目录下
libmemcached* 等文件，都是库文件。

/usr/local/share/man1 目录下，有 memcat 等命令行工具的 man 帮助文件。
/usr/local/share/man3 目录下，是函数库的一些帮助文件。

命令行工具中，memstat 可在命令行查看 memcached 服务器的情况，比如：

$ memcat –servers=127.0.0.1:11211

输出的为 memcached 服务器的一些统计数据等。

3）安装memcached\_functions\_mysql

To install the MySQL **memcached** UDFs, download the UDF package from [http://libmemcached.org/](http://libmemcached.org/). Unpack the package and run **configure**to configure the build process. When running **configure**, use the `--with-mysql`option and specify the location of the [**mysql_config**][1] command.安装教程请参考：

> tar zxvf memcached\_functions\_mysql-0.9.tar.gz
> cd memcached\_functions\_mysql-0.9
> ./configure –with-mysql=/usr/local/mysql51/bin/mysql_config
> make && make install

4）拷贝lib文件到mysql的plugin下面

> shell> cp -R /usr/local/lib/libmemcached\_functions\_mysql.* /usr/local/mysql51/lib/mysql/plugin/

5）添加memcache UDF 函数

在mysql里执行(要sql目录里)

> source install_functions.sql

这样我们就可以使用mysql memcached UDF 了，我们可以通过下面语句查看是否已经正常安装

1)查看mysql.func,有很多函数

> mysql> select * from mysql.func;
> +——————————+—–+———————————+———-+
> | name                         | ret | dl                              | type     |
> +——————————+—–+———————————+———-+
> | memc\_add                     |   2 | libmemcached\_functions_mysql.so | function |
> | memc\_add\_by\_key              |   2 | libmemcached\_functions_mysql.so | function |
> | memc\_servers\_set             |   2 | libmemcached\_functions\_mysql.so | function |

2）添加trigger，看是否向memcache里insert、update等

对于验证方法请参考：

> mysql> select memc\_servers\_set(‘127.0.0.1’);
> <8 new client connection
> <8 version
> >8 VERSION 1.2.0
> +——————————-+
> | memc\_servers\_set(‘127.0.0.1’) |
> +——————————-+
> | 0 |
> +——————————-+
> 1 row in set (0.03 sec)
>
> mysql> select memc\_servers\_set(‘127.0.0.1’);
> <8 version
> >8 VERSION 1.2.0
> <9 new client connection
> <9 version
> >9 VERSION 1.2.0
> +——————————-+
> | memc\_servers\_set(‘127.0.0.1’) |
> +——————————-+
> | 0 |
> +——————————-+
> 1 row in set (0.00 sec)
>
> mysql> select memc\_servers\_set(‘127.0.0.1’);
> <8 version
> >8 VERSION 1.2.0
> <9 version
> >9 VERSION 1.2.0
> <10 new client connection
> <10 version
> >10 VERSION 1.2.0
> +——————————-+
> | memc\_servers\_set(‘127.0.0.1’) |
> +——————————-+
> | 0 |
> +——————————-+
> 1 row in set (0.00 sec)
>
> mysql> select memc_set(‘myid’,’atest’);
> <11 new client connection
> <11 set myid 0 0 5
> >11 STORED
> <11 quit
> <11 connection closed.
> +————————–+
> | memc_set(‘myid’,’atest’) |
> +————————–+
> | 1 |
> +————————–+
> 1 row in set (0.00 sec)
>
> mysql> select memc_get(‘myid’);
> <11 new client connection
> <11 get myid
> >11 sending key myid
> >11 END
> <11 quit
> <11 connection closed.
> +——————+
> | memc_get(‘myid’) |
> +——————+
> | atest |
> +——————+
> 1 row in set (0.01 sec)
> mysql>

**总结：**由下面的信息可能看出，每次调用udf的时候，都会连接和关闭memcached．这里还是比较的消费资源的．后面还有注意事项！

**具体的语句，我们可以参照：**

1）memcached\_functions\_mysql-0.9/sql 目录下的trigger_fun.sql

2）使用参照文档：
 ****

**我们还必须注意以下几点：**

1）mysql 编译时一定不要带’–with-mysqld-ldflags=-all-static’ 这个参数，因为这样就限制了mysql 的动态安装功能了

2）使用时，要观察mysql.err日志，不知道是有意还是无意，udf更新memcache都会记录在err日志里，注意清理该日志，否则一下就爆满了

3）mysql 官网有这样一句话：

The list of servers used by the **memcached** UDFs is not persistent over restarts of the MySQL server. If the MySQL server fails, then you must re-set the list of **memcached** servers.

所以，当我们重启mysql，我们必须通过select  memc\_servers\_set(‘192.168.0.1:11211,192.168.0.2:11211’);语句重新注册memcache服务器！



 [1]: http://dev.mysql.com/doc/refman/5.1/en/mysql-config.html "4.7.2. mysql_config — Get Compile Options for Compiling Clients"