---
title: mysql集群从服务器复制传递和状态文件(master.info和relay_log.info)
author: admin
type: post
date: 2010-04-21T02:35:53+00:00
url: /archives/3463
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - 集群
 - mysql

---
### 6.3.4. 复制传递和状态文件

默认情况，中继日志使用_host_name-relay-bin.nnnnnn_形 式的文件名，其中_host_name_是从服务器主机名，_nnnnnn_是 序列号。用连续序列号来创建连续中继日志文件，从000001开始。从服务器跟踪索引文件中目前正 使用的中继日志。 默认中继日志索引文件名为_host_name-relay-bin.index_。 默认情况，在从服务器的数据目录中创建这些文件。可以用–relay-log和–relay-log-index服 务器选项覆盖 默认文件名。参见[6.8节，“复制启动选项”][1]。

中继日志与二进制日志的格式相同，并且可以用**mysqlbinlog**读取。SQL线 程执行完中继日志中的所有事件并且不再需要之后，立即自动删除它。没有直接的删除中继日志的机制，因为SQL线程可以负责完 成。然而，FLUSH LOGS可以循环中继日志，当SQL线程删除日志时会有影响。

在下面的条件下创建新的中继日志：

·每次I/O线程启动时创建一个新的中继日志。

·当日志被刷新时；例如，用FLUSH LOGS或**mysqladmin flush-logs**。

·当当前的中继日志文件变得太大时。“太大”含义的确定方法：

 omax_relay_log_size，如果max_relay_log_size> 0omax_binlog_size，如果max_relay_log_size= 0

从属复制服务器在数据目录中另外创建两个小文件。这些_状态文件_默认名为主master.info和relay-log.info。 它们包含SHOW SLAVE STATUS语句的输出所显示的信息(关于该语句的描述参见[13.6.2节，“用 于控制从服务器的SQL语句”][2])。状态文件保存在硬盘上，从服务器关闭时不会丢失。下次从服务器启动时，读取这些文件 以确定它已经从主服务器读取了多少二进制日志，以及处理自己的中继日志的程度。

由I/O线程更新master.info文件。文件中的行和SHOW SLAVE STATUS显示的列的对应关系为：

**行****描述**1
 文件中的行号
 2Master_Log_File3 Read_Master_Log_Pos4Master_Host5Master_User6
 密码(不由SHOW SLAVE STATUS显示)7Master_Port8Connect_Retry9 Master_SSL_Allowed10 Master_SSL_CA_File11 Master_SSL_CA_Path12Master_SSL_Cert13 Master_SSL_Cipher14Master_SSL_Key

由SQL线程更新relay-log.info文件。文件中的行和SHOW SLAVE STATUS显示的列的对应关系为：

**行****描述**1Relay_Log_File2Relay_Log_Pos3 Relay_Master_Log_File4 Exec_Master_Log_Pos

当备份从服务器的数据时，你还应备份这两个小文件以及中继日志文件。它们用来在恢复从服务器的数据后继续进行复制。如果丢失了中继日志但仍然有relay-log.info文 件，你可以通过检查该文件来确定SQL线程已经执行的主服务器中二进制日志的程度。然后可以用Master_Log_File和Master_LOG_POS选 项执行CHANGE MASTER TO来告诉从服务器重新从该点读取二进制日志。当然，要求二进制日志仍然在主服务器上。

如果从服务器正复制LOAD DATA INFILE语句，你应也备份该目录内从服务器用于该目的的任何SQL_LOAD-*文件。从 服务器需要这些文件继续复制任何中断的LOAD DATA INFILE操作。用–slave-load-tmpdir选项来指定目录的位置。如果未指 定， 默认值为tmpdir变量的值。

来源: [http://dev.mysql.com/doc/refman/5.1/zh/replication.html#slave-logs](http://dev.mysql.com/doc/refman/5.1/zh/replication.html#slave-logs)

对于二进制文件占用空间过大,可以删除一些无用的旧日志文件.参考教程:

 [1]: http://dev.mysql.com/doc/refman/5.1/zh/replication.html#replication-options "6.8. Replication Startup Options"
 [2]: http://dev.mysql.com/doc/refman/5.1/zh/sql-syntax.html#replication-slave-sql "13.6.2. SQL Statements for Controlling Slave Servers"