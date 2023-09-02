---
title: show profiles 详解
author: admin
type: post
date: 2010-07-09T07:15:06+00:00
url: /archives/4561
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql
 - mysql优化
 - 查询优化

---
[https://dev.mysql.com/doc/refman/5.7/en/show-profile.html](https://dev.mysql.com/doc/refman/5.7/en/show-profile.html)

此功能将在新版本中被移除，性能分析使用新方法来代替。(官方提供了此命令的使用方法, 对于 show profile for query ID / show profile  **CPU** for query ID 结果中每项的说明信息请参考： [https://www.cnblogs.com/itcomputer/articles/5056127.html](https://www.cnblogs.com/itcomputer/articles/5056127.html)）

>
>
>

>

>

> **Note**
>

>
>

> These statements are deprecated and will be removed in a future MySQL release. Use the Performance Schema instead; see [Chapter 25, _MySQL Performance Schema_](https://dev.mysql.com/doc/refman/5.7/en/performance-schema.html "Chapter 25 MySQL Performance Schema").
>

>

>

对于新版本我们也可以直接查询  `INFORMATION_SCHEMA` [`PROFILING`](https://dev.mysql.com/doc/refman/8.0/en/profiling-table.html "25.20 The INFORMATION_SCHEMA PROFILING Table") . See [Section 25.20, “The INFORMATION_SCHEMA PROFILING Table”](https://dev.mysql.com/doc/refman/8.0/en/profiling-table.html "25.20 The INFORMATION_SCHEMA PROFILING Table").


例如：


```
SHOW PROFILE FOR QUERY 2;
```

```
SELECT STATE, FORMAT(DURATION, 6) AS DURATION FROM INFORMATION_SCHEMA.PROFILING WHERE QUERY_ID = 2 ORDER BY SEQ;
```