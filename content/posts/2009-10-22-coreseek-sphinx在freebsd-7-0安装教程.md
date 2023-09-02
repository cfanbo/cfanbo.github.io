---
title: '[教程]coreseek sphinx在FreeBSD 7.0安装教程'
author: admin
type: post
date: 2009-10-22T09:04:14+00:00
excerpt: |
 感谢为中文全文检索做出贡献的所有同学。

 1、源码安装LibMMSeg 。 先在这里下载压缩包 http://www.coreseek.com/opensource/mmseg/
 # tar zxvf mmseg-0.7.3.tar.gz
 # cd mmseg-0.7.3
 # vim src/css/SegmentPkg.cpp 修改第27行， 将 #include  改为 #include
 # ./configure && make && make install
url: /archives/2540
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - coreseek
 - Sphinx

---
感谢为中文全文检索做出贡献的所有同学。

1、源码安装LibMMSeg 。 先在这里下载压缩包
**\# fetch  [http://www.coreseek.com/opensource/mmseg/](http://www.coreseek.com/opensource/mmseg/)**
**\# tar zxvf mmseg-0.7.3.tar.gz
\# cd mmseg-0.7.3**
**\# vim src/css/SegmentPkg.cpp** 修改第27行， 将 #include  改为 **#include
\# ./configure && make && make install**

2、测试 mmseg
**\# cd mmseg-0.7.3/data** 你会看到一个准备好的UTF-8编码的字典文件 unigram.txt
**\# mmseg -u unigram.txt** 该命令执行后，将会产生一个名为unigram.txt.uni的文件，将该文件改名为uni.lib，完成词典的构造。 你也可以进行分词测试。详见 [http://www.coreseek.com/opensource/mmseg/](http://www.coreseek.com/opensource/mmseg/)

3、ports安装 gawk
**\# cd /usr/ports/lang/gawk
\# make install clean**

4、ports安装 python25
**#cd /usr/ports/lang/python25
#make install clean**

5、设置环境变量
**\# cd –
\# vim .profile**

增加两行
**export CPPFLAGS=-I/usr/local/include/python2.5
export LDFLAGS=-lpython2.5**

6、源码安装 Coreseek Fulltext Search Server 2.5.2。 先在这里下载压缩包 [#fetch http://www.coreseek.com/uploads/sources/coreseek_fulltext_2.5.2.tar.gz](http://www.coreseek.com/uploads/sources/coreseek_fulltext_2.5.2.tar.gz)
**\# tar zxvf coreseek\_fulltext\_2.5.2.tar.gz
\# cd coreseek\_fulltext\_2.5.source
\# ./configure && make && make install**

7、配置服务
安装程序会默认创建 /usr/local/var/data 和 /usr/local/var/log 这里就是为了让你存放索引文件 和 运行日志的地方。
我们也把字典文件放在/usr/local/var下 。
**#cp -r dict /usr/local/var/dict** #这里把第一步生成的词典data目录内容复制到dict目录里

安装程序会在 /usr/local/etc/下放一个默认配置文件 Sphinx.conf.dist 。但是 Coreseek 默认要去读 csft.conf，所以要复制一份
**\# cp /usr/local/etc/sphinx.conf.dist /usr/local/etc/csft.conf**

修改 csft.conf 讲索引文件位置 和 日志文件位置 分别指向/usr/local/var/data 和 /usr/local/var/log
如需中文索引的配置节中加入
**charset\_type = zh\_cn.utf-8
charset_dictpath = /usr/local/var/dict**
其他配置项请参考手册.

8、导入测试数据
**\# mysql < source  /usr/local/etc/example.sql**

9、测试建立索引
**#indexer –all**

应该会看到如下输出
Coreseek Full Text Server 2.1
Copyright (c) 2006-2008 coreseek.com
using config file ‘/usr/local/etc/csft.conf’…
indexing index ‘test1’…
collected 5 docs, 0.0 MB
sorted 0.0 Mhits, 100.0% done
total 5 docs, 230 bytes
total 0.146 sec, 1577.50 bytes/sec, 34.29 docs/sec
indexing index ‘test1stemmed’…
collected 5 docs, 0.0 MB
sorted 0.0 Mhits, 100.0% done
total 5 docs, 230 bytes
total 0.011 sec, 21879.74 bytes/sec, 475.65 docs/sec

10、测试全文检索
**\# search doc**

应该会看到如下输出
search doc
Coreseek Full Text Server 2.1
Copyright (c) 2006-2008 coreseek.com
using config file ‘/usr/local/etc/csft.conf’…
index ‘test1’: query ‘doc ‘: returned 2 matches of 2 total in 0.091 sec

displaying matches:
1. document=3, weight=1, group\_id=2, date\_added=Fri Jul 4 10:22:13 2008
id=3
group_id=2
date_added=2008-07-04 10:22:13
title=another doc
content=this is another group
2. document=4, weight=1, group\_id=2, date\_added=Fri Jul 4 10:22:13 2008
id=4
group_id=2
date_added=2008-07-04 10:22:13
title=doc number four
content=this is to test groups

words:
1. ‘doc’: 2 documents, 2 hits

index ‘test1stemmed’: query ‘doc ‘: returned 2 matches of 2 total in 0.000 sec

displaying matches:
1. document=3, weight=1, group\_id=2, date\_added=Fri Jul 4 10:22:13 2008
id=3
group_id=2
date_added=2008-07-04 10:22:13
title=another doc
content=this is another group
2. document=4, weight=1, group\_id=2, date\_added=Fri Jul 4 10:22:13 2008
id=4
group_id=2
date_added=2008-07-04 10:22:13
title=doc number four
content=this is to test groups

words:
1. ‘doc’: 2 documents, 2 hits

11 、启动searchd
**\# searchd**

12 、测试searchd
**\# cd api
\# php test.php doc**

应该会输出
Query ‘doc ‘ retrieved 2 of 2 matches in 0.073 sec.
Query stats:
‘doc’ found 4 times in 4 documents

Matches:
1. doc\_id=3, weight=100, group\_id=2, date_added=2008-07-04 10:22:13
2. doc\_id=4, weight=100, group\_id=2, date_added=2008-07-04 10:22:13

12、关于中文全文检索
有些同学安装成功后 不能检索中文， 请检查你数据库的 variables 是否为utf8 ，请仔细反复检查一下。
可以参考这个帖子 [http://www.coreseek.com/forum/index.php?action=vthread&forum=2&topic=11](http://www.coreseek.com/forum/index.php?action=vthread&forum=2&topic=11)

注意:以上操作可能部分设置需要在csft.conf文件里做一些修改,如数据库用户名和密码,还有索引文件存放的目录,及词典存放位置等信息,请根据提示信息修改即可．