---
title: MySQL技术内幕：InnoDB存储-3.6 InnoDB存储引擎文件
author: admin
type: post
date: 2011-09-29T08:29:33+00:00
url: /archives/11592
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - innodb
 - mysql

---

官方教程： [http://dev.mysql.com/doc/refman/5.1/zh/storage-engines.html#innodb](http://dev.mysql.com/doc/refman/5.1/zh/storage-engines.html#innodb)

3.6   InnoDB存储引擎文件

之前介绍的文件都是MySQL数据库本身的文件，和存储引擎无关。除了这些文件外，每个表存储引擎还有其自己独有的文件。这一节将具体介绍和InnoDB存储引擎密切相关的文件，这些文件包括重做日志文件、表空间文件。

**3.6.1   表空间文件**

InnoDB存储引擎在存储设计上模仿了Oracle，将存储的数据按表空间进行存放。默认配置下，会有一个初始化大小为10MB、名为ibdata1的文件。该文件就是默认的表空间文件（tablespace file）。你可以通过参数innodb_data_file_path对其进行设置。格式如下：

>

> innodb_data_file_path=datafile_spec1[;datafile_spec2]…
>

你也可以用多个文件组成一个表空间，同时制定文件的属性，如：

>

> [mysqld]

innodb_data_file_path = /db/ibdata1:2000M;/dr2/db/ibdata2:2000M:autoextend
>

这里将/db/ibdata1和/dr2/db/ibdata2两个文件用来组成表空间。若这两个文件位于不同的磁盘上，则可以对性能带来一定程度的提升。两个文件的文件名后都跟了属性，表示文件idbdata1的大小为2000MB，文件ibdata2的大小为2000MB，但是如果用满了这2000MB后，该文件可以自动增长（autoextend）。

设置innodb_data_file_path参数后，之后对于所有基于InnoDB存储引擎的表的数据都会记录到该文件内。而通过设置参数innodb_file_per_table，我们可以将每个基于InnoDB存储引擎的表单独产生一个表空间，文件名为表名.ibd，这样不用将所有数据都存放于默认的表空间中。下面这台服务器设置了innodb_file_per_table，可以看到：

>

> mysql> show variables like ‘innodb_file_per_table’G;

*************************** 1. row ***************************

Variable_name: innodb_file_per_table

Value: ON

1 row in set (0.00 sec)
>

>
>

> mysql> system ls -lh /usr/local/mysql/data/member/*

 -rw-r—–  1 mysql mysql 8.7K 2009-02-24  /usr/local/mysql/data/member/ Profile.frm

 -rw-r—–  1 mysql mysql 1.7G  9月 25 11:13 /usr/local/mysql/data/member/ Profile.ibd

 -rw-rw—-  1 mysql mysql 8.7K  9月 27 13:38 /usr/local/mysql/data/member/t1.frm

 -rw-rw—-  1 mysql mysql  17M  9月 27 13:40 /usr/local/mysql/data/member/t1.ibd

 -rw-rw—-  1 mysql mysql 8.7K  9月 27 15:42 /usr/local/mysql/data/member/t2.frm

 -rw-rw—-  1 mysql mysql  17M  9月 27 15:54 /usr/local/mysql/data/member/t2.ibd
>

表Profile、t1、t2都是InnoDB的存储引擎，由于设置参数innodb_file_per_table=ON，因此产生了单独的.ibd表空间文件。需要注意的是，这些单独的表空间文件仅存储该表的数据、索引和插入缓冲等信息，其余信息还是存放在默认的表空间中。图3-1显示了InnoDB存储引擎对于文件的存储方式：

图3-1   InnoDB表存储引擎文件

**3.6.2   重做日志文件**

默认情况下会有两个文件，名称分别为ib_logfile0和ib_logfile1。MySQL官方手册中将其称为InnoDB存储引擎的日志文件，不过更准确的定义应该是重做日志文件（redo log file）。为什么强调是重做日志文件呢？因为重做日志文件对于InnoDB存储引擎至关重要，它们记录了对于InnoDB存储引擎的事务日志。

重做日志文件的主要目的是，万一实例或者介质失败（media failure），重做日志文件就能派上用场。如数据库由于所在主机掉电导致实例失败，InnoDB存储引擎会使用重做日志恢复到掉电前的时刻，以此来保证数据的完整性。

每个InnoDB存储引擎至少有1个重做日志文件组（group），每个文件组下至少有2个重做日志文件，如默认的ib_logfile0、ib_logfile1。为了得到更高的可靠性，你可以设置多个镜像日志组（mirrored log groups），将不同的文件组放在不同的磁盘上。日志组中每个重做日志文件的大小一致，并以循环方式使用。InnoDB存储引擎先写重做日志文件1，当达到文件的最后时，会切换至重做日志文件2，当重做日志文件2也被写满时，会再切换到重做日志文件1中。图3-2显示了一个拥有3个重做日志文件的重做日志文件组。

图3-2   日志文件组

参数innodb_log_file_size、innodb_log_files_in_group、innodb_mirrored_log_groups、innodb_log_group_home_dir影响着重做日志文件的属性。参数innodb_log_file_size指定了重做日志文件的大小；innodb_log_files_in_group指定了日志文件组中重做日志文件的数量，默认为2；innodb_mirrored_log_groups指定了日志镜像文件组的数量，默认为1，代表只有一个日志文件组，没有镜像；innodb_log_group_home_dir指定了日志文件组所在路径，默认在数据库路径下。以下显示了一个关于重做日志组的配置：

>

> mysql> show variables like ‘innodb%log%’G;

*************************** 1. row ***************************

Variable_name: innodb_flush_log_at_trx_commit

Value: 1

*************************** 2. row ***************************

Variable_name: innodb_locks_unsafe_for_binlog

Value: OFF

*************************** 3. row ***************************

Variable_name: innodb_log_buffer_size

Value: 8388608

*************************** 4. row ***************************

Variable_name: innodb_log_file_size

Value: 5242880

*************************** 5. row ***************************

Variable_name: innodb_log_files_in_group

Value: 2

*************************** 6. row ***************************

Variable_name: innodb_log_group_home_dir

Value: ./

*************************** 7. row ***************************

Variable_name: innodb_mirrored_log_groups

Value: 1

7 rows in set (0.00 sec)
>

     重做日志文件的大小设置对于MySQL数据库各方面还是有影响的。一方面不能设置得太大，如果设置得很大，在恢复时可能需要很长的时间；另一方面又不能太小了，否则可能导致一个事务的日志需要多次切换重做日志文件。在错误日志中可能会看到如下警告：

>

> 090924 11:39:44  InnoDB: ERROR: the age of the last checkpoint is 9433712,
>

>
>

> InnoDB: which exceeds the log group capacity 9433498.

InnoDB: If you are using big BLOB or TEXT rows, you must set the

InnoDB: combined size of log files at least 10 times bigger than the

InnoDB: largest such row.

090924 11:40:00  InnoDB: ERROR: the age of the last checkpoint is 9433823,

InnoDB: which exceeds the log group capacity 9433498.

InnoDB: If you are using big BLOB or TEXT rows, you must set the

InnoDB: combined size of log files at least 10 times bigger than the

InnoDB: largest such row.

090924 11:40:16  InnoDB: ERROR: the age of the last checkpoint is 9433645,

InnoDB: which exceeds the log group capacity 9433498.

InnoDB: If you are using big BLOB or TEXT rows, you must set the

InnoDB: combined size of log files at least 10 times bigger than the

InnoDB: largest such row.
>

     上面错误集中在InnoDB: ERROR: the age of the last checkpoint is 9433645,InnoDB: which exceeds the log group capacity 9433498。这是因为重做日志有一个capacity变量，该值代表了最后的检查点不能超过这个阈值，如果超过则必须将缓冲池（innodb buffer pool）中刷新列表（flush list）中的部分脏数据页写回磁盘。

     也许有人会问，既然同样是记录事务日志，那和我们之前的二进制日志有什么区别？首先，二进制日志会记录所有与MySQL有关的日志记录，包括InnoDB、MyISAM、Heap等其他存储引擎的日志。而InnoDB存储引擎的重做日志只记录有关其本身的事务日志。其次，记录的内容不同，不管你将二进制日志文件记录的格式设为STATEMENT还是ROW，又或者是MIXED，其记录的都是关于一个事务的具体操作内容。而InnoDB存储引擎的重做日志文件记录的关于每个页（Page）的更改的物理情况（如表3-2所示）。此外，写入的时间也不同，二进制日志文件是在事务提交前进行记录的，而在事务进行的过程中，不断有重做日志条目（redo entry）被写入重做日志文件中。

表3-2   重做日志结构

Space id PageNo OpCode Data

在第2章中已经提到，对于写入重做日志文件的操作不是直接写，而是先写入一个重做日志缓冲（redo log buffer）中，然后根据按照一定的条件写入日志文件。图3-3很好地表示了这个过程。

图3-3   重做日志写入过程

上面提到了从日志缓冲写入磁盘上的重做日志文件是按一定条件的，那这些条件有哪些呢？第2章分析了主线程（master thread），知道在主线程中每秒会将重做日志缓冲写入磁盘的重做日志文件中，不论事务是否已经提交。另一个触发这个过程是由参数innodb_ flush_log_at_trx_commit控制，表示在提交（commit）操作时，处理重做日志的方式。

参数innodb_flush_log_at_trx_commit可设的值有0、1、2。0代表当提交事务时，并不将事务的重做日志写入磁盘上的日志文件，而是等待主线程每秒的刷新。而1和2不同的地方在于：1是在commit时将重做日志缓冲同步写到磁盘；2是重做日志异步写到磁盘，即不能完全保证commit时肯定会写入重做日志文件，只是有这个动作。