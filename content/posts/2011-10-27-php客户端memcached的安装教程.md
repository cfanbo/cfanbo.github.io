---
title: php客户端memcached的安装教程
author: admin
type: post
date: 2011-10-27T02:10:34+00:00
url: /archives/11873
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - memcache
 - memcached

---
我们在上篇文章里（）介绍了LNMP的安装方法．只安装了memcache客户端．有些用户可能需要memcached这种客户端的．这里介绍一种php客户端memcached的安装方法．

之前在安装memcache时有提到memcached客户端是叫memcache，其实还有一个基于libmemcached的客户端叫memcached，据说性能更好，功能也更多。参考：

memcache的官方主页： [http://pecl.php.net/package/memcache](http://pecl.php.net/package/memcache)
memcached的官方主页： [http://pecl.php.net/package/memcached](http://pecl.php.net/package/memcached)

以下是我安装Memcached版本的PHP模块的过程记录：

```
#wget http://download.tangent.org/libmemcached-0.9.tar.gz
#tar zxf libmemcached-0.9.tar.gz
#cd libmemcached-0.9
#./configure --prefix=/usr/local/libmemcached --with-memcached
#make
#make install

#wget http://pecl.php.net/get/memcached-1.0.2.tgz
#tar zxf memcached-1.0.2.tgz
#cd memcached-1.0.2
#/usr/local/php/bin/phpize
#./configure --enable-memcached --with-php-config=/usr/local/php/bin/php-config --with-libmemcached-dir=/usr/local/libmemcached --prefix=/usr/local/phpmemcached
#make
#make install
```

在php.ini中加入

> extension=memcached.so

完成

另：在安装libmemcached时，如果只用./configure，可能会提示：

> checking for memcached… no
> configure: error: “could not find memcached binary”

两者使用起来几乎一模一样。

```
$mem = new Memcache;
$mem->addServer($memcachehost, '11211');
$mem->addServer($memcachehost, '11212');
$mem->set('hx','9enjoy');
echo $mem->get('hx');
```

```
$md = new Memcached;
$servers = array(
array($memcachehost, '11211'),
array($memcachehost, '11212')
);
$md->addServers($servers);
$md->set('hx','9enjoy');
echo $md->get('hx');
```

memcached的方法比memcache多不少，比如getMulti，getByKey，addServers等。
memcached没有memcache的connect方法，目前也还不支持长连接。
memcached 支持 Binary Protocol，而 memcache 不支持，意味着 memcached 会有更高的性能。
Memcache是原生实现的，支持OO和非OO两套接口并存，memcached是使用libmemcached，只支持OO接口。
更详细的区别： [http://code.google.com/p/memcached/wiki/PHPClientComparison](http://code.google.com/p/memcached/wiki/PHPClientComparison)

=================================
memcached服务端是集中式的缓存系统，分布式实现方法是由客户端决定的。
memcached的分布算法一般有两种选择：
1、根据hash(key)的结果，模连接数的余数决定存储到哪个节点，也就是hash(key)% sessions.size()，这个算法简单快速，表现良好。然而这个算法有个缺点，就是在memcached节点增加或者删除的时候，原有的缓存数据将大规模失效，命中率大受影响，如果节点数多，缓存数据多，重建缓存的代价太高，因此有了第二个算法。
2、Consistent Hashing，一致性哈希算法，他的查找节点过程如下：
首先求出memcached服务器（节点）的哈希值，并将其配置到0～232的圆（continuum）上。然后用同样的方法求出存储数据的键的哈希值，并映射到圆上。然后从数据映射到的位置开始顺时针查找，将数据保存到找到的第一个服务器上。如果超过2的32次方后仍然找不到服务器，就会保存到第一台memcached服务器上。

memcache在没有任何配置的情况下，是使用第一种方法。memcached要实现第一种方法，似乎是使用(未确认)：
$md->setOption(Memcached::OPT\_HASH, Memcached::HASH\_CRC);

第二种一致性哈希算法：

memcache在php.ini中加

Memcache.hash_strategy =consistent

Memcache.hash_function =crc32

memcached在程序中加(未确认)

$md->setOption(Memcached::OPT_DISTRIBUTION, Memcached::DISTRIBUTION_CONSISTENT);

$md->setOption(Memcached::OPT_HASH, Memcached::HASH_CRC);

或

$mem->setOption(Memcached::OPT_DISTRIBUTION,Memcached::DISTRIBUTION_CONSISTENT);

$mem->setOption(Memcached::OPT_LIBKETAMA_COMPATIBLE,true);

一些参考文档:
memcached分布测试报告（一致性哈希情况下的散列函数选择）： [http://www.iteye.com/topic/346682](http://www.iteye.com/topic/346682)
php模块memcache和memcached区别： [http://hi.baidu.com/dong_love_yan/blog/item/afbe1e12d22e7512203f2e21.html](http://hi.baidu.com/dong_love_yan/blog/item/afbe1e12d22e7512203f2e21.html)
PHP模块：Memcached > Memcache： [http://hi.baidu.com/thinkinginlamp/blog/item/717cd42a11f6e491023bf67a.html](http://hi.baidu.com/thinkinginlamp/blog/item/717cd42a11f6e491023bf67a.html)

20110509@@UPDATE：
如果安装libmemcached有如下出错提示：
make[2]: \*** [clients/ms_conn.o] Error 1
make[2]: Leaving directory \`/www/soft/libmemcached-0.48′
make[1]: \*** [all-recursive] Error 1
make[1]: Leaving directory \`/www/soft/libmemcached-0.48′
make: \*** [all] Error 2

可在configure时增加–disable-64bit CFLAGS=”-O3 -march=i686″
即：./configure –prefix=/usr/local/libmemcached –with-memcached –disable-64bit CFLAGS=”-O3 -march=i686″