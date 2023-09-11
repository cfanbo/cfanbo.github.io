---
title: MYSQL慢速(SLOW LOG)脚本分析
author: admin
type: post
date: 2009-06-23T01:41:38+00:00
excerpt: |
 mysql有一个功能就是可以log下来运行的比较慢的sql语句，默认是没有这个log的，为了开启这个功能，
 要修改my.cnf或者在mysql启动的时候加入一些参数。如果在my.cnf里面修改，需增加如下几行

 long_query_time = 1
 log-slow-queries = /var/youpath/slow.log
 log-queries-not-using-indexes
url: /archives/1896
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - msyql

---
mysql有一个功能就是可以log下来运行的比较慢的sql语句，默认是没有这个log的，为了开启这个功能，
要修改my.cnf或者在mysql启动的时候加入一些参数。如果在my.cnf(Windows为my.ini文件)里面修改，需增加如下几行

`long_query_time = 1

log-slow-queries = /var/youpath/slow.log

log-queries-not-using-indexes`

long\_query\_time 是指执行超过多久的sql会被log下来，这里是1秒。
log-slow-queries 设置把日志写在那里，可以为空，系统会给一个缺省的文件host_name-slow.log，
log-queries-not-using-indexes 就是字面意思，log下来没有使用索引的query。

mysql有以下几种日志：
**错误日志：   -log-err
查询日志：   -log
慢查询日志:     -log-slow-queries
更新日志:     -log-update
二进制日志：   -log-bin**

把上述参数打开，运行一段时间，就可以关掉了,详细请见 [http://blog.haohtml.com/index.php/archives/2772](/index.php/archives/2772)。

接下来就是分析了，这里的文件名字叫host-slow.log。
先mysqldumpslow –help以下，e它主要用的是
`-s ORDER what to sort by (t, at, l, al, r, ar etc), ‘at’ is default

-t NUM just show the top n queries

-g PATTERN grep: only consider stmts that include this string`

-s，是order的顺序，说明写的不够详细，主要有
c,t,l,r和ac,at,al,ar，分别是按照query次数，时间，lock的时间和返回的记录数来排序，前面加了a的时倒叙
-t，是top n的意思，即为返回前面多少条的数据
-g，后边可以写一个正则匹配模式，大小写不敏感的

`mysqldumpslow -s c -t 20 host-slow.log

mysqldumpslow -s r -t 20 host-slow.log`

上述命令可以看出访问次数最多的20个sql语句和返回记录集最多的20个sql。
`mysqldumpslow -t 10 -s t -g “left join” host-slow.log`
这个是按照时间返回前10条里面含有左连接的sql语句。

用了这个工具就可以查询出来那些sql语句是性能的瓶颈，进行优化，比如加索引，该应用的实现方式等。

`mysqldumpslow -s c -t 10 /tmp/slow_query.log`