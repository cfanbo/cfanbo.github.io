---
title: '百度、新浪、Mixi、Apache社区赞助的开源 key-value分布式存储系统[原创]'
author: admin
type: post
date: 2010-04-01T14:22:14+00:00
url: /archives/3170
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx

---
[文章作者：张宴 本文版本：v1.0 最后修改：2009.01.21 转载请注明原文链接： [http://blog.s135.com/post/394/](http://blog.s135.com/post/394/)]

key-value分布式存储系统查询速度快、存放数据量大、支持高并发，非常适合通过主键进行查询，但不能进行复杂的条件查询。如果辅以Real- Time Search Engine（实时搜索引擎）进行复杂条件检索、全文检索，就可以替代并发性能较低的MySQL等关系型数据库，达到高并发、高性能，节省几十倍服务器数 量的目的。以MemcacheDB、Tokyo Tyrant为代表的key-value分布式存储，在上万并发连接下，轻松地完成高速查询。而MySQL，在几百个并发连接下，就基本上崩溃了。

虽然key-value分布式存储具有极高的性能，但是只能做类似于MySQL的SELECT * FROM table WHERE id = 123;简单主键查询。

“搜索索引引擎＋key-value分布式存储”能够实现高并发的复杂条件查询、全文检索与数据显示。但是， 由于索引更新需要时间，目前还不能实现完全意义上的Real-Time Search（实时搜索），只能称之为Near Real-Time Search（准实时搜索）。“搜索索引引擎＋key-value分布式存储”除了做全文检索外，还可以在允许的索引延迟范围内，取代MySQL进行复杂 条件查询。

我的文章《 [亿级数据的高并发通用搜索引擎架构设计](http://blog.s135.com/post/385.htm)》的程序编码已经完成，第一轮测试昨天已经结束，能够在高并发情况下实现 1分钟内索引更新，属于“Near Real-Time Search Engine（准实时搜索引擎）＋key-value分布式存储”应用。其中，索引引擎采用Sphinx，存储采用key-value分布式数据库 [Tokyo Tyrant](http://tokyocabinet.sourceforge.net/index.html)。

以下是常见的key-value分布式存储系统：

其中，以下几款值得关注：

1、 [Hypertable](http://hypertable.org/)：它是搜索引擎公司Zvents根据Google的9位研究人员在2006年发表的一篇论 文《 [Bigtable： 结构化数据的分布存储系统](http://labs.google.com/papers/bigtable.html)》开发的一款开源分布式数据储存系统。Hypertable是按照1000节点比例设计，以 C++撰写，可架在 HDFS 和 KFS 上。尽管还在初期阶段，但已有不错的效能：写入 28M 列的资料，各节点写入速率可达7MB/s，读取速率可达 1M cells/s。Hypertable目前一直没有太多高负载和大存储的应用实例，但是最近，Hypertable项目得到了 [百度](http://www.baidu.com/) 的赞助支持，相信其会有更好的发展。

[][1][][2][![hypertable_baidu](http://blog.haohtml.com/wp-content/uploads/2010/04/hypertable_baidu.gif)][2]

* * *

[
][2] 2、 [Tokyo Tyrant](http://tokyocabinet.sourceforge.net/index.html)：它是日本最大的SNS社交网站 [mixi.jp](http://mixi.jp/) 开发的 Tokyo Cabinet key-value数据库网络接口。它拥有Memcached兼容协议，也可以通过HTTP协议进行数据交换。对任何原有Memcached客户端来讲， 可以将Tokyo Tyrant看成是一个Memcached，但是，它的数据是可以持久存储的。Tokyo Tyrant 具有故障转移、日志文件体积小、大数据量下表现出色等优势，详见： [http://blog.s135.com/post/362.htm](http://blog.s135.com/post/362.htm)

Tokyo Cabinet 2009年1月18日发布的新版本（Version 1.4.0）已经实现 Table Database，将key-value数据库又扩展了一步，有了MySQL等关系型数据库的表和字段的概念，相信不久的将来，Tokyo Tyrant 也将支持这一功能。值得期待。

[][2][![tabledatabasecmp](http://blog.haohtml.com/wp-content/uploads/2010/04/tabledatabasecmp.png)][3]

* * *

3、 [CouchDB](http://couchdb.apache.org/)：它是Apache社区 基于 Erlang/OTP 构建的高性能、分布式容错非关系型数据库系统（NRDBMS）。它充分利用 Erlang 本身所提供的高并发、分布式容错基础平台，并且参考 Lotus Notes 数据库实现，采用简单的文档数据类型（document-oriented）。在其内部，文档数据均以 JSON 格式存储。对外，则通过基于 HTTP 的 REST 协议实现接口，可以用十几种语言进行自由操作。
[![sketch](http://blog.haohtml.com/wp-content/uploads/2010/04/sketch.png)][4]

* * *

4、 [MemcacheDB](http://memcachedb.org/)：它是新浪互 动社区事业部为在Memcached基础上，增加Berkeley DB存储层而开发一款支持高并发的分布式持久存储系统，对任何原有Memcached客户端来讲，它仍旧是个Memcached，但是，它的数据是可以持 久存储的。

[![memcachedb](http://blog.haohtml.com/wp-content/uploads/2010/04/memcachedb.jpg)][5]

 [1]: ../wp-content/uploads/2010/04/hypertable_baidu.gif
 [2]: http://blog.haohtml.com/wp-content/uploads/2010/04/hypertable_baidu.gif
 [3]: http://blog.haohtml.com/wp-content/uploads/2010/04/tabledatabasecmp.png
 [4]: http://blog.haohtml.com/wp-content/uploads/2010/04/sketch.png
 [5]: http://blog.haohtml.com/wp-content/uploads/2010/04/memcachedb.jpg