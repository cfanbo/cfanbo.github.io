---
title: CentOS下安装lighttpd
author: admin
type: post
date: 2011-07-19T03:44:41+00:00
url: /archives/10477
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - lighttpd

---
在向大家详细介绍CentOS lighttpd安装之前，首先让大家了解下CentOS系统作用，然后全面介绍CentOS lighttpd安装，CentOS社区不断与其他的同类社区合并，使CentOS Linux逐渐成为使用最广泛的RHEL兼容版本。CentOS Linux的稳定性不比RHEL差，唯一不足的就是缺乏技术支持，因为它是由社区发布的免费版。希望对大家有用。

CentOS lighttpd安装

> wget http://www.lighttpd.net/download/lighttpd-1.4.19.tar.gz
> tar zxvf lighttpd*
> cd lightt*
> ./configure –prefix=/usr/local/lighttpd –with-pcre

CentOS lighttpd安装这时候说缺少pcre-devel

> yum install pcre-devel
> ./configure –with-pcre
> make
> make install

在ubuntu下用apt-get install lighttpd来安装,方便了很多,CentOS lighttpd安装就要自己配置了.

> cp doc/sysconfig.lighttpd /etc/sysconfig/lighttpd
> mkdir /etc/lighttpd

然后copy默认的配置文件

> cp doc/lighttpd.conf /etc/lighttpd/lighttpd.conf
> cp doc/rc.lighttpd.redhat /etc/init.d/lighttpd

然后用whereis lighttpd找到位置在/usr/local/sbin/lighttpd
在用vim /etc/init.d/lighttpd,找到prog=”lighttpd”节,注释掉默认的,添加：

> lighttpd=”/usr/local/sbin/lighttpd”

在/var/log/下mkdir lighttpd,再创建两个文件access.log和error.log

默认的网站目录配置是：

> server.document-root = “/srv/www/htdocs/”

以上介绍CentOS lighttpd安装。

lighttpd.conf: