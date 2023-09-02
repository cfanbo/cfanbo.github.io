---
title: Linux下的Memcache安装
author: admin
type: post
date: 2011-06-15T08:31:01+00:00
url: /archives/9841
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux
 - memcache

---
服务器端主要是安装memcache服务器端，目前的最新版本是 memcached-1.3.0 。
下载：http://www.danga.com/memcached/dist/memcached-1.2.2.tar.gz
另外，Memcache用到了libevent这个库用于Socket的处理，所以还需要安装libevent，libevent的最新版本是libevent-1.3。（如果你的系统已经安装了libevent，可以不用安装）
官网： [http://www.monkey.org/~provos/libevent/](http://www.monkey.org/~provos/libevent/)
下载： [http://www.monkey.org/~provos/libevent-1.3.tar.gz](http://www.monkey.org/~provos/libevent-1.3.tar.gz)

用wget指令直接下载这两个东西.下载回源文件后。
1.先安装libevent。这个东西在配置时需要指定一个安装路径，即./configure –prefix=/usr；然后make；然后make install；

2.再安装memcached，只是需要在配置时需要指定libevent的安装路径即./configure –with-libevent=/usr；然后make；然后make install；

这样就完成了Linux下Memcache服务器端的安装。

详细的方法如下：

> 1.分别把memcached和libevent下载回来，放到 /tmp 目录下：
> \# cd /tmp
> \# wget [http://www.danga.com/memcached/dist/memcached-1.2.0.tar.gz](http://www.danga.com/memcached/dist/memcached-1.2.0.tar.gz)
> \# wget [http://www.monkey.org/~provos/libevent-1.2.tar.gz](http://www.monkey.org/~provos/libevent-1.2.tar.gz)
>
> 2.先安装libevent：
> \# tar zxvf libevent-1.2.tar.gz
> \# cd libevent-1.2
> \# ./configure –prefix=/usr
> \# make
> \# make install
>
> 3.测试libevent是否安装成功：
> \# ls -al /usr/lib | grep libevent
> lrwxrwxrwx 1 root root 21 11?? 12 17:38 libevent-1.2.so.1 -> libevent-1.2.so.1.0.3
> -rwxr-xr-x 1 root root 263546 11?? 12 17:38 libevent-1.2.so.1.0.3
> -rw-r–r– 1 root root 454156 11?? 12 17:38 libevent.a
> -rwxr-xr-x 1 root root 811 11?? 12 17:38 libevent.la
> lrwxrwxrwx 1 root root 21 11?? 12 17:38 libevent.so -> libevent-1.2.so.1.0.3
> 还不错，都安装上了。
>
> 4.安装memcached，同时需要安装中指定libevent的安装位置：
> \# cd /tmp
> \# tar zxvf memcached-1.2.0.tar.gz
> \# cd memcached-1.2.0
> \# ./configure –with-libevent=/usr
> \# make
> \# make install
> 如果中间出现报错，请仔细检查错误信息，按照错误信息来配置或者增加相应的库或者路径。
> 安装完成后会把memcached放到 /usr/local/bin/memcached ，
>
> 5.测试是否成功安装memcached：
> \# ls -al /usr/local/bin/mem*
> -rwxr-xr-x 1 root root 137986 11?? 12 17:39 /usr/local/bin/memcached
> -rwxr-xr-x 1 root root 140179 11?? 12 17:39 /usr/local/bin/memcached-debug

**安装Memcache的PHP扩展**
1.在 [http://pecl.php.net/package/memcache](http://pecl.php.net/package/memcache) 选择相应想要下载的memcache版本。
2.安装PHP的memcache扩展

> tar vxzf memcache-2.2.1.tgz
> cd memcache-2.2.1
> /usr/local/php/bin/phpize
> ./configure –enable-memcache –with-php-config=/usr/local/php/bin/php-config –with-zlib-dir
> make
> make install

3.上述安装完后会有类似这样的提示：

> Installing shared extensions: /usr/local/php/lib/php/extensions/no-debug-non-zts-2007xxxx/

4.把php.ini中的extension_dir = “./”修改为

> extension_dir = “/usr/local/php/lib/php/extensions/no-debug-non-zts-2007xxxx/”

5.添加一行来载入memcache扩展：extension=memcache.so

**memcached的基本设置**：
1.启动Memcache的服务器端：
\# /usr/local/bin/memcached -d -m 10 -u root -l 192.168.0.200 -p 12000 -c 256 -P /tmp/memcached.pid

> -d选项是启动一个守护进程，
> -m是分配给Memcache使用的内存数量，单位是MB，我这里是10MB，
> -u是运行Memcache的用户，我这里是root，
> -l是监听的服务器IP地址，如果有多个地址的话，我这里指定了服务器的IP地址192.168.0.200，
> -p是设置Memcache监听的端口，我这里设置了12000，最好是1024以上的端口，
> -c选项是最大运行的并发连接数，默认是1024，我这里设置了256，按照你服务器的负载量来设定，
> -P是设置保存Memcache的pid文件，我这里是保存在 /tmp/memcached.pid，

2.如果要结束Memcache进程，执行：

> \# kill \`cat /tmp/memcached.pid\`

也可以启动多个守护进程，不过端口不能重复。

3.重启apache

> service httpd restart

**Memcache环境测试**：
运行下面的php文件，如果有输出This is a test!，就表示环境搭建成功。开始领略Memcache的魅力把！

>  $mem = new Memcache;
> $mem->connect(“127.0.0.1″, 11211);
> $mem->set(‘key’, ‘This is a test!’, 0, 60);
> $val = $mem->get(‘key’);
> echo $val;
> ?>

**参考资料**：
对Memcached有疑问的朋友可以参考下列文章：
[Linux下的Memcache安装][1]：http://www.ccvita.com/257.html
[Windows下的Memcache安装][2]：http://www.ccvita.com/258.html
[Memcache基础教程][3]：http://www.ccvita.com/259.html
[Discuz!的Memcache缓存实现][4]：http://www.ccvita.com/261.html
[Memcache协议中文版][5]：http://www.ccvita.com/306.html
[Memcache分布式部署方案][6]：http://www.ccvita.com/395.html

来源:

 [1]: http://www.ccvita.com/257.html
 [2]: http://www.ccvita.com/258.html
 [3]: http://www.ccvita.com/259.html
 [4]: http://www.ccvita.com/261.html
 [5]: http://www.ccvita.com/306.html
 [6]: http://www.ccvita.com/395.html