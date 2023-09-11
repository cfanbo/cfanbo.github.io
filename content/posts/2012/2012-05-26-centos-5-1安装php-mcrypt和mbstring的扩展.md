---
title: CentOS 5.1安装php mcrypt和mbstring的扩展
author: admin
type: post
date: 2012-05-26T14:02:49+00:00
url: /archives/13053
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos

---

还是先说系统及php相应的版本：

CentOS 5.1 内核 2.6.18； php 5.2.6 ；phpmyadmin3.2.2

其 实大家遇到的问题是phpmyadmin 3.2.2 这个版本需要更高的php版本来支持，当然CentOS 5.1 5.2 5.3这些版本的php都是5.1的，所以问题就自然来了。如果我们要安装php－mcrypt和php－mbstring 的扩展，用系统自带的yum 来升级安装是不行的。但是用rpm 或源码安装也是没有问题的。可是我从网上找了很多帖子不是这里有问题就是那里不行。今天就尝试下看是否有更快捷的方法。

结果还真是让我三番五次的试出来了，下面我就给大家说明下。

因为很多博客系统和网站都需要一个GD库的支持，默认情况下很多是不直接支持的，需要我们单独安装，所以为了升级安装我的php版本我从网上找了一个yum升级的源：http://www.jasonlitka.com

这样我们要具体做得就是更改 /etc/yum.reposd 里面的文件，我们先把原有的文件全部进行重命名的备份，然后新件一个 .repo后缀的文件 名字自己随便起，在这个文件中添加内容如下：


> [utterramblings]
>
> name=Jason’s Utter Ramblings Repo
>
> baseurl=http://www.jasonlitka.com/media/EL$releasever/$basearch/
>
> enabled=1
>
> gpgcheck=1
>
> gpgkey=http://www.jasonlitka.com/media/RPM-GPG-KEY-jlitka

最好先复制到一个文本文档中，免得编码不同有问题。

好了那就开始yum 升级需要的包吧！这里我建议大家现查询下自己系统已经安装的php版本


shell>rpm -aq|grep php


然后执行相应版本的升级


> shell>yum install php* 或shell>yum install php-devel

首先就是要升级我们的php 等升级完成以后就可以升级相应的包了。


> shell>yum install php-gd
>
> shell>yum install php-mcrypt
>
> shell>yum install php-mbstring

OK 等所有的都升级完以后我们用php －m查看下加载情况：


> shell>php -m
>
> bz2
>
> calendar
>
> ctype
>
> curl
>
> date
>
> dbase
>
> exif
>
> filter
>
> ftp
>
> gd
>
> gettext
>
> gmp
>
> hash
>
> iconv
>
> json
>
> ldap
>
> libxml
>
> mbstring
>
> mcrypt
>
> memcache
>
> mysql
>
> mysqli
>
> odbc
>
> openssl
>
> pcntl
>
> pcre
>
> PDO
>
> pdo_mysql
>
> PDO_ODBC
>
> pdo_sqlite
>
> posix
>
> pspell
>
> readline
>
> Reflection
>
> session
>
> shmop
>
> SimpleXML
>
> sockets
>
> SPL
>
> standard
>
> sysvmsg
>
> sysvsem
>
> sysvshm
>
> tokenizer
>
> wddx
>
> xml
>
> Zend Optimizer
>
> zip
>
> zlib
>
>
> [Zend Modules]
>
> Zend Extension Manager
>
> Zend Optimizer

####################################

看到了那些必须的包以后就可以重启apache了


> shell>service httpd restart

到这里就可以登录phpmyadmin看看还有那些烦人的提示吗！