---
title: 在 CentOS 装 Git
author: admin
type: post
date: 2011-06-28T06:44:53+00:00
url: /archives/10093
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - git

---
在 Ubuntu 上安装 Git 非常的简单，只需要：

> sudo apt-get install git-core

但是 CentOS 默认的 yum 源中没有 Git，只能下载 RPM 包安装，确保已安装了依赖的包

> sudo yum -y install curl curl-devel zlib-devel openssl-devel perl cpio expat-devel gettext-devel perl-ExtUtils-MakeMaker perl-CPAN tk

安装最新的 Git

> $ wget http://www.codemonkey.org.uk/projects/git-snapshots/git/git-latest.tar.gz
>
> $ tar xzvf git-latest.tar.gz
>
> $ cd git-{date}
>
> $ autoconf
>
> $ ./configure –with-curl=/usr/local
>
> $ make
>
> $ sudo make install

检查版本

> $ git –version
>
> git version 1.7.3.GIT

**常见问题：**

1.如果执行  git –version 的时候，提示

> git: error while loading shared libraries: libiconv.so.2: cannot open shared object file: No such file or directory

错误的话，解决办法如下：

> 1.在/etc/ld.so.conf中加一行/usr/local/lib
>
> 2.然后运行/sbin/ [ldconfig](http://blog.haohtml.com/tag/ldconfig)，文件解决，没有报错

2.如果在安装的过程中遇到＂Can’t locate CPAN.pm in @INC…＂之类的错误，请参考：,我是直接用命令＂yum -y install perl-CPAN＂搞定的．

3.如果遇到错误 ”tclsh failed; using unoptimized loading”。 需要执行一下

# yum -y install gettext

– EOF –

**相关教程：**

CentOS下搭建Git服务器Gitosis:

git相关教程：