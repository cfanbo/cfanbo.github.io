---
title: CentOS下安装lnmp(Nginx+PHP+MySQL)fpm
author: admin
type: post
date: 2010-09-18T13:14:37+00:00
url: /archives/5732
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - nginx

---

PHP 5.3.1


MySQL 5.0.89


Nginx 0.8.33 或 0.7.65 （可选）


这个可比网上流传的什么一键安装包要好得多，强烈推荐此法安装，适合所有菜鸟和高手。我服务器上全用的源代码编译安装，也好不到哪去，还很费劲。我这个装完已经包含 php 的一些常用扩展， PDO，eaccelerator，memcache，tidy等等。


CentOS 最小化安装，然后先新建一个 repo


> # vi /etc/yum.repos.d/centos.21andy.com.repo

放入如下内容


[21Andy.com]

name=21Andy.com Packages for Enterprise Linux 5 – $basearch

baseurl=http://www.21andy.com/centos/5/$basearch/

enabled=1

gpgcheck=0

protect=1

或者使用中国科技大学的yum源:


> #cd /etc/yum.repos.d
>
>
> #mv CentOS-Base.repo  CentOS-Base.repo.save
>
>
> #wget http://centos.ustc.edu.cn/CentOS-Base.repo
>
>
> 也可以使用http://mirrors.163.com/.help/CentOS-Base-163.repo (网易的,也不错的)

启用 EPEL repo


CentOS i386 输入如下命令


> rpm -ihv [http://download.fedora.redhat.com/pub/epel/5/i386/epel-release-5-4.noarch.rpm](http://download.fedora.redhat.com/pub/epel/5/i386/epel-release-5-4.noarch.rpm)

CentOS x86_64 输入如下命令

> rpm -ihv [http://download.fedora.redhat.com/pub/epel/5/x86_64/epel-release-5-4.noarch.rpm](http://download.fedora.redhat.com/pub/epel/5/x86_64/epel-release-5-4.noarch.rpm)

然后导入key


> rpm –import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL

OK，一键安装吧


yum -y install nginx mysql-server php-fpm php-cli php-pdo php-mysql php-mcrypt php-mbstring php-gd php-tidy php-xml php-xmlrpc php-pear php-pecl-memcache php-eaccelerator

最后


> yum -y update

一下，全是最新的.


如果 nginx 你要用 0.7.65 最新稳定版，把


yum -y install nginx


换成


yum -y install nginx-stable


就可以了


装完你已经可以这样玩了（注意此教程对于i386平台无法安装php-fpm，提示包不存在！）


> service mysqld start
>
>
> service php-fpm start
>
>
> service nginx start

别忘了设置开机启动


> chkconfig –level 345 mysqld on
>
>
> chkconfig –level 345 php-fpm on
>
>
> chkconfig –level 345 nginx on

配置文件都在 /etc 下自己找


看看安装多自动


Dependencies Resolved

==========================================================

Package Arch Version Repository Size

==========================================================

Installing:

mysql x86_64 5.0.89-1.el5 21Andy.com 3.5 M

mysql-server x86_64 5.0.89-1.el5 21Andy.com 10 M

nginx x86_64 0.8.33-3.el5 21Andy.com 422 k

php-cli x86_64 5.3.1-2.el5 21Andy.com 2.4 M

php-eaccelerator x86_64 2:0.9.6-1.el5 21Andy.com 118 k

php-fpm x86_64 5.3.1-2.el5 21Andy.com 1.2 M

php-gd x86_64 5.3.1-2.el5 21Andy.com 110 k

php-mbstring x86_64 5.3.1-2.el5 21Andy.com 1.1 M

php-mcrypt x86_64 5.3.1-2.el5 21Andy.com 27 k

php-mysql x86_64 5.3.1-2.el5 21Andy.com 84 k

php-pdo x86_64 5.3.1-2.el5 21Andy.com 91 k

php-pear noarch 1:1.9.0-1.el5 21Andy.com 420 k

php-pecl-memcache x86_64 2.2.5-3.el5 21Andy.com 44 k

php-tidy x86_64 5.3.1-2.el5 21Andy.com 31 k

php-xml x86_64 5.3.1-2.el5 21Andy.com 115 k

php-xmlrpc x86_64 5.3.1-2.el5 21Andy.com 48 k

Installing for dependencies:

gmp x86_64 4.1.4-10.el5 base 201 k

libXaw x86_64 1.0.2-8.1 base 329 k

libXmu x86_64 1.0.2-5 base 63 k

libXpm x86_64 3.5.5-3 base 44 k

libedit x86_64 2.11-2.20080712cvs.el5 epel 80 k

libmcrypt x86_64 2.5.8-4.el5.centos extras 105 k

libtidy x86_64 0.99.0-14.20070615.el5 epel 140 k

php-common x86_64 5.3.1-2.el5 21Andy.com 554 k

sqlite2 x86_64 2.8.17-5.el5 21Andy.com 165 k

t1lib x86_64 5.1.1-7.el5 epel 208 k

Updating for dependencies:

libevent x86_64 1.4.12-1.el5 21Andy.com 129 k

Transaction Summary

==========================================================

Install 26 Package(s)

Update 1 Package(s)

Remove 0 Package(s)

**查看Nginx + php-fpm Benchmark 性能测试**

以下分别测试我本地的虚拟机和 VPS 上 Nginx + php-fpm 的性能


我的本机虚拟机测试，配置为PD930 双核3.0G，2G内存，给虚拟机分配的是 1G 内存，安装的系统为 **CentOS 5.4 64bit**

**测试内容为**

**500** 并发测试，CPU使用率到了30%，系统负载在 **10** 左右，页面打开还是飞快


> [root@localhost ~]# **webbench -c 500 -t 30** http://127.0.0.1/
>
> Webbench – Simple Web Benchmark 1.5
>
> Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.
>
>
> Benchmarking: GET http://127.0.0.1/
>
> 500 clients, running 30 sec.
>
>
> Speed= **223504 pages/min**, 21806556 bytes/sec.
>
> Requests: 111752 susceed, 0 failed.

**2000** 并发测试，CPU使用率35%，系统负载在 **18** 左右，页面打开还是飞快


> [root@localhost ~]# **webbench -c 2000 -t 30** http://127.0.0.1/
>
> Webbench – Simple Web Benchmark 1.5
>
> Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.
>
>
> Benchmarking: GET http://127.0.0.1/
>
> 2000 clients, running 30 sec.
>
>
> Speed= **429494 pages/min**, 39004788 bytes/sec.
>
> Requests: 214747 susceed, 0 failed.

**5000** 并发测试，CPU使用率30%，系统负载到了 **35**，页面打还速度还不错，看了这数据，前些天说的那个1500万PHP请求也没啥了


> [root@localhost ~]# **webbench -c 5000 -t 30** http://127.0.0.1/
>
> Webbench – Simple Web Benchmark 1.5
>
> Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.
>
>
> Benchmarking: GET http://127.0.0.1/
>
> 5000 clients, running 30 sec.
>
>
> Speed= **788986 pages/min**, 66952700 bytes/sec.
>
> Requests: 394493 susceed, 0 failed.

还不过瘾，变态一下，10000并发


**10000** 并发，CPU使用还是不到30%，系统负载从 **60** 左右一直升到 **1000** 左右，晕死！居然还能打开！只是有点卡！负载到 **600** 多的时候居然不卡！疯了，我这还是虚拟机，webbench 还是在自己机上开的，汗，太强了


> [root@localhost ~]# **webbench -c 10000 -t 30** http://127.0.0.1/
>
> Webbench – Simple Web Benchmark 1.5
>
> Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.
>
>
> Benchmarking: GET http://127.0.0.1/
>
> 10000 clients, running 30 sec.
>
>
> Speed= **1513718 pages/min**, -17973622 bytes/sec.
>
> Requests: 756859 susceed, 0 failed.

而我的 VPS , 2G内存，8核CPU测试，但我不是使用上面的 yum 安装，而是全用源代码编译安装的，测试结果如下：


500并发，CPU使用率20%，负载2左右


> # **webbench -c 500 -t 30** http://127.0.0.1/index.php
>
> Webbench – Simple Web Benchmark 1.5
>
> Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.
>
>
> Benchmarking: GET http://127.0.0.1/index.php
>
> 500 clients, running 30 sec.
>
>
> Speed= **120520 pages/min**, -36244332 bytes/sec.
>
> Requests: 60260 susceed, 0 failed.

2000并发，CPU使用率20%左右，负载2左右，没啥变化


> **webbench -c 2000 -t 30** http://127.0.0.1/index.php
>
> Webbench – Simple Web Benchmark 1.5
>
> Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.
>
>
> Benchmarking: GET http://127.0.0.1/index.php
>
> 2000 clients, running 30 sec.
>
>
> Speed= **111454 pages/min**, -44285944 bytes/sec.
>
> Requests: 55727 susceed, 0 failed.

开到3000并发也一样，但打开页面要等几秒，突然一下出来，说明我进程开少了，还有余地。


现在我明白了前几天那个1500万PHP请求还能稳定访问是怎么回事了，哈哈，你只要CentOS 5.4 64bit，再按我上面的 yum 方法安装，也一样能顶住。


fastcgi-php运行模式:[http://blog.haohtml.com/index.php/archives/5750](http://blog.haohtml.com/index.php/archives/5750)