---
title: MySQL压力测试工具 mysqlslap
author: admin
type: post
date: 2010-07-06T13:24:03+00:00
url: /archives/4415
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql
 - 压力测试

---
**mysqlslap是一个mysql官方提供的压力测试工具。以下是比较重要的参数：**
–-defaults-file，配置文件存放位置
–-concurrency，并发数
–-engines，引擎
–-iterations，迭代的实验次数
–-socket，socket文件位置

**自动测试：**
-–auto-generate-sql，自动产生测试SQL
–-auto-generate-sql-load-type，测试SQL的类型。类型有mixed，update，write，key，read。
–-number-of-queries，执行的SQL总数量
–-number-int-cols，表内int列的数量
–-number-char-cols，表内char列的数量

例如：
shell>mysqlslap –-defaults-file=/u01/mysql1/mysql/my.cnf –-concurrency=50,100 –-iterations=1 –-number-int-cols=4 –-auto-generate-sql –auto-generate-sql-load-type=write –-engine=myisam -–number-of-queries=200   -S/tmp/mysql.sock
Benchmark
Running for engine myisam
Average number of seconds to run all queries: 0.016 seconds
Minimum number of seconds to run all queries: 0.016 seconds
Maximum number of seconds to run all queries: 0.016 seconds
Number of clients running queries: 50
Average number of queries per client: 4

Benchmark
Running for engine myisam
Average number of seconds to run all queries: 0.265 seconds
Minimum number of seconds to run all queries: 0.265 seconds
Maximum number of seconds to run all queries: 0.265 seconds
Number of clients running queries: 100
Average number of queries per client: 2

**指定数据库的测试：**
–create-schema，指定数据库名称
–query，指定SQL语句，可以定位到某个包含SQL的文件

例如：
shell>mysqlslap –defaults-file=/u01/mysql1/mysql/my.cnf –concurrency=25,50 –iterations=1 –create-schema=test –query=/u01/test.sql -S/tmp/mysql1.sock
Benchmark
Average number of seconds to run all queries: 0.018 seconds
Minimum number of seconds to run all queries: 0.018 seconds
Maximum number of seconds to run all queries: 0.018 seconds
Number of clients running queries: 25
Average number of queries per client: 1

Benchmark
Average number of seconds to run all queries: 0.011 seconds
Minimum number of seconds to run all queries: 0.011 seconds
Maximum number of seconds to run all queries: 0.011 seconds
Number of clients running queries: 50
Average number of queries per client: 1

如果提示参数有错误的话,再参数前面多加一个-符号,连续两个符号就可以了.

项目主页： [http://dev.mysql.com/doc/refman/5.1/en/mysqlslap.html](http://dev.mysql.com/doc/refman/5.1/en/mysqlslap.html)