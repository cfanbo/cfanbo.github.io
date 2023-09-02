---
title: 'Sphinx在Windows下安装使用[支持中文全文检索]'
author: admin
type: post
date: 2009-10-14T01:38:50+00:00
excerpt: |
 前一阵子尝试使用了一下Sphinx，一个能够被各种语言(PHP/Python/Ruby/etc)方便调用的全文检索系统。网上的资料大多是在linux环境下的安装使用，当然，作为生产环境很有必要部署在*nix环境下，作为学习测试，还是windows环境比较方便些。

 本文旨在提供一种便捷的方式让Sphinx在windows下安装配置以支持中文全文检索，配置部分在linux下通用。

 一、关于Sphinx

 Sphinx 是一个在GPLv2 下发布的一个全文检索引擎，商业授权（例如, 嵌入到其他程序中）需要联系作者（Sphinxsearch.com）以获得商业授权。
url: /archives/2505
keywords:
 - sphinx,windows,sphinx的安装
description:
 - 中文全文检索Sphinx在Windows下的安装教程
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Sphinx

---
前一阵子尝试使用 了一下Sphinx，一个能够被各种语言(PHP/Python/Ruby/etc)方便调用的全文检索系统。网上的资料大多是在linux环境下的安装 使用，当然，作为生产环境很有必要部署在*nix环境下，作为学习测试，还是windows环境比较方便些。

本文旨在提供一种便捷的方式让Sphinx在windows下安装配置以支持中文全文检索，配置部分在linux下通用。

**一、关于Sphinx**

Sphinx 是一个在GPLv2 下发布的一个全文检索引擎，商业授权（例如, 嵌入到其他程序中）需要联系作者（Sphinxsearch.com）以获得商业授权。

一般而言，Sphinx是一个独立的搜索引擎，意图为其他应用提供高速、低空间占用、高结果相关度的全文搜索功能。Sphinx可以非常容易的与SQL数据库和脚本语言集成。

当前系统内置MySQL和PostgreSQL 数据库数据源的支持，也支持从标准输入读取特定格式的XML数据。通过修改源代码，用户可以自行增加新的数据源（例如：其他类型的DBMS的原生支持）。

搜索API支持PHP、Python、Perl、Rudy和Java，并且也可以用作MySQL存储引擎。搜索API非常简单，可以在若干个小时之内移植到新的语言上。

Sphinx特性：

 * 高速的建立索引(在当代CPU上，峰值性能可达到10MB/秒);
 * 高性能的搜索(在2–4GB的文本数据上，平均每次检索响应时间小于0.1秒);
 * 可处理海量数据(目前已知可以处理超过100GB的文本数据,在单一CPU的系统上可处理100M文档);
 * 提供了优秀的相关度算法，基于短语相似度和统计（BM25）的复合Ranking方法;
 * 支持分布式搜索;
 *

提供文件的摘录生成;


 *

可作为MySQL的存储引擎提供搜索服务;


 *

支持布尔、短语、词语相似度等多种检索模式;


 *

文档支持多个全文检索字段(最大不超过32个);


 *

文档支持多个额外的属性信息(例如：分组信息，时间戳等);


 *

停止词查询;


 *

支持单一字节编码和UTF-8编码;


 *

原生的MySQL支持(同时支持MyISAM和InnoDB);


 *

原生的PostgreSQL支持.


中文手册可以在 [这里](http://www.coreseek.com/uploads/pdf/sphinx_doc_zhcn_0.9.pdf) 获得（酷勤网备用下载地址： [sphinx_doc_zhcn_0.9.pdf](http://down1.kuqin.com/smallbook/sphinx_doc_zhcn_0.9.pdf)）。

**二、Sphinx在windows上的安装**

1.直接在 [http://www.sphinxsearch.com/downloads.html](http://www.sphinxsearch.com/downloads.html) 找到最新的windows版本，我这里下的是 [Win32 release binaries with MySQL support](http://www.sphinxsearch.com/downloads/sphinx-0.9.8.1-win32.zip)，下载后解压在D:\sphinx目录下；

2.在D:\sphinx\下新建一个data目录用来存放索引文件，一个log目录方日志文件，复制D:\sphinx\sphinx.conf.in到D:\sphinx\bin\sphinx.conf（注意修改文件名）；

3.修改D:\sphinx\bin\sphinx.conf，我这里列出需要修改的几个：

>

```
type        = mysql # 数据源，我这里是mysql
sql_host    = localhost # 数据库服务器
sql_user    = root # 数据库用户名
sql_pass    = '' # 数据库密码
sql_db      = test # 数据库
sql_port    = 3306 # 数据库端口
```

>
>

```
sql_query_pre   = SET NAMES utf8 # 去掉此行前面的注释，如果你的数据库是uft8编码的
```

>
>

```
index test1
{
# 放索引的目录
 path   = D:/sphinx/data/
# 编码
 charset_type  = utf-8
 #  指定utf-8的编码表
 charset_table  = 0..9, A..Z->a..z, _, a..z, U+410..U+42F->U+430..U+44F, U+430..U+44F
 # 简单分词，只支持0和1，如果要搜索中文，请指定为1
 ngram_len    = 1
# 需要分词的字符，如果要搜索中文，去掉前面的注释
 ngram_chars   = U+3000..U+2FA1F
}
```

>
>

```
# index test1stemmed : test1
# {
 # path   = @CONFDIR@/data/test1stemmed
 # morphology  = stem_en
# }

# 如果没有分布式索引，注释掉下面的内容

# index dist1
# {
 # 'distributed' index type MUST be specified
 # type    = distributed
```

>
>

```
 # local index to be searched
 # there can be many local indexes configured
 # local    = test1
 # local    = test1stemmed
```

>
>

```
 # remote agent
 # multiple remote agents may be specified
 # syntax is 'hostname:port:index1,[index2[,...]]
 # agent    = localhost:3313:remote1
 # agent    = localhost:3314:remote2,remote3
```

>
>

```
 # remote agent connection timeout, milliseconds
 # optional, default is 1000 ms, ie. 1 sec
 # agent_connect_timeout = 1000
```

>
>

```
 # remote agent query timeout, milliseconds
 # optional, default is 3000 ms, ie. 3 sec
 # agent_query_timeout  = 3000
# }
```

>
>

```
# 搜索服务需要修改的部分
searchd
{
 # 日志
 log     = D:/sphinx/log/searchd.log
```

>
>

```
 # PID file, searchd process ID file name
 pid_file   = D:/sphinx/log/searchd.pid
```

>
>

```
# windows下启动searchd服务一定要注释掉这个
 # seamless_rotate  = 1
}
```

4.导入测试数据

C:\Program Files\MySQL\MySQL Server 5.0\bin>mysql -uroot test D:\sphinx\bin>indexer.exe –all
> Sphinx 0.9.8-release (r1533)
> Copyright (c) 2001-2008, Andrew Aksyonoff
>
> using config file ‘./sphinx.conf’…
> indexing index ‘test1′…
> collected 4 docs, 0.0 MB
> sorted 0.0 Mhits, 100.0% done
> total 4 docs, 193 bytes
> total 0.101 sec, 1916.30 bytes/sec, 39.72 docs/sec
>
> D:\sphinx\bin>

6.搜索’test’试试

> D:\sphinx\bin>search.exe test
> Sphinx 0.9.8-release (r1533)
> Copyright (c) 2001-2008, Andrew Aksyonoff
>
> using config file ‘./sphinx.conf’…
> index ‘test1′: query ‘test ‘: returned 3 matches of 3 total in 0.000 sec
>
> displaying matches:
> 1. document=1, weight=2, group\_id=1, date\_added=Wed Nov 26 14:58:59 2008
> id=1
> group_id=1
> group_id2=5
> date_added=2008-11-26 14:58:59
> title=test one
> content=this is my test document number one. also checking search within
> phrases.
> 2. document=2, weight=2, group\_id=1, date\_added=Wed Nov 26 14:58:59 2008
> id=2
> group_id=1
> group_id2=6
> date_added=2008-11-26 14:58:59
> title=test two
> content=this is my test document number two
> 3. document=4, weight=1, group\_id=2, date\_added=Wed Nov 26 14:58:59 2008
> id=4
> group_id=2
> group_id2=8
> date_added=2008-11-26 14:58:59
> title=doc number four
> content=this is to test groups
>
> words:
> 1. ‘test’: 3 documents, 5 hits
> D:\sphinx\bin>

都所出来了吧。

6.测试中文搜索

修改test数据库中documents数据表，

> UPDATE \`test\`.\`documents\` SET \`title\` = ‘测试中文’, \`content\` = ‘this is my test document number two，应该搜的到吧’ WHERE \`documents\`.\`id\` = 2;

重建索引：

D:\sphinx\bin>indexer.exe –all

搜索’中文’试试：

> D:\sphinx\bin>search.exe 中文
> Sphinx 0.9.8-release (r1533)
> Copyright (c) 2001-2008, Andrew Aksyonoff
>
> using config file ‘./sphinx.conf’…
> index ‘test1′: query ‘中文 ‘: returned 0 matches of 0 total in 0.000 sec
>
> words:
> D:\sphinx\bin>

貌似没有搜到，这是因为windows命令行中的编码是gbk，当然搜不出来。我们可以用程序试试，在D:\sphinx\api下新建一个foo.php的文件，注意utf-8编码

>  require ’sphinxapi.php’;
> $s = new SphinxClient();
> $s->SetServer(’localhost’,3312);
> $result = $s->Query(’中文’);
> var_dump($result);
> ?>

启动Sphinx searchd服务

> D:\sphinx\bin>searchd.exe
> Sphinx 0.9.8-release (r1533)
> Copyright (c) 2001-2008, Andrew Aksyonoff
>
> WARNING: forcing –console mode on Windows
> using config file ‘./sphinx.conf’…
> creating server socket on 0.0.0.0:3312
> accepting connections

执行PHP查询：

> php d:/sphinx/api/foo.php

结果是不是出来？剩下的工作就是去看手册，慢慢摸索高阶的配置。