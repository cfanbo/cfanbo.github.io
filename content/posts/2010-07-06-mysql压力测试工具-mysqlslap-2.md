---
title: MySQL压力测试工具 mysqlslap 使用简介
author: admin
type: post
date: 2010-07-06T13:27:22+00:00
url: /archives/4418
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql
 - 压力测试

---

MySQL从5.1.4版开始带有一个压力测试工具 **mysqlslap**，通过模拟多个并发客户端访问 mysql来执行测试，使用起来非常的简单。通过mysqlslap –help可以获得可用的选项，这里列一些主要的参数，更详细的说明参考 [官方手册](http://dev.mysql.com/doc/refman/5.1/en/mysqlslap.html)。


> –auto-generate-sql, -a
>
> 自动生成测试表和数据
>
>
> –auto-generate-sql-load-type=type
>
> 测试语句的类型。取值包括：read，key，write，update和mixed(默认)。
>
>
> –number-char-cols=N, -x N
>
> 自动生成的测试表中包含多少个字符类型的列，默认1
>
>
> –number-int-cols=N, -y N
>
> 自动生成的测试表中包含多少个数字类型的列，默认1
>
> –number-of-queries=N
>
> 总的测试查询次数(并发客户数×每客户查询次数)
>
>
> –query=name,-q
>
> 使用自定义脚本执行测试，例如可以调用自定义的一个存储过程或者sql语句来执行测试。
>
>
> –create-schema
>
> 测试的schema，MySQL中schema也就是database
>
>
> –commint=N
>
> 多少条DML后提交一次
>
>
> –compress, -C
>
> 如果服务器和客户端支持都压缩，则压缩信息传递
>
>
> –concurrency=N, -c N
>
> 并发量，也就是模拟多少个客户端同时执行select。可指定多个值，以逗号或者–delimiter参数指定的值做为分隔符
>
>
> –engine=engine_name, -e engine_name
>
> 创建测试表所使用的存储引擎，可指定多个
>
>
> –iterations=N, -i N
>
> 测试执行的迭代次数
>
>
> –detach=N
>
> 执行N条语句后断开重连
>
>
> –debug-info, -T
>
> 打印内存和CPU的信息
>
>
> –only-print
>
> 只打印测试语句而不实际执行

测试的过程需要生成测试表，插入测试数据，这个mysqlslap可以自动生成，默认生成一个mysqlslap的schema，如果已经存在则先 删除，这里要注意了，不要用–create-schema指定已经存在的库，否则后果可能很严重。可以用–only-print来打印实际的测试过程：


$mysqlslap -a –only-print

DROP SCHEMA IF EXISTS `mysqlslap`;

CREATE SCHEMA `mysqlslap`;

use mysqlslap;

CREATE TABLE `t1` (intcol1 INT(32) ,charcol1 VARCHAR(128));

INSERT INTO t1 VALUES (1804289383,’mxvtvmC9127qJNm06sGB8R92q2j7vTiiITRDGXM9ZLzkdekbWtmXKwZ2qG1llkRw5m9DHOFilEREk3q7oce8O3BEJC0woJsm6uzFAEynLH2xCsw1KQ1lT4zg9rdxBL’);

…

SELECT intcol1,charcol1 FROM t1;

INSERT INTO t1 VALUES (364531492,’qMa5SuKo4M5OM7ldvisSc6WK9rsG9E8sSixocHdgfa5uiiNTGFxkDJ4EAwWC2e4NL1BpAgWiFRcp1zIH6F1BayPdmwphatwnmzdwgzWnQ6SRxmcvtd6JRYwEKdvuWr’);

DROP SCHEMA IF EXISTS `mysqlslap`;


可以看到最后由删除一开始创建的schema的动作，整个测试完成后不会在数据库中留下痕迹。假如我们执行一次测试，分别50和100个并发，执行 1000次总查询，那么：


$mysqlslap -a –concurrency=50,100 –number-of-queries 1000 –debug-info

Benchmark

Average number of seconds to run all queries: 0.375 seconds

Minimum number of seconds to run all queries: 0.375 seconds

Maximum number of seconds to run all queries: 0.375 seconds

Number of clients running queries: 50

Average number of queries per client: 20Benchmark

Average number of seconds to run all queries: 0.453 seconds

Minimum number of seconds to run all queries: 0.453 seconds

Maximum number of seconds to run all queries: 0.453 seconds

Number of clients running queries: 100

Average number of queries per client: 10

User time 0.29, System time 0.11

Maximum resident set size 0, Integral resident set size 0

Non-physical pagefaults 4032, Physical pagefaults 0, Swaps 0

Blocks in 0 out 0, Messages in 0 out 0, Signals 0

Voluntary context switches 7319, Involuntary context switches 681


上结果可以看出，50和100个并发分别得到一次测试结果(Benchmark)，并发数越多，执行完所有查询的时间越长。为了准确起见，可以多迭 代测试几次:


$ mysqlslap -a –concurrency=50,100 –number-of-queries 1000 –iterations=5 –debug-info

Benchmark

Average number of seconds to run all queries: 0.380 seconds

Minimum number of seconds to run all queries: 0.377 seconds

Maximum number of seconds to run all queries: 0.385 seconds

Number of clients running queries: 50

Average number of queries per client: 20Benchmark

Average number of seconds to run all queries: 0.447 seconds

Minimum number of seconds to run all queries: 0.444 seconds

Maximum number of seconds to run all queries: 0.451 seconds

Number of clients running queries: 100

Average number of queries per client: 10

User time 1.44, System time 0.67

Maximum resident set size 0, Integral resident set size 0

Non-physical pagefaults 17922, Physical pagefaults 0, Swaps 0

Blocks in 0 out 0, Messages in 0 out 0, Signals 0

Voluntary context switches 36796, Involuntary context switches 4093


测试同时不同的存储引擎的性能进行对比：


$ mysqlslap -a –concurrency=50,100 –number-of-queries 1000 –iterations=5 –engine=myisam,innodb –debug-info

Benchmark

Running for engine myisam

Average number of seconds to run all queries: 0.200 seconds

Minimum number of seconds to run all queries: 0.188 seconds

Maximum number of seconds to run all queries: 0.210 seconds

Number of clients running queries: 50

Average number of queries per client: 20Benchmark

Running for engine myisam

Average number of seconds to run all queries: 0.238 seconds

Minimum number of seconds to run all queries: 0.228 seconds

Maximum number of seconds to run all queries: 0.251 seconds

Number of clients running queries: 100

Average number of queries per client: 10

Benchmark

Running for engine innodb

Average number of seconds to run all queries: 0.375 seconds

Minimum number of seconds to run all queries: 0.370 seconds

Maximum number of seconds to run all queries: 0.379 seconds

Number of clients running queries: 50

Average number of queries per client: 20


Benchmark

Running for engine innodb

Average number of seconds to run all queries: 0.443 seconds

Minimum number of seconds to run all queries: 0.440 seconds

Maximum number of seconds to run all queries: 0.447 seconds

Number of clients running queries: 100

Average number of queries per client: 10


User time 2.83, System time 1.66

Maximum resident set size 0, Integral resident set size 0

Non-physical pagefaults 34692, Physical pagefaults 0, Swaps 0

Blocks in 0 out 0, Messages in 0 out 0, Signals 0

Voluntary context switches 87306, Involuntary context switches 10326


**本文网址：**[http://www.ningoo.net/html/2008/mysql\_load\_test\_tool\_mysqlslap.html][1]

 [1]: http://www.ningoo.net/html/2008/mysql_load_test_tool_mysqlslap.html