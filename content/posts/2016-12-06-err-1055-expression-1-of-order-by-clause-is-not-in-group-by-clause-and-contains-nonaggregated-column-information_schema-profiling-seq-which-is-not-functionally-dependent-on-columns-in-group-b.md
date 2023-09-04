---
title: '[Err] 1055 – Expression #1 of ORDER BY clause is not in GROUP BY clause and contains nonaggregated column ‘information_schema.PROFILING.SEQ’ which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by的解决办法'
author: admin
type: post
date: 2016-12-06T11:07:28+00:00
url: /archives/17316
categories:
 - MySQL

---
线上用的MySQL版本为5.7.11，线下用的5.6版本，发现将程序上线后，有些地方报这个错误

> [Err] 1055 – Expression #1 of ORDER BY clause is not in GROUP BY clause and contains nonaggregated column ‘information_schema.PROFILING.SEQ’ which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by

**ONLY_FULL_GROUP_BY：**
对于GROUP BY聚合操作，若select中的列没有在group by中出现，那么这句SQL是不合法的。

解决办法下my.cnf中添加以下几行

```
[mysqld]
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'

```

在sql_mode 中去掉only_full_group_by

然后重启MySQL Server即可。

此ONLY_FULL_GROUP_BY的解释： [http://dev.mysql.com/doc/refman/5.7/en/group-by-handling.html](http://dev.mysql.com/doc/refman/5.7/en/group-by-handling.html)

==========================================

```
mysql> SELECT name, address, MAX(age) FROM t GROUP BY name;
ERROR 1055 (42000): Expression #2 of SELECT list is not in GROUP
BY clause and contains nonaggregated column 'mydb.t.address' which
is not functionally dependent on columns in GROUP BY clause; this
is incompatible with sql_mode=only_full_group_by

```

If you know that, _for a given data set,_ each `name` value in fact uniquely determines the `address` value, `address` is effectively functionally dependent on `name`. To tell MySQL to accept the query, you can use the [ `ANY_VALUE()`][1]{.link} function:

```
SELECT name, ANY_VALUE(address), MAX(age) FROM t GROUP BY name;

```

Alternatively, disable [ `ONLY_FULL_GROUP_BY`][2]{.link}.

对于以前的语句，可以使用ANY_VALUE() 函数来解决。
对于SQL_MODE各值的用法见：http://blog.itpub.net/29773961/viewspace-1813501/

 [1]: http://dev.mysql.com/doc/refman/5.7/en/miscellaneous-functions.html#function_any-value
 [2]: http://dev.mysql.com/doc/refman/5.7/en/sql-mode.html#sqlmode_only_full_group_by