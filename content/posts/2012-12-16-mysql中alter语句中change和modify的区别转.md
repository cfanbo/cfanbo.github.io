---
title: mysql中alter语句中change和modify的区别
author: admin
type: post
date: 2012-12-16T14:07:01+00:00
url: /archives/13537
categories:
 - MySQL
tags:
 - mysql

---
以下摘自mysql5手册

您可以使用CHANGE _old\_col\_name__column_definition_子句对列进行重命名。重命名时，需给定旧的和新的列名称和列当前的类型。例如：要把一个INTEGER列的名称从a变更到b，您需要如下操作：

·                mysql> **ALTER TABLE t1 CHANGE a b INTEGER;**

如果您想要更改列的类型而不是名称， CHANGE语法仍然要求旧的和新的列名称，即使旧的和新的列名称是一样的。例如：

mysql> **ALTER TABLE t1 CHANGE b b BIGINT NOT NULL;**

您也可以使用MODIFY来改变列的类型，此时不需要重命名：

mysql> **ALTER TABLE t1 MODIFY b BIGINT NOT NULL;**

mysql alter 语句用法,添加、修改、删除字段等

//主键549830479

alter table tabelname add new\_field\_id int(5) unsigned default 0 not null auto\_increment ,add primary key (new\_field_id);
//增加一个新列549830479

alter table t2 add d timestamp;
alter table infos add ex tinyint not null default ‘0’;
//删除列549830479

alter table t2 drop column c;
//重命名列549830479

alter table t1 change a b integer;

//改变列的类型549830479

alter table t1 change b b bigint not null;
alter table infos change list list tinyint not null default ‘0’;

//重命名表549830479

alter table t1 rename t2;
加索引549830479

mysql> alter table tablename change depno depno int(5) not null;
mysql> alter table tablename add index 索引名 (字段名1[，字段名2 …]);
mysql> alter table tablename add index emp_name (name);
加主关键字的索引549830479

mysql> alter table tablename add primary key(id);
加唯一限制条件的索引549830479

mysql> alter table tablename add unique emp_name2(cardnumber);
删除某个索引549830479

mysql>alter table tablename drop index emp_name;
修改表：549830479

增加字段：549830479

mysql> ALTER TABLE table\_name ADD field\_name field_type;
修改原字段名称及类型：549830479

mysql> ALTER TABLE table\_name CHANGE old\_field\_name new\_field\_name field\_type;
删除字段：549830479

mysql> ALTER TABLE table\_name DROP field\_name;

删除主键： alter table Employees drop primary key;