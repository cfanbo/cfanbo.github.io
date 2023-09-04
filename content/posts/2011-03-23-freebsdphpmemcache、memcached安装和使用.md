---
title: freebsd+php+memcache、memcached安装和使用
author: admin
type: post
date: 2011-03-23T06:00:45+00:00
url: /archives/8092
IM_contentdowned:
 - 1
categories:
 - 程序开发
 - 服务器
tags:
 - memcache
 - php

---

来源： [http://www.lifecrunch.biz/archives/55](http://www.lifecrunch.biz/archives/55)

## Memcache 介绍

memcache是一个高性能的分布式的内存对象缓存系统，通过在内存里维护一个统一的巨大的hash表，它能够用来存储各种格式的数据，包括图 像、视频、文件以及数据库检索的结果等。Memcache是danga.com的一个项目，最早是为 LiveJournal 服务的，最初为了加速 LiveJournal 访问速度而开发的，后来被很多大型的网站采用。目前全世界不少人使用这个缓存项目来构建自己大负载的网站，来分担数据库的压力。起初作 者编写它可能是为了提高动态网页应用，为了减轻数据库检索的压力，来做的这个缓存系统。它的缓存是一种分布式的，也就是可以允许不同主机上的多个用户同时 访问这个缓存系统， 这种方法不仅解决了共享内存只能是单机的弊端，同时也解决了数据库检索的压力，最大的优点是提高了访问获取数据的速度！基于memcache作者对分布式 cache的理解和解决方案。 memcache完全可以用到其他地方 比如分布式数据库， 分布式计算等领域。


Memcache官方网站： [http://www.danga.com/memcached](http://www.danga.com/memcached)

## Memcached服务器端安装：

freebsd下服务器端的memcached使用ports安装非常简单，

```
cd /usr/ports/databases/memcached
make install clean
rehash
```

```
安装好memcache以后，编辑/etc/rc.conf文件,添加一行
memcached_enable="YES"
然后保存退出。memcache会随着开机自动启动，手动启动的命令是：
/usr/local/etc/rc.d/memcached start
好了，现在memcache已经安装并启动完毕了。
手动调整服务命令如下:
memcached  -p 11211 -l 172.16.236.150 -d -u nobody -P  /var/run/memcached/memcached.pid -m  64M -c 1024 -vv

```

**几个参数的解释：**

 -p memcached监听的TCP端口

 -l 监听的ip地址，172.16.236.150是我服务器的IP地址，如果你需要多个服务器都能够读取这台memcached的缓存数据，那么就必须设定 这个ip

 -d 以daemon方式运行，将程序放入后台

 -u memcached的运行用户，我设定的是nobody，memcache默认不允许以root用户登录

 -P memcached的pid文件路径

 -m memcached可以使用的最大内存数量

 -c memcached同时可以接受的最大的连接数

如果你希望以socket方式来访问memcached，那么在启动的时候就必须去掉 -l和-p参数，并加上-s参数：

 -s memcached的socket文件路径


 -vv显示debug信息


## PHP Memecache 客户端的安装

PHP下使用Memcache 有3种方式。

1. 使用memcache 扩展 手册上有说明：http://php.net/manual/en/book.memcache.php

2. 使用memcached扩展 手册上有说明：http://php.net/manual/en/book.memcached.php

3. 使用memcache-client.php类库. 网上没有找到具体出处，暂且提供我珍藏的.(稍候上传..)

这三种方式都可以和Memcache缓存系统交互.但是有一些细微差别。


**第一种**，常见方式，这个扩展在Win环境下使用方便只要去下载一个memcache.dll 配置一下 php.ini就可以了。win32平台扩展的下载地址 [www.pureformsolutions.com/pureform.wordpress.com/2008/06/17/php_memcache.dll](http://www.pureformsolutions.com/pureform.wordpress.com/2008/06/17/php_memcache.dll) for PHP 5.2.*


当然freebsd下配置也很简单。


freebsd下安装：


下载 memcache: [http://pecl.php.net/get/memcache-2.2.5.tgz](http://pecl.php.net/get/memcache-2.2.5.tgz)

> tar -zvxf memcache-2.2.5.tgz
>
> cd memcache-2.2.5
>
>
> phpize
>
>
> ./configure
>
>
> make
>
>
> make install

最后在/usr/local/etc/php/extension.ini中加上


> extension=”memcache.so”

安装完毕之后重新启动apache，可以用下面的测试代码检查memcache工作是否正常：


> ```
> <?php
>
> /* OO API */
>
> $memcache = new Memcache;
> $memcache->addServer('192.168.30.192', 11211);
> $memcache->addServer('192.168.30.191', 11211);
>
> $tmp_object = new stdClass;
> $tmp_object->str_attr = 'test';
> $tmp_object->int_attr = 123;
>
> $memcache->set('key', $tmp_object, false, 10) or die ("Failed to save data at the server");
> echo "Store data in the cache (data will expire in 10 seconds)<br/>\n";
>
> $get_result = $memcache->get('key');
> echo "Data from the cache:<br/>\n";
>
> var_dump($get_result);
>  ?>
> ```

**第二种方式**，memcached 的版本比较新，并且使用的是 libmemcached 库。libmemcached 被认为做过更好的优化，应该比 php only 版本的 memcache 有着更高的性能。差别比较大的一点是，memcached 支持 Binary Protocol，而 memcache 不支持，这也意味着 memcached 会有更高的性能。但安装配置起来也比较麻烦.该扩展目前我没有看到能支持Win平台的。所以只能在freebsd下自行编译安装了

以下是linux安装：

1.首先安装libmemcached请参考： [http://www.phpup.net/post/54](http://www.phpup.net/post/54)

2.下载php的memcached: [http://pecl.php.net/get/memcached-1.0.1.tgz](http://pecl.php.net/get/memcached-1.0.1.tgz)

3.安装：


1) [root@localhost soft]# tar -zvxf phpmemcached-1.0.1.tgz


2)cd memcached-1.0.1


3) 在此目录下运行phpize(此处phpize是要安装php的开发包后才会有的，因为phpize是在/usr/bin目录下，所以可直接运行,完了会 在此目录下生成configure等文件)


4)在此目录下运行./configure –with-php-config=/usr/local/php/bin/php-config –with-libmemcached-dir=/usr/local/


5)运行 make && make install （实际安装过程中，这一步编译出错）


6)在/etc/php.ini中加入 extension=”memcached.so” 命令行：


> echo “extension = memcached.so” >> /usr/local/php/etc/php.ini

7）重启apache


**第三种方式**,使用简单方便。适合新手使用，无须任何php配置，只要一个包含就可以使用了.并非底层，所以性能上面自然是稍逊不少。

以下是使用：


> include "memcache-client.php
>
> 文件下载地址: http://www.lifecrunch.biz/wp-content/uploads/2010/06/memcached-client-php-0.1.2.tar.gz

个人推荐使用第一种 memcached 扩展，毕竟性能不错，新手的话推荐使用 第三种 客户端类库，方便实用。第二种方式没有尝试成功，不知道是什么原因，编译安装出错，以后有空再解决吧。


有关Memcache , Memcached 和 MemcacheDB三者的区别: [http://blog.haohtml.com/archives/8089](http://blog.haohtml.com/archives/8089)