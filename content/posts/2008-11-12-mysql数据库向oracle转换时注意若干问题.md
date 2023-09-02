---
title: MySQL数据库向Oracle转换时注意若干问题
author: admin
type: post
date: 2008-11-12T09:12:38+00:00
excerpt: |
 |
 　　有很多应用项目, 刚起步的时候用MySQL数据库基本上能实现各种功能需求，随着应用用户的增多，数据量的增加，MySQL渐渐地出现不堪重负的情况：连接很慢甚至宕机，于是就有把数据从MySQL迁到 Oracle的需求，应用程序也要相应做一些修改。本人总结出以下几点注意事项，希望对大家有所帮助。

 　　1． 自动增长的数据类型处理 MySQL有自动增长的数据类型，插入记录时不用操作此字段，会自动获得数据值。 Oracle没有自动增长的数据类型，需要建立一个自动增长的序列号，插入记录时要把序列号的下一个值赋于此字段。

 　　CREATE SEQUENCE 序列号的名称 (最好是表名 序列号标记) INCREMENT BY 1 START WITH 1 MAXVALUE 99999 CYCLE NOCACHE；其中最大的值按字段的长度来定, 如果定义的自动增长的序列号 NUMBER(6) , 最大值为999999 INSERT 语句插入这个字段值为: 序列号的名称.NEXTVAL
url: /archives/552
IM_contentdowned:
 - 1
categories:
 - 数据库
tags:
 - mysql

---
　　有很多应用项目, 刚起步的时候用MySQL数据库基本上能实现各种功能需求，随着应用用户的增多，数据量的增加，MySQL渐渐地出现不堪重负的情况：连接很慢甚至宕机，于是就有把数据从MySQL迁到 Oracle的需求，应用程序也要相应做一些修改。本人总结出以下几点注意事项，希望对大家有所帮助。

　　1． 自动增长的数据类型处理 MySQL有自动增长的数据类型，插入记录时不用操作此字段，会自动获得数据值。 Oracle没有自动增长的数据类型，需要建立一个自动增长的序列号，插入记录时要把序列号的下一个值赋于此字段。

　　CREATE SEQUENCE 序列号的名称 (最好是表名 序列号标记) INCREMENT BY 1 START WITH 1 MAXVALUE 99999 CYCLE NOCACHE；其中最大的值按字段的长度来定, 如果定义的自动增长的序列号 NUMBER(6) , 最大值为999999 INSERT 语句插入这个字段值为: 序列号的名称.NEXTVAL

　　2. 单引号的处理 MySQL里可以用双引号包起字符串，Oracle里只可以用单引号包起字符串。在插入和修改字符串前必须做单引号的替换：把所有出现的一个单引号替换成两个单引号。

　　3. 翻页的SQL语句的处理 MySQL处理翻页的SQL语句比较简单，用LIMIT 开始位置, 记录个数；PHP里还可以用SEEK定位到结果集的位置。 Oracle处理翻页的SQL语句就比较繁琐了。每个结果集只有一个ROWNUM字段标明它的位置, 并且只能用ROWNUM＆lt;100, 不能用ROWNUM>80。 以下是经过分析后较好的两种Oracle翻页SQL语句( ID是唯一关键字的字段名 )：

　　语句一：

　　SELECT ID, [FIELD\_NAME,…] FROM TABLE\_NAME WHERE ID IN ( SELECT ID FROM (SELECT ROWNUM AS NUMROW, ID FROM TABLE_NAME WHERE 条件1 ORDER BY 条件2) WHERE NUMROW >80 AND NUMROW ＆lt; 100 ) ORDER BY 条件3;

　　语句二：

　　SELECT \* FROM (( SELECT ROWNUM AS NUMROW, c.\* from (select [FIELD\_NAME,…] FROM TABLE\_NAME WHERE 条件1 ORDER BY 条件2) c) WHERE NUMROW >80 AND NUMROW ＆lt; 100 ) ORDER BY 条件3;

　　4． 长字符串的处理 长字符串的处理Oracle也有它特殊的地方。INSERT和UPDATE时最大可操作的字符串长度小于等于 4000个单字节, 如果要插入更长的字符串, 请考虑字段用CLOB类型，方法借用Oracle里自带的DBMS_LOB程序包。插入修改记录前一定要做进行非空和长度判断，不能为空的字段值和超出长度字段值都应该提出警告, 返回上次操作。

　　5. 日期字段的处理 MySQL日期字段分DATE和TIME两种，Oracle日期字段只有DATE，包含年月日时分秒信息，用当前数据库的系统时间为SYSDATE, 精确到秒，或者用字符串转换成日期型函数TO\_DATE(‘2001-08-01’,’YYYY-MM-DD’) 年-月-日 24小时:分钟:秒 的格式YYYY-MM-DD HH24:MI:SS TO\_DATE()还有很多种日期格式, 可以参看 Oracle DOC. 日期型字段转换成字符串函数TO\_CHAR(‘2001-08-01’,’YYYY-MM-DD HH24:MI:SS’) 日期字段的数学运算公式有很大的不同。 MySQL找到离当前时间7天用 DATE\_FIELD\_NAME > SUBDATE（（NOW（），INTERVAL 7 DAY） Oracle找到离当前时间7天用 DATE\_FIELD_NAME >SYSDATE – 7；

　　6. 空字符的处理 MySQL的非空字段也有空的内容，Oracle里定义了非空字段就不容许有空的内容。 按MySQL的NOT NULL来定义Oracle表结构, 导数据的时候会产生错误。因此导数据时要对空字符进行判断，如果为NULL或空字符，需要把它改成一个空格的字符串。

　　7. 字符串的模糊比较 MySQL里用 字段名 like Oracle里也可以用 字段名 like 但这种方法不能使用索引, 速度不快用字符串比较函数 instr(字段名,字符串)>0 会得到更精确的查找结果

　　8. 程序和函数里，操作数据库的工作完成后请注意结果集和指针的释放。