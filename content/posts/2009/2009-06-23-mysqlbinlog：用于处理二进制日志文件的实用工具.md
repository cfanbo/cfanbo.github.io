---
title: mysqlbinlog：用于处理二进制日志文件的实用工具
author: admin
type: post
date: 2009-06-23T01:47:08+00:00
excerpt: |
 服务器生成的二进制日志文件写成二进制格式。要想检查这些文本格式的文件，应使用mysqlbinlog实用工具。

 应这样调用mysqlbinlog：

 shell> mysqlbinlog [options] log-files...

 例如，要想显示二进制日志binlog.000003的内容，使用下面的命令：

 shell> mysqlbinlog binlog.0000003
 输出包括在binlog.000003中包含的所有语句，以及其它信息例如每个语句花费的时间、客户发出的线程ID、发出线程时的时间戳等等。

 通常情况，可以使用mysqlbinlog直接读取二进制日志文件并将它们用于本地MySQL服务器。也可以使用--read-from-remote-server选项从远程服务器读取二进制日志。

 当读取远程二进制日志时，可以通过连接参数选项来指示如何连接服务器，但它们经常被忽略掉，除非你还指定了--read-from-remote-server选项。这些选项是--host、--password、--port、--protocol、--socket和--user。
url: /archives/1899
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql
 - mysqlbinlog

---

## 服务器生成的二进制日志文件写成二进制格式。要想检查这些文本格式的文件，应使用mysqlbinlog实用工具。

应这样调用mysqlbinlog：shell> **mysqlbinlog [options] _log-files_…**

例如，要想显示二进制日志binlog.000003的内容，使用下面的命令：

```
shell> mysqlbinlog binlog.0000003
```

输出包括在binlog.000003中包含的所有语句，以及其它信息例如每个语句花费的时间、客户发出的线程ID、发出线程时的时间戳等等。

通常情况，可以使用**mysqlbinlog**直接读取二进制日志文件并将它们用于本地MySQL服务器。也可以使用–read-from-remote-server选项从远程服务器读取二进制日志。

当读取远程二进制日志时，可以通过连接参数选项来指示如何连接服务器，但它们经常被忽略掉，除非你还指定了–read-from-remote-server选项。这些选项是–host、–password、–port、–protocol、–socket和–user。

还可以使用**mysqlbinlog**来读取在复制过程中从服务器所写的中继日志文件。中继日志格式与二进制日志文件相同。

在[5.11.3节，“二进制日志”][1]中详细讨论了二进制日志。

**mysqlbinlog**支持下面的选项：

·—help，–？

显示帮助消息并退出。

·—database= _db_name_，-d _db_name_

只列出该数据库的条目(只用本地日志)。

·–force-read，-f

使用该选项，如果**mysqlbinlog**读它不能识别的二进制日志事件，它会打印警告，忽略该事件并继续。没有该选项，如果**mysqlbinlog**读到此类事件则停止。

·–hexdump，-H

在注释中显示日志的十六进制转储。该输出可以帮助复制过程中的调试。在MySQL 5.1.2中添加了该选项。

·–host= _host_name_，-h _host_name_

获取给定主机上的MySQL服务器的二进制日志。

·–local-load= _path_，-l _pat_

为指定目录中的LOAD DATA INFILE预处理本地临时文件。

·–offset= _N_，-o _N_

跳过前_N_个条目。

·–password[= _password_]，-p[ _password_]

当连接服务器时使用的密码。如果使用短选项形式(-p)，选项和 密码之间_不能_有空格。如果在命令行中–password或-p选项后面没有 密码值，则提示输入一个密码。

·–port= _port_num_，-P port_ _num_

用于连接远程服务器的TCP/IP端口号。

·–position= _N_，-j _N_

不赞成使用，应使用–start-position。

·–protocol={TCP | SOCKET | PIPE | -position

使用的连接协议。

·–read-from-remote-server，-R

从MySQL服务器读二进制日志。如果未给出该选项，任何连接参数选项将被忽略。这些选项是–host、–password、–port、–protocol、–socket和–user。

·–result-file= _name_, -r _name_

将输出指向给定的文件。

·–short-form，-s

只显示日志中包含的语句，不显示其它信息。

·–socket= _path_，-S _path_

用于连接的套接字文件。

·–start-datetime= _datetime_

从二进制日志中第1个日期时间等于或晚于_datetime_参量的事件开始读取。_datetime_值相对于运行**mysqlbinlog**的机器上的本地时区**。该**值格式应符合DATETIME或TIMESTAMP数据类型。例如：

```
shell> mysqlbinlog --start-datetime="2004-12-25 11:25:56" binlog.000003
```

该选项可以帮助点对点恢复。

·–stop-datetime= _datetime_

从二进制日志中第1个日期时间等于或晚于_datetime_参量的事件起停止读。关于_datetime_值的描述参见–start-datetime选项。该选项可以帮助及时恢复。

·–start-position= _N_

从二进制日志中第1个位置等于_N_参量时的事件开始读。

·–stop-position= _N_

从二进制日志中第1个位置等于和大于_N_参量时的事件起停止读。

·–to-last-logs，-t

在MySQL服务器中请求的二进制日志的结尾处不停止，而是继续打印直到最后一个二进制日志的结尾。如果将输出发送给同一台MySQL服务器，会导致无限循环。该选项要求–read-from-remote-server。

·—disable-logs-bin，-D

禁用二进制日志。如果使用–to-last-logs选项将输出发送给同一台MySQL服务器，可以避免无限循环。该选项在崩溃恢复时也很有用，可以避免复制已经记录的语句。**注释：**该选项要求有SUPER权限。

·–user= _user_name_，-u _user_name_

连接远程服务器时使用的MySQL用户名。

·–version，-V

显示版本信息并退出。

还可以使用–var_name=value选项设置下面的变量：

·open_files_limit

指定要保留的打开的文件描述符的数量。

可以将**mysqlbinlog**的输出传到**mysql**客户端以执行包含在二进制日志中的语句。如果你有一个旧的备份，该选项在崩溃恢复时也很有用(参见[5.9.1节，“数据库备份”][2])：

```
shell> mysqlbinlog hostname-bin.000001 | mysql
```

或：

```
shell> mysqlbinlog hostname-bin.[0-9]* | mysql
```

如果你需要先修改含语句的日志，还可以将**mysqlbinlog**的输出重新指向一个文本文件。(例如，想删除由于某种原因而不想执行的语句)。编辑好文件后，将它输入到**mysql**程序并执行它包含的语句。

**mysqlbinlog**有一个–position选项，只打印那些在二进制日志中的偏移量大于或等于某个给定位置的语句(给出的位置必须匹配一个事件的开始)。它还有在看见给定日期和时间的事件后停止或启动的选项。这样可以使用–stop-datetime选项进行点对点恢复(例如，能够说“将数据库前滚动到今天10:30 AM的位置”)。

如果MySQL服务器上有多个要执行的二进制日志，安全的方法是在一个连接中处理它们。下面是一个说明什么是_不安全_的例子：

```
shell> mysqlbinlog hostname-bin.000001 | mysql # DANGER!!
```

```
shell> mysqlbinlog hostname-bin.000002 | mysql # DANGER!!
```

使用与服务器的不同连接来处理二进制日志时，如果第1个日志文件包含一个CREATE TEMPORARY TABLE语句，第2个日志包含一个使用该临时表的语句，则会造成问题。当第1个**mysql**进程结束时，服务器撤销临时表。当第2个**mysql**进程想使用该表时，服务器报告 “不知道该表”。

要想避免此类问题，使用一个连接来执行想要处理的所有二进制日志中的内容。下面提供了一种方法：

```
shell> mysqlbinlog hostname-bin.000001 hostname-bin.000002 | mysql
```

另一个方法是：

```
shell> mysqlbinlog hostname-bin.000001 >   /tmp/statements.sql
```

```
shell> mysqlbinlog hostname-bin.000002 >> /tmp/statements.sql
```

```
shell> mysql -e "source /tmp/statements.sql"
```

**mysqlbinlog**产生的输出可以不需要原数据文件即可重新生成一个LOAD DATA INFILE操作。**mysqlbinlog**将数据复制到一个临时文件并写一个引用该文件的LOAD DATA LOCAL INFILE语句。由系统确定写入这些文件的目录的默认位置。要想显式指定一个目录，使用–local-load选项。

因为**mysqlbinlog**可以将LOAD DATA INFILE语句转换为LOAD DATA LOCAL INFILE语句(也就是说，它添加了LOCAL)，用于处理语句的客户端和服务器必须配置为允许LOCAL操作。参见[5.6.4节，“LOAD DATA LOCAL安全问题”][3]。

**警告：**为LOAD DATA LOCAL语句创建的临时文件不会自动删除，因为在实际执行完那些语句前需要它们。不再需要语句日志后应自己删除临时文件。文件位于临时文件目录中，文件名类似original_file_name-#-#。

–hexdump选项可以在注释中产生日志内容的十六进制转储：

```
shell> mysqlbinlog --hexdump master-bin.000001
```

上述命令的输出应类似：

```
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
```

```
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
```

```
# at 4
```

```
#051024 17:24:13 server id 1   end_log_pos 98
```

```
# Position   Timestamp    Type    Master ID         Size       Master Pos     Flags
```

```
# 00000004 9d fc 5c 43    0f    01 00 00 00    5e 00 00 00    62 00 00 00    00 00
```

```
# 00000017 04 00 35 2e 30 2e 31 35   2d 64 65 62 75 67 2d 6c |..5.0.15.debug.l|
```

```
# 00000027 6f 67 00 00 00 00 00 00   00 00 00 00 00 00 00 00 |og..............|
```

```
# 00000037 00 00 00 00 00 00 00 00   00 00 00 00 00 00 00 00 |................|
```

```
# 00000047 00 00 00 00 9d fc 5c 43   13 38 0d 00 08 00 12 00 |.......C.8......|
```

```
# 00000057 04 04 04 04 12 00 00 4b   00 04 1a                 |.......K...|
```

```
#        Start: binlog v 4, server v 5.0.15-debug-log created 051024 17:24:13
```

```
#        at startup
```

```
ROLLBACK;
```

十六进制转储的输出包含下面的元素：

·Position: The byte position within the log file.·Timestamp: The event timestamp. In the example just shown, ‘9d fc 5c 43’is the representation of ‘051024 17:24:13’in hexadecimal.·Type: The type of the log event. ‘0f’means that the example event is a FORMAT_DESCRIPTION_EVENT. The types are:

```
·                00   UNKNOWN_EVENT
```

```
·                     This event should never be present in the log.
```

```
·                01   START_EVENT_V3
```

```
·                     This indicates the start of a log file written by MySQL 4 or earlier.
```

```
·                02   QUERY_EVENT
```

```
·                     The most common type of events.   These contain queries executed
```

```
·                     on the master.
```

```
·                03   STOP_EVENT
```

```
·                     Indicates that master has stopped.
```

```
·                04   ROTATE_EVENT
```

```
·                     Written when the master switches to a new log file.
```

```
·                05   INTVAR_EVENT
```

```
·                     Used mainly for AUTO_INCREMENT values and if the LAST_INSERT_ID()
```

```
·                     function is used in the statement.
```

```
·                06   LOAD_EVENT
```

```
·                     Used for LOAD DATA INFILE in MySQL 3.23.
```

```
·                07   SLAVE_EVENT
```

```
·                     Reserved for future use.
```

```
·                08   CREATE_FILE_EVENT
```

```
·                     Used for LOAD DATA INFILE statements.   This indicates the start
```

```
·                     of execution of such a statement.   A temporary file is created
```

```
·                     on the slave.   Used in MySQL 4 only.
```

```
·                09   APPEND_BLOCK_EVENT
```

```
·                     Contains data for use in a LOAD DATA INFILE statement.   The
```

```
·                     data is stored in the temporary file on the slave.
```

```
·                0a   EXEC_LOAD_EVENT
```

```
·                     Used for LOAD DATA INFILE statements.   The contents of the
```

```
·                     temporary file is stored in the table on the slave.
```

```
·                     Used in MySQL 4 only.
```

```
·                0b   DELETE_FILE_EVENT
```

```
·                     Rollback of LOAD DATA INFILE statement.   The temporary file
```

```
·                     should be deleted on slave.
```

```
·                0c   NEW_LOAD_EVENT
```

```
·                     Used for LOAD DATA INFILE in MySQL 4 and earlier.
```

```
·                0d   RAND_EVENT
```

```
·                     Used to send information about random values if the RAND()
```

```
·                     function is used in the query.
```

```
·                0e   USER_VAR_EVENT
```

```
·                     Used to replicate user variables.
```

```
·                0f   FORMAT_DESCRIPTION_EVENT
```

```
·                     This indicates the start of a log file written by MySQL 5 or later.
```

```
·                10   XID_EVENT
```

```
·                     Event indicating commit of XA transaction
```

```
·                11   BEGIN_LOAD_QUERY_EVENT
```

```
·                     Used for LOAD DATA statements in MySQL 5 and later.
```

```
·                12   EXECUTE_LOAD_QUERY_EVENT
```

```
·                     Used for LOAD DATA statements in MySQL 5 and later.
```

```
·                13   TABLE_MAP_EVENT
```

```
·                     Reserved for future use
```

```
·                14   WRITE_ROWS_EVENT
```

```
·                     Reserved for future use
```

```
·                15   UPDATE_ROWS_EVENT
```

```
·                     Reserved for future use
```

```
·                16   DELETE_ROWS_EVENT
```

```
·                     Reserved for future use
```

·Master ID: The server id of the master that created the event.·Size: The size in bytes of the event.·Master Pos: The position of the event in the original master log file.·Flags: 16 flags.

```
·                01   LOG_EVENT_BINLOG_IN_USE_F
```

```
·                     Log file correctly closed (Used only in FORMAT_DESCRIPTION_EVENT)
```

```
·                     If this flag is set (if the flags are e.g. '01 00') in an
```

```
·                     FORMAT_DESCRIPTION_EVENT, then the log file has not been
```

```
·                     properly closed.   Most probably because of a master crash (for
```

```
·                     example, due to power failure).
```

```
·                02   Reserved for future use.
```

```
·                04   LOG_EVENT_THREAD_SPECIFIC_F
```

```
·                     Set if the event is dependent on the connection it was
```

```
·                     executed in (example '04 00'), e.g. if the event uses
```

```
·                     temporary tables.
```

```
·                08   LOG_EVENT_SUPPRESS_USE_F
```

```
·                     Set in some circumstances when the event is not dependent on
```

```
·                     the current database
```

其它标志保留用于将来使用。

在以后的版本中十六进制转储输出的格式可能会改变。

原文来自:MySQL 5.1参考手册

 [1]: http://dev.mysql.com/doc/refman/5.1/zh/database-administration.html#binary-log "5.11.3. The Binary Log"
 [2]: http://dev.mysql.com/doc/refman/5.1/zh/database-administration.html#backup "5.9.1. Database Backups"
 [3]: http://dev.mysql.com/doc/refman/5.1/zh/database-administration.html#load-data-local "5.6.4. Security Issues with LOAD DATA LOCAL"