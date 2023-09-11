---
title: MySQL中的innodb_file_format 配置项解读
author: admin
type: post
date: 2018-08-20T09:02:37+00:00
url: /archives/18089
categories:
 - MySQL
tags:
 - mysql

---
## 一：innodb_file_format参数 

在 innodb 1.0.6版本之前，innodb文件格式`innodb_file_format`只有 Antelope(Antelope 文件格式支持Redundant，Compact两种格式来存放行记录，Redundant是为了兼容之前版本而保留的。在mysql 5.1版本中，默认设置为Compact，用户可以通过 `show table status like 'table_name'`来查看表使用的行格式row_format)

从innodb 1.0.6开始引入了新的文件格式 Barracuda 在原来的基础上(Antelope)新增了Dynamic和Compressed两种行格式。

## 二：innodb_file_format如何使用 

  一般， `innodb_file_format` 在配置文件中指定；`row_format`则在创建数据表时指定：

```
CREATE TABLE test2 (column1 INT PRIMARY KEY)
ENGINE=InnoDB ROW_FORMAT=Compressed KEY_BLOCK_SIZE=4;
```

## 三：innodb_file_format = Barracuda 

在指定innodb_file_format = Barracuda时，建议配合如下参数一起使用

```
innodb_file_format_max = Barracuda
innodb_strict_mode = 1
innodb_file_per_table = 1
```

因为system tablespace使用的是Antelope文件格式，如果不指定innodb_file_per_table = 1(5.6.6是默认开启的)，那么innodb表被存储在system tablespace,这时在建表的时候无法使用Dynamic和Compressed两种行格式(如果设置了innodb_strict_mode = 1，那么建表的时候指定Dynamic和Compressed会直接报错。如果没有指定innodb_strict_mode = 1，建表的时候指定Dynamic和Compressed会被忽略，默认设置为Compact)

关于innodb_strict_mode参数介绍：

innodb_strict_mode

Command-Line Format –innodb_strict_mode=#
System Variable Name innodb_strict_mode
Variable Scope Global, Session
Dynamic Variable Yes
Permitted Values Type boolean
Default OFF
When innodb_strict_mode is ON, InnoDB returns errors rather than warnings for certain conditions. The default value is OFF.

Strict mode helps guard against ignored typos and syntax errors in SQL, or other unintended consequences of various combinations of operational modes and SQL statements. When innodb_strict_mode is ON, InnoDB raises error conditions in certain cases, rather than issuing a warning and processing the specified statement (perhaps with unintended behavior). This is analogous to sql_mode in MySQL, which controls what SQL syntax MySQL accepts, and determines whether it silently ignores errors, or validates input syntax and data values.

The innodb_strict_mode setting affects the handling of syntax errors for CREATE TABLE, ALTER TABLE and CREATE INDEX statements.innodb_strict_mode also enables a record size check, so that an INSERT or UPDATE never fails due to the record being too large for the selected page size.

Oracle recommends enabling innodb_strict_mode when using ROW_FORMAT and KEY_BLOCK_SIZE clauses on CREATE TABLE, ALTER TABLE, andCREATE INDEX statements. When innodb_strict_mode is OFF, InnoDB ignores conflicting clauses and creates the table or index, with only a warning in the message log. The resulting table might have different behavior than you intended, such as having no compression when you tried to create a compressed table. When innodb_strict_mode is ON, such problems generate an immediate error and the table or index is not created, avoiding a troubleshooting session later.

You can turn innodb_strict_mode ON or OFF on the command line when you start mysqld, or in the configuration file my.cnf or my.ini. You can also enable or disable innodb_strict_mode at runtime with the statement SET [GLOBAL|SESSION] innodb_strict_mode=mode, where mode is either ONor OFF. Changing the GLOBAL setting requires the SUPER privilege and affects the operation of all clients that subsequently connect. Any client can change theSESSION setting for innodb_strict_mode, and the setting affects only that client.

关于innodb_file_per_table参数介绍：

 innodb_file_per_table

Command-Line Format –innodb_file_per_table
System Variable Name innodb_file_per_table
Variable Scope Global
Dynamic Variable Yes
Permitted Values (<= 5.6.5) Type boolean Default OFF Permitted Values (>= 5.6.6) Type boolean
Default ON
When innodb_file_per_table is enabled (the default in 5.6.6 and higher), InnoDB stores the data and indexes for each newly created table in a separate.ibd file, rather than in the system tablespace. The storage for these InnoDB tables is reclaimed when the tables are dropped or truncated. This setting enables several other InnoDB features, such as table compression. See Section 14.4.4, “InnoDB File-Per-Table Tablespaces” for details about such features as well as advantages and disadvantages of using file-per-table tablespaces.

Be aware that enabling innodb_file_per_table also means that an ALTER TABLE operation will move InnoDB table from the system tablespace to an individual .ibd file in cases where ALTER TABLE recreates the table (ALGORITHM=COPY).

When innodb_file_per_table is disabled, InnoDB stores the data for all tables and indexes in the ibdata files that make up the system tablespace. This setting reduces the performance overhead of filesystem operations for operations such as DROP TABLE or TRUNCATE TABLE. It is most appropriate for a server environment where entire storage devices are devoted to MySQL data. Because the system tablespace never shrinks, and is shared across all databases in an instance, avoid loading huge amounts of temporary data on a space-constrained system when innodb_file_per_table=OFF. Set up a separate instance in such cases, so that you can drop the entire instance to reclaim the space.

By default, innodb_file_per_table is enabled as of MySQL 5.6.6, disabled before that. Consider disabling it if backward compatibility with MySQL 5.5 or 5.1 is a concern. This will prevent ALTER TABLE from moving InnoDB tables from the system tablespace to individual .ibd files.

innodb_file_per_table is dynamic and can be set ON or OFF using SET GLOBAL. You can also set this parameter in the MySQL configuration file (my.cnfor my.ini) but this requires shutting down and restarting the server.

Dynamically changing the value of this parameter requires the SUPER privilege and immediately affects the operation of all connections.

关于各个参数的详细介绍，请参见官方文档。