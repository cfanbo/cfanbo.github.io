---
title: CentOS 5下Memcached安装
author: admin
type: post
date: 2011-10-13T03:59:26+00:00
url: /archives/11664
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - memcached

---
参考前面的教程：安装了lnmp环境，这里要安装memcached．但在编译的时候提示需要指定libevent库，可是在安装lnmp的候默认是已经安装过的．解决办法如下：

memcached需要libevent支持，所以首先安装libevent

查看系统是否已经安装libevent

> \# rpm -qa|grep libevent

如果有，不要高兴，先升级

> #yum -y install libevent libevent-devel

测试libevent是不是已经安装成功

> #ls -al /usr/lib | grep libevent

可以看到多个已经安装的类包 **安装memcached( [http://memcached.org/](http://memcached.org/))**

可以先查看编译参数

>

> [shell]wget http://memcached.googlecode.com/files/memcached-1.4.15.tar.gz

tar zxvf memcached-1.4.15.tar.gz

cd memcached-1.4.15

./configure –help

./configure –prefix=/usr/local/memcached

make

make install[/shell]

>

在这个时候，不一定会编译通过，依旧会出现:

>

> checking for libevent directory… configure: error: libevent is required. You can get it from http://www.monkey.org/~provos/libevent/

If it’s already installed, specify its path using –with-libevent=/dir/
>

因为libevent 这个包是系统默认安装的，没有安装相应的开发所用的头文件。

所以，还要使用如下命令来安装：

>

> [shell]yum -y install libevent-devel[/shell]

>

再编辑，即可通过。。

启用Memcached,参考： [http://blog.haohtml.com/archives/364](http://blog.haohtml.com/archives/364)

[shell]/usr/local/memcached/bin/memcached -d -m 128 -l 192.168.1.1 -p 11211 -u root[/shell]

============================================

memcached 启动报error while loading shared libraries: libevent-1.4.s解决办法:


原因是找不到libevent-1.4.so.2类库，解决办法如下：


使用LD_DEBUG=help ./memcached -v来确定 加载的类库路径,方法如下:


> [shell]ln -s /usr/local/lib/libevent-1.4.so.2 /lib/libevent-1.4.so.2[/shell]

貌似用ldconfig也能解决问题