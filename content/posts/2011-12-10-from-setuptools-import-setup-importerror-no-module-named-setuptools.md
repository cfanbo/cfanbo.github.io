---
title: FreeBSD/Linux下安装cacti的memcached的监控插件
author: admin
type: post
date: 2011-12-10T04:18:11+00:00
url: /archives/12231
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - CACTI

---
因为python的模板使用了python来获取数据，所以需要安装python环境以及python的memcached客户端

**1.安装ez_setup工具**

> wget -q http://peak.telecommunity.com/dist/ez_setup.py
> python ez_setup.py

**2.安装python的memcached客户端**

> wget ftp://ftp.tummy.com/pub/python-memcached/python-memcached-1.45.tar.gz
> tar -zxvf python-memcached-1.45.tar.gz
> cd python-memcached-1.45
> python setup.py install

**3.下载cacti的memcached模板**

> wget http://content.dealnews.com/dealnews/developers/cacti-memcached-1.0.tar.gz
> tar -zxvf cacti-memcached-1.0.tar.gz
> cd cacti-memcached
> cp memcached.py /var/www/cacti/scripts/

 注意：对于Freebsd系统下的memcached.py脚本文件里的python文件位置有可能根据自己系统的情况时行一些修改．

可以进行测试:

> python /var/www/cacti/scripts/memcached.py 192.168.1.11

会有如下类似输出:

> total\_items:33475 get\_hits:72993 uptime:339594 cmd\_get:72993 time:1274776533 bytes:870220 curr\_connections:11 connection\_structures:14 bytes\_written:44760499 limit\_maxbytes:1073741824 cmd\_set:33527 curr\_items:1377 rusage\_user:2.472154 get\_misses:67152 rusage\_system:6.628414 bytes\_read:6793684 total\_connections:36729

4.在cacti中引入memcached模板,具体可以参考相关 cacti 的模板安装方法

**5.参考资料**

[http://dealnews.com/developers/cacti/memcached.html](http://dealnews.com/developers/cacti/memcached.html) [http://www.docin.com/p-47052887.html](http://www.docin.com/p-47052887.html)