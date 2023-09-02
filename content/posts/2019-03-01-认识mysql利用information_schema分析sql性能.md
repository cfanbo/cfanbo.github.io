---
title: MySQL利用 INFORMATION_SCHEMA.PROFILING 分析SQL性能
author: admin
type: post
date: 2019-03-01T07:52:27+00:00
url: /archives/18837
categories:
 - MySQL
tags:
 - mysql
 - profile

---
MySQL5.7中有一个系统默认库 information_schema , 里面有些表如 PROFILING,、**OPTIMIZER_TRACE**、 PROCESSLIST、INNODB_TRX等，其中 PROFILE 对于我们分析sql有很大的帮助，在此以前我们需要使用 [SHOW PROFILE](https://dev.mysql.com/doc/refman/5.7/en/show-profile.html) 命令,不过此命令以后将被废弃。下面我们就介绍一下如何使用此表。

> 从 MySQL8.0开始， 这个表也开始被废弃了，以后分析性能问题直接使用另一个系统库 performance_schema 里的相关表（setup_actors）就可以了。到时候 show profiles 和show profile两个命令也不能用了。

**1.在使用此表前，我们需要开户性能检测功能。**

```
mysql> SELECT @@profiling;
+-------------+
| @@profiling |
+-------------+
|           0 |
+-------------+
1 row in set (0.00 sec)

mysql> SET profiling = 1;
Query OK, 0 rows affected (0.00 sec)
```

默认情况下是 OFF/0 状态。在我们分析完，最好关闭以减少服务器压力。

相关查询命令

```
show VARIABLES like 'profil%'
-------------------------------
profiling	ON
profiling_history_size	15
```

**2. 了解 information_schema.profiling 表的常用字段**

官方文档： [https://dev.mysql.com/doc/refman/8.0/en/profiling-table.html](https://dev.mysql.com/doc/refman/8.0/en/profiling-table.html)
 1

 QUERY_ID

 查询ID, 用于标记不同的查询

 2

 SEQ

 一个查询内部执行的步骤 , 从2开始

 3

 STATE

 步骤的状态

 4

 DURATION

 持续时间

 5

 CPU_USER

 用户空间的cpu 使用量

 6

 CPU_SYSTEM

 内核空间的cpu 使用量

 7

 CONTEXT_VOLUNTARY

 上下文主动切换

 8

 CONTEXT_INVOLUNTARY

 上下文被动切换

 9

 BLOCK_OPS_IN

 阻塞输入操作

 10

 BLOCK_OPS_OUT

 阻塞输出操作

 11

 MESSAGES_SENT

 消息发送

 12

 MESSAGES_RECEIVED

 消息接受

 13

 PAGE_FAULTS_MAJOR

 主分页错误

 14

 PAGE_FAULTS_MINOR

 次分页错误

 15

 SWAPS

 swap 发生的次数

 16

 SOURCE_FUNCTION

 MySQL源码执行函数

 17

 SOURCE_FILE

 源码文件

 18

 SOURCE_LINE

 源码行数


以下是我执行了一个join语句的输出，从结果中我们可以分析出哪个步骤执行的时间最长，进行相应的优化即可。![](https://blog.haohtml.com/wp-content/uploads/2019/03/mysql_profile-1024x364.png)

我们可以根据DURATION 列的值来分析哪一个模块消耗的时间多来进行相应的优化。

**推荐使用 OPTIMIZER_TRACER 来分析** SQL 执行过程
 ，每种参数用法可参考：