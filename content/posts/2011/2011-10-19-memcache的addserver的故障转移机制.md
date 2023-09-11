---
title: memcache的addServer的故障转移机制
author: admin
type: post
date: 2011-10-19T04:47:47+00:00
url: /archives/11782
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - memcache

---
如果有多台memcached服务器端（当然其他支持memcache协议的服务前端都可以，比如Tokyo Tyrant），最好使用$memcache->addServer 来连接服务前端。

**连接示例：**

```
$mem_conf = array(
    array('host'=>'192.168.0.11', 'port'=>'11211'),
    array('host'=>'192.168.0.12', 'port'=>'11211'),
    array('host'=>'192.168.0.13', 'port'=>'11211')
);

$memcache = new Memcache ( );
foreach ( $mem_conf as $v ) {
    $memcache->addServer ( $v ['host'], $v ['port'] );
}

```

使用$memcache->addServer 而不是 $memcache->connect 去连接 Memcached 服务器，是因为当 Memcache 客户端使用 addServer 服务器池时，是根据“crc32(key) % current_server_num”哈希算法将 key 哈希到不同的服务器的，PHP、C 和 python 的客户端都是如此的算法。

如:

```
$mem = new Memcache;
$mem->addServer('192.168.0.71', 11211);
$mem->addServer('192.168.0.72', 11211);
$mem->addServer('192.168.0.73', 11211);
for($i=0; $i < 100; $i++)
	$mem->set('key'.$i, md5($i).'This is a memcached test!', 0, 60);
}
```

然后我们通过 [memadmin](http://www.junopen.com/memadmin) 工具可以看到上面的key被分布存储在这三台Memcached上了(根据算法自动计算).

Memcache 客户端的 addserver 具有故障转移机制，当 addserver 了2台 Memcached 服务器，而其中1台宕机了，那么 current\_server\_num 会由原先的2变成1。(2011-10-11个人测试的时候发现无法实现此功能,第二台Memcached不可用的时候,会造成正常要保存到这台server的数据会丢失的现象)

引用 memcached 官方网站和 PHP 手册中的两段话：
引用

If a host goes down, the API re-maps that dead host’s requests onto the servers that are available.


Failover may occur at any stage in any of the methods, as long as other servers are available the request the user won’t notice. Any kind of socket or Memcached server level errors (except out-of-memory) may trigger the failover. Normal client errors such as adding an existing key will not trigger a failover.

**二、repcached实现memcached的复制功能**

repcached是日本人开发的实现memcached复制功能，它是一个单 master单 slave的方案，但它的 master/slave都是可读写的，而且可以相互同步，如果 master坏掉， slave侦测到连接断了，它会自动 listen而成为 master；而如果 slave坏掉， master也会侦测到连接断，它就会重新 listen等待新的 slave加入

安装：先安装memcached(我安装的1.2.8)

有两种方式：

方式一、下载对应的repcached版本

> #wget http://downloads.sourceforge.net/repcached/memcached-1.2.8-repcached-2.2.tar.gz
> #tar zxf memcached-1.2.8-repcached-2.2.tar.gz
> #cd memcached-1.2.8-repcached-2.2

方式二、下载对应patch版本

> #wget http://downloads.sourceforge.net/repcached/repcached-2.2-1.2.8.patch.gz
> #gzip -cd ../repcached-2.2-1.2.8.patch.gz | patch -p1
> #./configure –enable-replication
> # make
> # make install

启动：

> 启动master
>
> #memcached -v -l 192.168.50.240 -p 11211 -u root
> replication: listen (master监听)启动salve
> #memcached -v -l 192.168.50.241 -p 11213 -u root -x 127.0.0.1 -X 11212
> replication: connect (peer=192.168.50.240:11212)
> replication: marugoto copying
> replication: start启动正常后，master将accept。

测试：

> 操作master
>
> #telnet 192.168.50.240 11211
> #set key1 0 0 3
>
> 111查看slave
>
> #telnet 192.168.50.241 11213
> #get key1如果正常表示，配置成功应用：

可以实现cache冗余注意：如果master down机，slave接管并成为master，这时down机的master只能启用slave,他们之间互换角色，才能保持复制功能。换句话说，master没有抢占功能。

**相关教程:**

Memcache基础教程: