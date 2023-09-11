---
title: 'MySQL Memcache_engine的安装与使用[原创]'
author: admin
type: post
date: 2010-04-01T14:31:49+00:00
url: /archives/3180
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - memcache
 - mysql

---
[文章作者：张宴 本文版本：v1.1 最后修改：2008.09.09 转载请注明原文链接： [http://blog.s135.com/post/357/](http://blog.s135.com/post/357/)]

鉴于国内外还没有人撰写如何安装Memcache_engine的文章，于是，我根据自己的编译安装步骤，写下此文。

Memcache_engine是一个MySQL 5.1数据库的存储引擎，它能够让用户通过标准的SQL语句（SELECT/UPDATE/INSERTE/DELETE）访问Memcached（还支 持新浪的 [Memcachedb](http://www.memcachedb.org/)、 [dbcached](http://code.google.com/p/dbcached)）中 存放的数据。

**限制：**
1、Memcache表必须有主键。
2、只能使用主 键去查询，即只能使用SELECT … FROM … WHERE id = … 方式去查询。
3、不支持自增ID。

**安装与使用：**
1、编译安装memcache_engine的步骤：

cd /tmp

wget [http://dev.mysql.com/get/Downloads/MySQL-5.1/mysql-5.1.26-rc.tar.gz/from/http://mirror.x10.com/mirror/mysql/](http://dev.mysql.com/get/Downloads/MySQL-5.1/mysql-5.1.26-rc.tar.gz/from/http://mirror.x10.com/mirror/mysql/)

tar zxvf mysql-5.1.26-rc.tar.gz

#安装、配置MySQL的步骤省略，注意不要以静态方式编译安装。

wget [http://download.tangent.org/libmemcached-0.23.tar.gz](http://download.tangent.org/libmemcached-0.23.tar.gz)

tar zxvf libmemcached-0.23.tar.gz

cd libmemcached-0.23/

./configure –prefix=/usr/local/memcache_engine

make

make install

cd ../


wget [http://xmlsoft.org/sources/libxml2-2.6.32.tar.gz](http://xmlsoft.org/sources/libxml2-2.6.32.tar.gz)

tar zxvf libxml2-2.6.32.tar.gz

cd libxml2-2.6.32/

./configure –prefix=/usr/local/memcache_engine

make

make install

cd ../


wget [http://download.tangent.org/libxmlrow-0.2.tar.gz](http://download.tangent.org/libxmlrow-0.2.tar.gz)

tar zxvf libxmlrow-0.2.tar.gz

cd libxmlrow-0.2/

export PKG_CONFIG_PATH=/usr/local/memcache_engine/lib/pkgconfig/

./configure –prefix=/usr/local/memcache_engine

make

make install

cd ../


wget [http://download.tangent.org/memcache_engine-0.7.tar.gz](http://download.tangent.org/memcache_engine-0.7.tar.gz)

tar zxvf memcache_engine-0.7.tar.gz

cd memcache_engine-0.7/

sed -i “s#uint16_t#uint32_t#g” ./src/ha_memcache.cc

export PKG_CONFIG_PATH=/usr/local/memcache_engine/lib/pkgconfig/

./configure –prefix=/usr/local/memcache_engine –with-mysql=/tmp/mysql-5.1.26-rc

make

make install

cd ../

**注意：** 红色标记部分为MySQL 5.1.22以上版本的源码路径。

2、拷贝 libmemcache_engine.so到MySQL默认插件目录（假设MySQL安装在/usr/local/mysql目录下）：


mkdir -p /usr/local/mysql/lib/mysql/plugin/

cp /usr/local/memcache_engine/lib/libmemcache_engine.so.0.0.0 /usr/local/mysql/lib/mysql/plugin/libmemcache_engine.so


3、安装libmemcache_engine.so插件的SQL语句：


INSTALL PLUGIN memcache SONAME ‘libmemcache_engine.so’;


4、查看libmemcache_engine.so插件是否安装成 功的SQL语句：


SELECT * FROM mysql.plugin;

SHOW PLUGINS;


5、创建一张 memcache_engine表的SQL语句：


CREATE TABLE `table` (

`id` int(11) NOT NULL DEFAULT ‘0’,

`a` int(11) DEFAULT NULL,

`b` int(11) DEFAULT NULL,

PRIMARY KEY (`id`)

) ENGINE=MEMCACHE DEFAULT CHARSET=latin1

CONNECTION=’localhost:11211′;