---
title: 'MySQL模式 : Strict Mode'
author: admin
type: post
date: 2010-08-01T14:45:49+00:00
url: /archives/4946
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---

**I. Strict Mode阐述**

根据 mysql5.0以上版本 strict mode (STRICT_TRANS_TABLES) 的限制：

1).不支持对not null字段插入null值

2).不支持对自增长字段插入”值，可插入null值

3).不支持 text 字段有默认值

看下面代码：(第一个字段为自增字段)

$query=”insert into demo values(”,’$firstname’,’$lastname’,’$sex’)”;

上边代码只在非strict模式有效。

$query=”insert into demo values(NULL,’$firstname’,’$lastname’,’$sex’)”;

上边代码只在strict模式有效。把空值”换成了NULL.

**II.让数据库支持Strict Mode**

1.对数据库结构进行以下改进来支持strict mode:

1) 给所有not null字段都设置非null默认值，字符串默认值为 ”，数值默认值为 0，日期默认值为 ‘0000-00-00 00:00:00’

2) 去掉text字段的默认值

3) 规范化改进: 把 title 字段统一改为 varchar(255)，把有默认值的null字段改为not null字段

2.如果安装的PHP程序数据库结构关闭Strict mode

1).一个是安装mysql5.0（含以上）版本的时候去掉strict mode。

编辑 my.cnf,关闭Strict Mode:

>

> sql-mode=”NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION”
>

2). 另一个就是修改查询语句。例如在

>

> if ($this->dbcharset) {
>

>
>

> @mysql_query(“SET NAMES “.$this->dbcharset);
>

>
>

> }
>

后面执行

>

> mysql_query(“SET @@sql_mode = ””);
>

注意确定你使用的是MySQL5.

mysqli方式类似，就是执行的是

mysqli_query($this->connection_id, “SET @@sql_mode = ””);

I. Strict Mode阐述 根据 mysql5.0以上版本 strict mode (STRICT\_TRANS\_TABLES) 的限制：
1).不支持对not null字段插入null值 2).不支持对自增长字段插入”值，可插入null值 3).不支持 text 字段有默认值
看下面代码：(第一个字段为自增字段) Sql代码 $query=”insert into demo values(”,’$firstname’,’$lastname’,’$sex’)”;
上边代码只在非strict模式有效。
Code代码 $query=”insert into demo values(NULL,’$firstname’,’$lastname’,’$sex’)”;
上边代码只在strict模式有效。把空值”换成了NULL.
II.让数据库支持Strict Mode
1.对数据库结构进行以下改进来支持strict mode:

1) 给所有not null字段都设置非null默认值，字符串默认值为 ”，数值默认值为 0，日期默认值为 ‘0000-00-00 00:00:00’ 2) 去掉text字段的默认值 3) 规范化改进: 把 title 字段统一改为 varchar(255)，把有默认值的null字段改为not null字段
2.如果安装的PHP程序数据库结构关闭Strict mode 1).一个是安装mysql5.0（含以上）版本的时候去掉strict mode。 编辑 my.cnf,关闭Strict Mode: sql-mode=”NO\_AUTO\_CREATE\_USER,NO\_ENGINE_SUBSTITUTION”
2). 另一个就是修改查询语句。

例如在

> if ($this->dbcharset) {
> @mysql_query(“SET NAMES “.$this->dbcharset);
> }

后面执行 mysql\_query(“SET @@sql\_mode = ””);

注意确定你使用的是MySQL5
mysqli方式类似，就是执行的是 mysqli\_query($this->connection\_id, “SET @@sql_mode = ””);