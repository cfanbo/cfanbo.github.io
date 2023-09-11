---
title: 'MySQL之Handler_read_*'
author: admin
type: post
date: 2010-07-02T01:32:37+00:00
url: /archives/4262
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql优化
 - 查询优化

---
在MySQL里，我们一般使用 [SHOW STATUS](http://dev.mysql.com/doc/refman/5.0/en/show-status.html) 查询服务器状态，语法一般来说如下：

SHOW [GLOBAL | SESSION] STATUS [LIKE ‘pattern’ | WHERE expr]

执行命令后会看到很多内容，其中有一部分是Handler\_read\_*，它们显示了数据库处理SELECT查询语句的状态，对于调试SQL语句有很大意 义，可惜实际很多人并不理解它们的实际意义，本文简单介绍一下：

为了让介绍更易懂，先建立一个测试用的表：

CREATE TABLE IF NOT EXISTS `foo` (

 `id` int(10) unsigned NOT NULL auto_increment,

 `col1` varchar(10) NOT NULL,

 `col2` text NOT NULL,

 PRIMARY KEY (`id`),

 KEY `col1` (`col1`)

 );

INSERT INTO \`foo\` (\`id\`, \`col1\`, \`col2\`) VALUES
(1, ‘a’, ‘a’),
(2, ‘b’, ‘b’),
(3, ‘c’, ‘c’),
(4, ‘d’, ‘d’),
(5, ‘e’, ‘e’),
(6, ‘f’, ‘f’),
(7, ‘g’, ‘g’),
(8, ‘h’, ‘h’),
(9, ‘i’, ‘i’);

在下面的测试里，每次执行SQL时按照如下过程执行：

FLUSH STATUS;

 SELECT …;

 SHOW SESSION STATUS LIKE ‘Handler_read%’;

 EXPLAIN SELECT …;

**Handler\_read\_first**

The number of times the first entry was read from an index. If this value is high, it suggests that the server is doing a lot of full index scans; for example, SELECT col1 FROM foo, assuming that col1 is indexed.

此选项表明SQL是在做一个全索引扫描，注意是全部，而不是部分，所以说如果存在WHERE语句，这个选项是不会变的。如果这个选项的数值很大，既是好事 也是坏事。说它好是因为毕竟查询是在索引里完成的，而不是数据文件里，说它坏是因为大数据量时，简便是索引文件，做一次完整的扫描也是很费时的。

FLUSH STATUS;

SELECT col1 FROM foo;

mysql> SHOW SESSION STATUS LIKE ‘Handler_read%’;

 +———————–+——-+

 | Variable_name         | Value |

 +———————–+——-+

 | Handler_read_first    | 1     |

 | Handler_read_key      | 0     |

 | Handler_read_next     | 9     |

 | Handler_read_prev     | 0     |

 | Handler_read_rnd      | 0     |

 | Handler_read_rnd_next | 0     |

 +———————–+——-+

 6 rows in set (0.00 sec)

mysql> EXPLAIN SELECT col1 FROM foo\G

type: index

 Extra: Using index


**Handler\_read\_key**

The number of requests to read a row based on a key. If this value is high, it is a good indication that your tables are properly indexed for your queries.

此选项数值如果很高，那么恭喜你，你的系统高效的使用了索引，一切运转良好。

FLUSH STATUS;

SELECT * FROM foo WHERE col1 = ‘e’;

mysql> SHOW SESSION STATUS LIKE ‘Handler_read%’;
+———————–+——-+
| Variable_name         | Value |
+———————–+——-+
| Handler\_read\_first    | 0     |
| Handler\_read\_key      | 1     |
| Handler\_read\_next     | 1     |
| Handler\_read\_prev     | 0     |
| Handler\_read\_rnd      | 0     |
| Handler\_read\_rnd_next | 0     |
+———————–+——-+

mysql> EXPLAIN SELECT * FROM foo WHERE col1 = ‘e’\G
type: ref
Extra: Using where

**
Handler\_read\_next**

The number of requests to read the next row in key order. This value is incremented if you are querying an index column with a range constraint or if you are doing an index scan.

此选项表明在进行索引扫描时，按照索引从数据文件里取数据的次数。

FLUSH STATUS;

SELECT col1 FROM foo ORDER BY col1 ASC;

mysql> SHOW SESSION STATUS LIKE ‘Handler_read%’;
+———————–+——-+
| Variable_name         | Value |
+———————–+——-+
| Handler\_read\_first    | 1     |
| Handler\_read\_key      | 0     |
| Handler\_read\_next     | 9     |
| Handler\_read\_prev     | 0     |
| Handler\_read\_rnd      | 0     |
| Handler\_read\_rnd_next | 0     |
+———————–+——-+

mysql> EXPLAIN SELECT * FROM foo WHERE col1 = ‘e’\G
type: index
Extra: Using index

**Handler\_read\_prev**

The number of requests to read the previous row in key order. This read method is mainly used to optimize ORDER BY … DESC.

此选项表明在进行索引扫描时，按照索引倒序从数据文件里取数据的次数，一般就是ORDER BY … DESC。

FLUSH STATUS;

SELECT col1 FROM foo ORDER BY col1 DESC;

mysql> SHOW SESSION STATUS LIKE ‘Handler_read%’;
+———————–+——-+
| Variable_name         | Value |
+———————–+——-+
| Handler\_read\_first    | 0     |
| Handler\_read\_key      | 0     |
| Handler\_read\_next     | 0     |
| Handler\_read\_prev     | 9     |
| Handler\_read\_rnd      | 0     |
| Handler\_read\_rnd_next | 0     |
+———————–+——-+

mysql> EXPLAIN SELECT col1 FROM foo ORDER BY col1 DESC\G
type: index
Extra: Using index
**
Handler\_read\_rnd**

The number of requests to read a row based on a fixed position. This value is high if you are doing a lot of queries that require sorting of the result. You probably have a lot of queries that require MySQL to scan entire tables or you have joins that don’t use keys properly.

简单的说，就是查询直接操作了数据文件，很多时候表现为没有使用索引或者文件排序。

FLUSH STATUS;

 SELECT * FROM foo ORDER BY col2 DESC;

mysql> SHOW SESSION STATUS LIKE ‘Handler_read%’;
+———————–+——-+
| Variable_name         | Value |
+———————–+——-+
| Handler\_read\_first    | 0     |
| Handler\_read\_key      | 0     |
| Handler\_read\_next     | 0     |
| Handler\_read\_prev     | 0     |
| Handler\_read\_rnd      | 9     |
| Handler\_read\_rnd_next | 10    |
+———————–+——-+

mysql> EXPLAIN SELECT * FROM foo ORDER BY col2 DESC\G
type: ALL
Extra: Using filesort

**Handler\_read\_rnd_next**

The number of requests to read the next row in the data file. This value is high if you are doing a lot of table scans. Generally this suggests that your tables are not properly indexed or that your queries are not written to take advantage of the indexes you have.

此选项表明在进行数据文件扫描时，从数据文件里取数据的次数。

FLUSH STATUS;

SELECT * FROM foo;

mysql> SHOW SESSION STATUS LIKE ‘Handler_read%’;
+———————–+——-+
| Variable_name         | Value |
+———————–+——-+
| Handler\_read\_first    | 0     |
| Handler\_read\_key      | 0     |
| Handler\_read\_next     | 0     |
| Handler\_read\_prev     | 0     |
| Handler\_read\_rnd      | 0     |
| Handler\_read\_rnd_next | 10    |
+———————–+——-+

mysql> EXPLAIN SELECT * FROM foo ORDER BY col2 DESC\G
type: ALL
Extra: Using filesort

后记：不同平台，不同版本的MySQL，在运行上面例子的时候，Handler\_read\_\*的数值可能会有所不同，这并不要紧，关键是你要意识到 Handler\_read\_\*可以协助你理解MySQL处理查询的过程，很多时候，为了完成一个查询任务，我们往往可以写出几种查询语句，这时，你不妨挨 个按照上面的方式执行，根据结果中的Handler\_read\_*数值，你就能相对容易的判断各种查询方式的优劣。

说到判断查询方式优劣这个问题，就再顺便提提show profile语法，在新版MySQL里提供了这个功能：

mysql> set profiling=on;

mysql> use mysql;
mysql> select * from user;

mysql> show profile;
+——————–+———-+
| Status             | Duration |
+——————–+———-+
| starting           | 0.000078 |
| Opening tables     | 0.000022 |
| System lock        | 0.000010 |
| Table lock         | 0.000014 |
| init               | 0.000054 |
| optimizing         | 0.000008 |
| statistics         | 0.000015 |
| preparing          | 0.000014 |
| executing          | 0.000007 |
| Sending data       | 0.000139 |
| end                | 0.000007 |
| query end          | 0.000007 |
| freeing items      | 0.000044 |
| logging slow query | 0.000004 |
| cleaning up        | 0.000005 |
+——————–+———-+
15 rows in set (0.00 sec)

mysql> show profiles;
+———-+————+——————–+
| Query_ID | Duration   | Query              |
+———-+————+——————–+
|        1 | 0.00017725 | SELECT DATABASE().|

 |        2 | 0.00042675 | select * from user |

 +———-+————+——————–+

 2 rows in set (0.00 sec)

参考链接： [http://www.fromdual.com/mysql-handler-read-status-variables](http://www.fromdual.com/mysql-handler-read-status-variables)