---
title: perl首次安装Can’t locate CPAN.pm in @INC的解决办法
author: admin
type: post
date: 2012-04-07T16:39:01+00:00
url: /archives/12708
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - perl

---
perl -MCPAN -e ‘install “模块名称”

我在在线安装perl 模块时，发现有这样的问题。应该是说没有安装CPAN这个服务吧。

1、执行perl -MCPAN -e shell出错，提示如下:

[root@GM ~]# perl -MCPAN -e shell

Can’t locate CPAN.pm in @INC (@INC contains:……省略

2、到cpan的官方站点下载CPAN模块

[http://search.cpan.org/search?query=CPAN&mode=all](http://search.cpan.org/search?query=CPAN&mode=all)

> [root@GM ~]#wget http://cpan.communilink.net/authors/id/A/AN/ANDK/CPAN-1.9600.tar.gz

3、解压，编绎，安装

> [root@GM ~]# tar -zxvf CPAN-1.9600.tar.gz
> [root@GM ~]#cd CPAN-1.9600
> [root@GM CPAN-1.9600]# perl Makefile.PL
> [root@GM CPAN-1.9600]# make
> [root@GM CPAN-1.9600]# make install

4、成功进入CPAN的shell模式

> [root@GM CPAN-1.9600]# perl -MCPAN -e shell

5、install XXX 安装对应模块

Can’t locate extUtils/makemaker.pm
centos安装cpan先执行
这个错误!可能是没安装perl-CPAN

> yum -y install perl-CPAN