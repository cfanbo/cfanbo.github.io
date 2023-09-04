---
title: MySQL中的sql_mode模式
author: admin
type: post
date: 2018-06-07T07:42:24+00:00
url: /archives/17847
categories:
 - MySQL

---
官方文档： [https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sql-mode-setting](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sql-mode-setting)

**一、模式分类**

在MySQL8.0中主要包括以下几种模式

| **ONLY_FULL_GROUP_BY**         | 对于GROUP BY聚合操作，如果在SELECT中的列，没有在GROUP BY中出现，那么将认为这个SQL是不合法的，因为列不在GROUP BY从句中 |
| ------------------------------ | ------------------------------------------------------------ |
| **STRICT_TRANS_TABLES**        | 在该模式下，如果一个值不能插入到一个事务表中，则中断当前的操作，对非事务表不做任何限制 |
| **NO_ZERO_IN_DATE**            | 在严格模式，不接受月或日部分为0的日期。如果使用IGNORE选项，我们为类似的日期插入’0000-00-00’。在非严格模式，可以接受该日期，但会生成警告。 |
| **NO_ZERO_DATE**               | 在严格模式，不要将 ‘0000-00-00’做为合法日期。你仍然可以用IGNORE选项插入零日期。在非严格模式，可以接受该日期，但会生成警告 |
| **ERROR_FOR_DIVISION_BY_ZERO** | 在严格模式，在INSERT或UPDATE过程中，如果被零除(或MOD(X，0))，则产生错误(否则为警告)。如果未给出该模式，被零除时MySQL返回NULL。如果用到INSERT IGNORE或UPDATE IGNORE中，MySQL生成被零除警告，但操作结果为NULL。 |
| **NO_ENGINE_SUBSTITUTION**     | 如果需要的存储引擎被禁用或未编译，那么抛出错误。不设置此值时，用默认的存储引擎替代，并抛出一个异常。 |



修改当前数据库的模式，有两种办法，一种是全局，一种是会话期间

```
SET GLOBAL sql_mode = 'modes';
SET SESSION sql_mode = 'modes';

```

同样，查看方法为

``` sql
SELECT @@GLOBAL.sql_mode;
SELECT @@SESSION.sql_mode;
```

**二、最重要的几种sql_mode**

| **ANSI模式**                | 宽松模式，对插入数据进行校验，如果不符合定义类型或长度，对数据类型调整或截断保存，报warning警告。 ANSI模式等于 [`REAL_AS_FLOAT`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_real_as_float), [`PIPES_AS_CONCAT`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_pipes_as_concat), [`ANSI_QUOTES`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_ansi_quotes), [`IGNORE_SPACE`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_ignore_space), and [`ONLY_FULL_GROUP_BY`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by)几种的组合。 |
| --------------------------- | ------------------------------------------------------------ |
| **TRADITIONAL模式**         | 严格模式，当向mysql数据库插入数据时，进行数据的严格校验，保证错误数据不能插入，报error错误。用于事物时，会进行事物的回滚。等于[`STRICT_TRANS_TABLES`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_strict_trans_tables), [`STRICT_ALL_TABLES`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_strict_all_tables), [`NO_ZERO_IN_DATE`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_no_zero_in_date), [`NO_ZERO_DATE`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_no_zero_date),[`ERROR_FOR_DIVISION_BY_ZERO`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_error_for_division_by_zero), and [`NO_ENGINE_SUBSTITUTION`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_no_engine_substitution).的组合 |
| **STRICT_TRANS_TABLES模式** | 严格模式，进行数据的严格校验，错误数据不能插入，报error错误。 |

目前新版本MySQL5.7.22的默认模式为 **STRICT_TRANS_TABLES**, **NO_ENGINE_SUBSTITUTION** 两种。对于不同版本默认的sql_mode可查看https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_sql_mode