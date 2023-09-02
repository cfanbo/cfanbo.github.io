---
title: windows下memcached的安装（修正版）
author: admin
type: post
date: 2008-09-24T15:54:33+00:00
excerpt: |
 Memcached是什么:
 Memcached是高性能的，分布式的内存对象缓存系统，用于在动态应用中减少数据库负载，提升访问速度。
 Memcached由Danga Interactive开发，用于提升LiveJournal.com访问速度的。LJ每秒动态页面访问量几千次，用户700万。Memcached将数据库负载大幅度降低，更好的分配资源，更快速访问。
 首先我在windows下实现它,过后再在linux试试.
 下载memcached-win32 和php_memcache.dll(要和php的版本对应上)

 1.memcahced下载后,压缩之前发现不到100K,压缩后也不到200K,这东西居然有这么神奇,放到C盘,进入目录里面有一个memcached.exe,双击就启动了,让窗口开着,或者在cmd里面c:\memcached\memcached.exe -d start 都可以启动.
url: /archives/406
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - memcached

---
[**Memcached是什么:**](http://www.danga.com/memcached/)
Memcached是高性能的，分布式的内存对象缓存系统，用于在动态应用中减少数据库负载，提升访问速度。
Memcached由Danga Interactive开发，用于提升LiveJournal.com访问速度的。LJ每秒动态页面访问量几千次，用户700万。Memcached将数据库负载大幅度降低，更好的分配资源，更快速访问。
首先我在windows下实现它,过后再在linux试试.
下载 [memcached-win32](http://www.splinedancer.com/memcached-win32/) 和[php_memcache.dll(要和php的版本对应上)](http://pecl4win.php.net/ext.php/php_memcache.dll),或者 [（点击这里下载memcached for win32）](http://blog.haohtml.com/wp-content/uploads/2008/09/memcached_win.zip)

1.memcahced下载后,压缩之前发现不到100K,压缩后也不到200K,这东西居然有这么神奇,放到d:memcached目录,进入目录里面有一个memcached.exe,双击就启动了,让窗口开着,或者在cmd里面d:memcachedmemcached.exe -d start 都可以启动.

2.修改php.ini的配制文件

加入extension=php_memcache.dll 这一行代码,位置无所谓

嗨，我一直都烦配置的东西，不过好东西还是要学习的，以上是转载好朋友的，现保存在说，不知什么时候就用上了

3.php_memcache.dll放到php的安装文件中,一般在php源码的ext目录下

4.重启apache后,查看一下phpinfo(写一个phpinfo()函数就可以看到)，如果有memcache，那么就说明安装成功

**memcached的基本设置**：

> -p 监听的端口
> -l 连接的IP地址, 默认是本机
> -d start 启动memcached服务
> -d restart 重起memcached服务
> -d stop|shutdown 关闭正在运行的memcached服务
> -d install 安装memcached服务
> -d uninstall 卸载memcached服务
> -u 以的身份运行 (仅在以root运行的时候有效)
> -m 最大内存使用，单位MB。默认64MB
> -M 内存耗尽时返回错误，而不是删除项
> -c 最大同时连接数，默认是1024
> -f 块大小增长因子，默认是1.25
> -n 最小分配空间，key+value+flags默认是48
> -h 显示帮助

**安装memcached服务**

> d:memcachedmemcached -l 127.0.0.1 -m 256 -d install
> d:memcachedmemcached -d start

开始测试一下代码

```
$mem = new Memcache;
$mem->connect(”127.0.0.1″, 11211);
$mem->set(’key’, ‘This is a memcached test!’, 0, 60);
$val = $mem->get(’key’);
echo $val;
```

如果输出This is a memcached test!刚表明安装成功.

Memcached中英文参考手册: [http://docs.haohtml.com/memcached/Memcached.html](http://docs.haohtml.com/memcached/Memcached.html)

**相关教程:**

**
** **Linux下Memcached的安装及配置: [http://blog.haohtml.com/archives/364](http://blog.haohtml.com/archives/364)**

Window下Memcached的安装及使用: