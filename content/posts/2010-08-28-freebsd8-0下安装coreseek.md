---
title: '[教程]freebsd8.0下安装coreseek'
author: admin
type: post
date: 2010-08-28T04:32:30+00:00
url: /archives/5339
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - coreseek
 - Sphinx

---
一**、安装coreseek**

A、安装环境配置,为安装coreseek做准备

>

> #pkg_add -r autoconf262 automake110 libtool mysql50-client libxml2 expat
>

B、下载整个安装包(内含mmseg,coreseek)：

>

> #fetch http://www.coreseek.cn/uploads/csft/3.2/coreseek-3.2.13.tar.gz
>

>
>

> #tar xzvf coreseek-3.2.13.tar.gz
>

>
>

> #cd coreseek-3.2.13
>

======================================

C **、** 安装coreseek开发的mmseg，为coreseek提供中文分词功能

>

> #cd mmseg-3.2.13
>

>
>

> #./bootstrap
>

>
>

> #./configure –prefix=/usr/local/mmseg3
>

>
>

> #make
>

>
>

> #make install
>

至此,mmseg已经安装完成,下面进入csft-3.2.13目录里进行安装coreseek

======================================

D、安装coreseek

> #cd ../
> #cd csft-3.2.13
> #./configure –prefix=/usr/local/coreseek –without-python –without-mysql –with-mmseg –with-mmseg-includes=/usr/local/mmseg3/include/mmseg/ –with-mmseg-libs=/usr/local/mmseg3/lib/
> #make
> #make install

配置测试，测试是否可以正确运行

>

> #/usr/local/coreseek/bin/indexer -c /usr/local/coreseek/etc/sphinx-min.conf.dist
>

以下为正常测试时的提示信息：

>

> Coreseek Fulltext 3.2 [ Sphinx 0.9.9-release (r2117)]
>

>
>

> Copyright (c) 2007-2010,
>

>
>

> Beijing Choice Software Technologies Inc (http://www.coreseek.com)
>

>
>

> using config file ‘/usr/local/coreseek/etc/sphinx-min.conf.dist’…
>

>
>

> total 0 reads, 0.000 sec, 0.0 kb/call avg, 0.0 msec/call avg
>

>
>

> total 0 writes, 0.000 sec, 0.0 kb/call avg, 0.0 msec/call avg
>

至此，coreseek已经正常安装，我们可以开始后续的工作啦!

**二**、**安装数据源:**

```
##重新安装coreseek，以支持mysql数据源和xml数据源
```

> #cd csft-3.2.13
>
>
> #./configure –prefix=/usr/local/coreseek –with-mmseg –with-mmseg-includes=/usr/local/mmseg3/include/mmseg/ –with-mmseg-libs=/usr/local/mmseg3/lib/
>
>
> #make
>
>
> #make install

**三、coreseek中文全文检索测试**

> #cd testpack
>
>
> # /usr/local/coreseek/bin/indexer -c etc/csft.conf

##以下为正常情况下的提示信息：


> Coreseek Fulltext 3.2 [ Sphinx 0.9.9-release (r2117)]
>
>
> Copyright (c) 2007-2010,
>
>
> Beijing Choice Software Technologies Inc (http://www.coreseek.com)
>
>
> using config file ‘etc/csft.conf’…
>
>
> total 0 reads, 0.000 sec, 0.0 kb/call avg, 0.0 msec/call avg
>
>
> total 0 writes, 0.000 sec, 0.0 kb/call avg, 0.0 msec/call avg

> # /usr/local/coreseek/bin/indexer -c etc/csft.conf –all

##以下为正常索引全部数据时的提示信息：


> Coreseek Fulltext 3.2 [ Sphinx 0.9.9-release (r2117)]
>
>
> Copyright (c) 2007-2010,
>
>
> Beijing Choice Software Technologies Inc (http://www.coreseek.com)
>
>
> using config file ‘etc/csft.conf’…
>
>
> indexing index ‘xml’…
>
>
> collected 3 docs, 0.0 MB
>
>
> sorted 0.0 Mhits, 100.0% done
>
>
> total 3 docs, 7585 bytes
>
>
> total 0.075 sec, 101043 bytes/sec, 39.96 docs/sec
>
>
> total 2 reads, 0.000 sec, 5.6 kb/call avg, 0.0 msec/call avg
>
>
> total 7 writes, 0.000 sec, 3.9 kb/call avg, 0.0 msec/call avg

> # /usr/local/coreseek/bin/indexer -c etc/csft.conf xml

##以下为正常索引指定数据时的提示信息：


> Coreseek Fulltext 3.2 [ Sphinx 0.9.9-release (r2117)]
>
>
> Copyright (c) 2007-2010,
>
>
> Beijing Choice Software Technologies Inc (http://www.coreseek.com)
>
>
> using config file ‘etc/csft.conf’…
>
>
> indexing index ‘xml’…
>
>
> collected 3 docs, 0.0 MB
>
>
> sorted 0.0 Mhits, 100.0% done
>
>
> total 3 docs, 7585 bytes
>
>
> total 0.069 sec, 109614 bytes/sec, 43.35 docs/sec
>
>
> total 2 reads, 0.000 sec, 5.6 kb/call avg, 0.0 msec/call avg
>
>
> total 7 writes, 0.000 sec, 3.9 kb/call avg, 0.0 msec/call avg

> # /usr/local/coreseek/bin/search -c etc/csft.conf

##以下为正常测试搜索时的提示信息：


> Coreseek Fulltext 3.2 [ Sphinx 0.9.9-release (r2117)]
>
>
> Copyright (c) 2007-2010,
>
>
> Beijing Choice Software Technologies Inc (http://www.coreseek.com)
>
>
> using config file ‘etc/csft.conf’…
>
>
> index ‘xml’: query ”: returned 3 matches of 3 total in 0.093 sec
>
>
> displaying matches:
>
>
> 1. document=1, weight=1, published=Thu Apr  1 22:20:07 2010, author_id=1
>
>
> 2. document=2, weight=1, published=Thu Apr  1 23:25:48 2010, author_id=1
>
>
> 3. document=3, weight=1, published=Thu Apr  1 12:01:00 2010, author_id=2
>
>
> words:

> # /usr/local/coreseek/bin/search -c etc/csft.conf -a Twittter和Opera都提供了搜索服务

##以下为正常测试搜索关键词时的提示信息：


> Coreseek Fulltext 3.2 [ Sphinx 0.9.9-release (r2117)]
>
>
> Copyright (c) 2007-2010,
>
>
> Beijing Choice Software Technologies Inc (http://www.coreseek.com)
>
>
> using config file ‘etc/csft.conf’…
>
>
> index ‘xml’: query ‘Twittter和Opera都提供了搜索服务 ‘: returned 3 matches of 3 total in 0.038 sec
>
>
> displaying matches:
>
>
> 1. document=3, weight=24, published=Thu Apr  1 12:01:00 2010, author_id=2
>
>
> 2. document=1, weight=4, published=Thu Apr  1 22:20:07 2010, author_id=1
>
>
> 3. document=2, weight=3, published=Thu Apr  1 23:25:48 2010, author_id=1
>
>
> words:
>
>
> 1. ‘twittter’: 1 documents, 3 hits
>
>
> 2. ‘和’: 3 documents, 15 hits
>
>
> 3. ‘opera’: 1 documents, 25 hits
>
>
> 4. ‘都’: 2 documents, 4 hits
>
>
> 5. ‘提供’: 0 documents, 0 hits
>
>
> 6. ‘了’: 3 documents, 18 hits
>
>
> 7. ‘搜索’: 2 documents, 5 hits
>
>
> 8. ‘服务’: 1 documents, 1 hits

安装为系统服务

> # **/usr/local/coreseek/bin/searchd -c etc/csft.conf**

#以下为正常开启搜索服务时的提示信息：


> Coreseek Fulltext 3.2 [ Sphinx 0.9.9-release (r2117)]
>
>
> Copyright (c) 2007-2010,
>
>
> Beijing Choice Software Technologies Inc (http://www.coreseek.com)
>
>
> using config file ‘etc/csft.conf’…
>
>
> listening on all interfaces, port=9312

#如要停止搜索服务，请使用


> **/usr/local/coreseek/bin/searchd -c etc/csft.conf –stop**

#然后，请参考csft-3.2.13下api目录中的相关文件，使用PHP、Python、Ruby、Java来测试搜索服务；也可以前往 [搜索服务建立三步曲，查看第三步使用PHP测试](http://www.coreseek.com//products-install/step_by_step/)。


**四、继续**

##通过以上步骤，coreseek已经安装测试完成，可以提供正常的xml数据源索引以及提供对应的搜索服务了


##下一步工作，请查看手册，准备好mysql数据信息，以及进行mysql数据源的测试，并在您的应用中调用搜索服务；mysql数据源的配置可参考testpack/etc/csft_mysql.conf文件


中文分词核心配置


Coreseek/Sphinx(0.9)中文手册


##如有问题，请前往：http://www.coreseek.cn/forum/ 提问，我们将在第一时间解答


##如果您需要获得专业的支持，请前往：http://www.coreseek.cn/contact/ 与我们取得联系，洽谈支持与合作事宜.


注意:上面代码参数有部分为两个-字母,由于wordpress系统的问题,所以前台显示的为一个”-“字母.请自行添加,不要直接复制这里的代码,请参考官方为准 [http://www.coreseek.cn/products/products-install/install_on_bsd_linux/](http://www.coreseek.cn/products/products-install/install_on_bsd_linux/).

转摘请注明出处: [http://blog.haohtml.com/index.php/archives/5339](http://blog.haohtml.com/index.php/archives/5339)