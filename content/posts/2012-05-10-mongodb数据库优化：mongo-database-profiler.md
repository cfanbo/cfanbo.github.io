---
title: MongoDB数据库优化：Mongo Database Profiler
author: admin
type: post
date: 2012-05-10T09:39:02+00:00
url: /archives/12898
IM_contentdowned:
 - 1
categories:
 - nosql
tags:
 - MongoDB

---
在MySQL中，慢查询日志是经常作为我们优化数据库的依据，那在MongoDB中是否有类似的功能呢?答案是肯定的，那就是Mongo Database Profiler.不仅有，而且还有一些比MySQL的Slow Query Log更详细的信息。它就是我们这篇文章的主题。

**　　开启 Profiling 功能**

有两种方式可以控制 Profiling 的开关和级别，第一种是直接在启动参数里直接进行设置。

启动MongoDB时加上–profile=级别 即可。

也可以在客户端调用db.setProfilingLevel(级别) 命令来实时配置。可以通过db.getProfilingLevel()命令来获取当前的Profile级别。
 　　> db.setProfilingLevel(2);

 {“was” : 0 , “ok” : 1}

 > db.getProfilingLevel()
 　上面斜体的级别可以取0，1，2 三个值，他们表示的意义如下：

0 – 不开启

1 – 记录慢命令 (默认为>100ms)

2 – 记录所有命令

Profile 记录在级别1时会记录慢命令，那么这个慢的定义是什么?上面我们说到其默认为100ms，当然有默认就有设置，其设置方法和级别一样有两种，一种是通过添加–slowms启动参数配置。第二种是调用db.setProfilingLevel时加上第二个参数：



 　　db.setProfilingLevel( level , slowms )

 db.setProfilingLevel( 1 , 10 );




**　　查询 Profiling 记录**

与MySQL的慢查询日志不同，Mongo Profile 记录是直接存在系统db里的，记录位置 system.profile ，所以，我们只要查询这个Collection的记录就可以获取到我们的 Profile 记录了。



 　　> db.system.profile.find()

 {“ts” : “Thu Jan 29 2009 15:19:32 GMT-0500 (EST)” , “info” : “query test.$cmd ntoreturn:1 reslen:66 nscanned:0

 query: { profile: 2 } nreturned:1 bytes:50” , “millis” : 0}

 db.system.profile.find( { info: /test.foo/ } )

 {“ts” : “Thu Jan 29 2009 15:19:40 GMT-0500 (EST)” , “info” : “insert test.foo” , “millis” : 0}

 {“ts” : “Thu Jan 29 2009 15:19:42 GMT-0500 (EST)” , “info” : “insert test.foo” , “millis” : 0}

 {“ts” : “Thu Jan 29 2009 15:19:45 GMT-0500 (EST)” , “info” : “query test.foo ntoreturn:0 reslen:102 nscanned:2

 query: {} nreturned:2 bytes:86” , “millis” : 0}

 {“ts” : “Thu Jan 29 2009 15:21:17 GMT-0500 (EST)” , “info” : “query test.foo ntoreturn:0 reslen:36 nscanned:2

 query: { $not: { x: 2 } } nreturned:0 bytes:20” , “millis” : 0}

 {“ts” : “Thu Jan 29 2009 15:21:27 GMT-0500 (EST)” , “info” : “query test.foo ntoreturn:0 exception bytes:53” , “millis” : 88}




列出执行时间长于某一限度(5ms)的 Profile 记录：



 　　> db.system.profile.find( { millis : { $gt : 5 } } )

 {“ts” : “Thu Jan 29 2009 15:21:27 GMT-0500 (EST)” , “info” : “query test.foo ntoreturn:0 exception bytes:53” , “millis” : 88}




查看最新的 Profile 记录：

db.system.profile.find().sort({$natural:-1})

Mongo Shell 还提供了一个比较简洁的命令show profile，可列出最近5条执行时间超过1ms的 Profile 记录。

**　　Profile 信息内容详解：**

ts-该命令在何时执行.

millis Time-该命令执行耗时，以毫秒记.

info-本命令的详细信息.

query-表明这是一个query查询操作.

ntoreturn-本次查询客户端要求返回的记录数.比如, findOne()命令执行时 ntoreturn 为 1.有limit(n) 条件时ntoreturn为n.

query-具体的查询条件(如x>3).

nscanned-本次查询扫描的记录数.

reslen-返回结果集的大小.

nreturned-本次查询实际返回的结果集.

update-表明这是一个update更新操作.

fastmod-Indicates a fast modify operation. See Updates. These operations are normally quite fast.

fastmodinsert – indicates a fast modify operation that performed an upsert.

upsert-表明update的upsert参数为true.此参数的功能是如果update的记录不存在，则用update的条件insert一条记录.

moved-表明本次update是否移动了硬盘上的数据，如果新记录比原记录短，通常不会移动当前记录，如果新记录比原记录长，那么可能会移动记录到其它位置，这时候会导致相关索引的更新.磁盘操作更多，加上索引更新，会使得这样的操作比较慢.

insert-这是一个insert插入操作.

getmore-这是一个getmore 操作，getmore通常发生在结果集比较大的查询时，第一个query返回了部分结果，后续的结果是通过getmore来获取的。

**　　MongoDB 查询优化**

如果nscanned(扫描的记录数)远大于nreturned(返回结果的记录数)的话，那么我们就要考虑通过加索引来优化记录定位了。

reslen 如果过大，那么说明我们返回的结果集太大了，这时请查看find函数的第二个参数是否只写上了你需要的属性名。(类似 于MySQL中不要总是select *)

对于创建索引的建议是：如果很少读，那么尽量不要添加索引，因为索引越多，写操作会越慢。如果读量很大，那么创建索引还是比较划算的。(和RDBMS一样，貌似是废话 -_-!!)

**　　MongoDB 更新优化**

如果写查询量或者update量过大的话，多加索引是会有好处的。以及～～～～(省略N字，和RDBMS差不多的道理)

Use fast modify operations when possible (and usually with these, an index). See Updates.

**　　Profiler 的效率**

Profiling 功能肯定是会影响效率的，但是不太严重，原因是他使用的是system.profile 来记录，而system.profile 是一个capped collection 这种collection 在操作上有一些限制和特点，但是效率更高。