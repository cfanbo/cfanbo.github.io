---
title: mysql explain 中key_len的计算
author: admin
type: post
date: 2012-10-08T02:20:39+00:00
url: /archives/13433
categories:
 - MySQL
tags:
 - EXPLAIN

---
今天丁原问我mysql执行计划中的key_len是怎么计算得到的，当时还没有注意，在高性能的那本书讲到过这个值的计算，但是自己看执行计划的时候一直都没有太在意这个值，更不用说深讨这个值的计算了：

ken_len表示索引使用的字节数，根据这个值，就可以判断索引使用情况，特别是在组合索引的时候，判断所有的索引字段都被查询用到。

在查看官方文档的时候，也没有发现详细的key_len的计算介绍，后来做了一些测试，在咨询了丁奇关于变长数据类型的值计算的时候，突然想到innodb 行的格式，在这里的计算中有点类似，总结一下需要考虑到以下一些情况：(1).索引字段的附加信息：可以分为**变长**和**定长**数据类型讨论
当索引字段为定长数据类型，比如char，int，datetime，需要有是否为空的标记，这个标记需要占用**1**个字节；
对于变长数据类型，比如：varchar，除了是否为空的标记外，还需要有长度信息，需要占用**2**个字节；

(备注：当字段定义为非空的时候，是否为空的标记将不占用字节)

(2).同时还需要考虑表所使用的字符集，不同的字符集，gbk编码的为一个字符2个字节，utf8编码的一个字符3个字节;

**先看定长数据类型的一个例子(编码为gbk)：**

root@test 07:32:39>create table test\_char(id int not null ,name\_1 char(20),name_2 char(20),

-> primary key(id),key ind\_name(name\_1,name_2))engine=innodb charset=gbk;

root@test 07:33:55>insert into test_char values(1,’xuancan’,’taobaodba’);

root@test 07:34:55>explain select * from test\_char where name\_1=’xuancan’\G;

\***\***\***\***\***\***\***\***\*\\*\* 1. row \*\*\***\***\***\***\***\***\***\****

id: 1

select_type: SIMPLE

table: test_char

type: ref

possible\_keys: ind\_name

key: ind_name

key_len: 41

ref: const

rows: 1

Extra: Using where; Using index

key_len=41=20*2+1（备注：由于name_1为空，isnull的标记被打上，需要计算1个字节）

root@test 07:35:31>explain select * from test\_char where name\_1=’xuancan’ and name_2=’taobaodba’\G;

\***\***\***\***\***\***\***\***\*\\*\* 1. row \*\*\***\***\***\***\***\***\***\****

id: 1

select_type: SIMPLE

table: test_char

type: ref

possible\_keys: ind\_name

key: ind_name

key_len: 82

ref: const,const

rows: 1

Extra: Using where; Using index

key_len=82=20*2+20*2+1+1（备注：由于name\_1,name\_2两列被使用到，但两列都为为空，需要计算2个字节）

**变长数据类型（gbk编码）：**

root@test 08:30:51>create table test\_varchar(id int not null ,name\_1 varchar(20),name_2 varchar(20),

-> primary key(id),key ind\_name(name\_1,name_2))engine=innodb charset=gbk;

root@test 08:37:51>insert into test_varchar values(1,’xuancan’,’taobaodba’);

root@test 08:38:14>explain select * from test\_varchar where name\_1=’xuancan’\G;

\***\***\***\***\***\***\***\***\*\\*\* 1. row \*\*\***\***\***\***\***\***\***\****

id: 1

select_type: SIMPLE

table: test_varchar

type: ref

possible\_keys: ind\_name

key: ind_name

key_len: 43

ref: const

rows: 1

Extra: Using where; Using index

key_len=43=20*2+1+2（备注：由于为name_1字段定义为空，所以需要计算1,；同时由于是变长字段varchar，所以需要加上2）

root@test 08:38:46>alter table test\_varchar modify column name\_1 varchar(20) not null;

Query OK, 1 row affected (0.52 sec)

Records: 1 Duplicates: 0 Warnings: 0

root@test 08:42:11>explain select * from test\_varchar where name\_1=’xuancan’\G;

\***\***\***\***\***\***\***\***\*\\*\* 1. row \*\*\***\***\***\***\***\***\***\****

id: 1

select_type: SIMPLE

table: test_varchar

type: ref

possible\_keys: ind\_name

key: ind_name

key_len: 42

ref: const

rows: 1

Extra: Using where; Using index

key_len=42=20*2+2(备注由于name_1字段修改为not null之后，isnull的标记锁占用的字节释放掉，但是变长字段长度所占用的2个字节没有释放)；

上面是测试gbk编码的测试，同时也可以测试一下其他编码的key_len计算。

摘自：