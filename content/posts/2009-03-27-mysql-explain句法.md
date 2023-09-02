---
title: MySQL EXPLAIN句法
author: admin
type: post
date: 2009-03-27T01:18:35+00:00
excerpt: |
 EXPLAIN tbl_name or EXPLAIN SELECT select_options

 EXPLAIN tbl_name是DESC[RIBE] tbl_name或SHOW COLUMNS FROM tbl_name的一个同义词。

 当你在一条SELECT语句前放上关键词EXPLAIN，MySQL解释它将如何处理SELECT，提供有关表如何联结和以什么次序联结的信息。

 借助于EXPLAIN，你可以知道
 1)你什么时候必须为表加入索引以得到一个使用索引找到记录的更快的SELECT。
 2)你也能知道优化器是否以一个最佳次序联结表。为了强制优化器对一个SELECT语句使用一个特定联结次序，增加一个STRAIGHT_JOIN子句。
url: /archives/1120
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - EXPLAIN
 - mysql

---
Explain虽然是大家常用的分析mysql优化的办法，但对于系统级别内容的消耗资源信息就无能为力了．这时需要用到Mysql中的Profiling(程序剖析) 功能．参考：

EXPLAIN tbl\_name or EXPLAIN SELECT select\_options

EXPLAIN tbl\_name是DESC[RIBE] tbl\_name或SHOW COLUMNS FROM tbl_name的一个同义词。

当你在一条SELECT语句前放上关键词EXPLAIN，MySQL解释它将如何处理SELECT，提供有关表如何联结和以什么次序联结的信息。

借助于EXPLAIN，你可以知道
1)你什么时候必须为表加入索引以得到一个使用索引找到记录的更快的SELECT。
2)你也能知道优化器是否以一个最佳次序联结表。为了强制优化器对一个SELECT语句使用一个特定联结次序，增加一个STRAIGHT_JOIN子句。

对于非简单的联结，EXPLAIN为用于SELECT语句中的每个表返回一行信息。表以他们将被读入的顺序被列出。
MySQL用一边扫描多次联结的方式解决所有联结，这意味着MySQL 1)从第一个表中读一行，2)然后找到在第二个表中的一个匹配行，3)然后在第3个表中等等。=>当所有的表被处理完，它输出选择的列并且回溯表列表直到找到一个表有更多的匹配行，从该表读入下一行并继续处理下一个表。

从EXPLAIN的输出包括下面列：

table输出的行所引用的表。

**type**
联结类型。各种类型的信息在下面给出。

**possible_keys**
possible_keys列指出MySQL能使用哪个索引在该表中找到行。
注意，该列完全独立于表的次序。这意味着在possible_keys中的某些键实际上不能以生成的表次序使用。
如果该列是空的，没有相关的索引。在这种情况下，你也许能通过检验WHERE子句看是否它引用某些列或列不是适合索引来提高你的查询性能。如果是这样，创造一个适当的索引并且在用EXPLAIN检查查询。见7.8 ALTER TABLE句法。为了看清一张表有什么索引，使用SHOW INDEX FROM tbl_name。 (可以查出，哪些索引基本上未用，或用的少)

**key**
key列显示MySQL实际决定使用的键。如果没有索引被选择，键是NULL。

**key_len**
key_len列显示MySQL决定使用的键长度。如果键是NULL，长度是NULL。注意这告诉我们MySQL将实际使用一个多部键值的几个部分。

**ref**
ref列显示哪个列或常数与key一起用于从表中选择行。

**rows**
rows列显示MySQL相信它必须检验以执行查询的行数。

**Extra**
如果是Only index，这意味着信息只用索引树中的信息检索出的。通常，这比扫描整个表要快。
如果是where used，它意味着一个WHERE子句将被用来限制哪些行与下一个表匹配或发向客户。
如果是impossible where 表示用不着where

**Type的取值常会有下面这些:**

**system**
表仅有一行(=系统表)。这是const联结类型的一个特例。

**const**
表有最多一个匹配行，它将在查询开始时被读取。因为仅有一行，在这行的列值可被剩下的优化器认为是常数。 const表很快，因为它们只读取一次！

**eq_ref**
对于每个来自于先前的表的行组合，从该表中读取一行。这可能是最好的联结类型，除了const类型。它用在一个索引的所有部分被联结使用并且索引是UNIQUE或PRIMARY KEY。

**ref**
对于每个来自于先前的表的行组合，所有有匹配索引值的行将从这张表中读取。
如果联结只使用键的最左面前缀，不或如果键不是UNIQUE或PRIMARY KEY（换句话说，如果联结能基于键值选择单个行的话)，使用ref。如果被使用的键仅仅匹配一些行，该联结类型是不错的。

**range
** 只有在一个给定范围的行将被检索，使用一个索引选择行。ref列显示哪个索引被使用。

**index**
这与ALL相同，除了只有索引树被扫描。这通常比ALL快，因为索引文件通常比数据文件小。

**ALL**
对于每个来自于先前的表的行组合，将要做一个完整的表扫描。
如果表格是第一个没标记const的表，这通常不好，并且通常在所有的其他情况下很差。你通常可以通过增加更多的索引来避免ALL，使得行能从早先的表中基于常数值或列值被检索出。

通过相乘EXPLAIN输出的rows行的所有值，你能得到一个关于一个联结要多好的提示。这应该粗略地告诉你MySQL必须检验多少行以执行查询。当你使用max\_join\_size变量限制查询时，也用这个数字。见10.2.3 调节服务器参数。

下列例子显示出一个JOIN如何能使用EXPLAIN提供的信息逐步被优化。

假定你有显示在下面的SELECT语句，你使用EXPLAIN检验：

EXPLAIN SELECT tt.TicketNumber, tt.TimeIn,
tt.ProjectReference, tt.EstimatedShipDate,
tt.ActualShipDate, tt.ClientID,
tt.ServiceCodes, tt.RepetitiveID,
tt.CurrentProcess, tt.CurrentDPPerson,
tt.RecordVolume, tt.DPPrinted, et.COUNTRY,
et_1.COUNTRY, do.CUSTNAME
FROM tt, et, et AS et_1, do
WHERE tt.SubmitTime IS NULL
AND tt.ActualPC = et.EMPLOYID
AND tt.AssignedPC = et_1.EMPLOYID
AND tt.ClientID = do.CUSTNMBR;

对于这个例子，假定：

被比较的列被声明如下： 表 列 列类型
tt ActualPC CHAR(10)
tt AssignedPC CHAR(10)
tt ClientID CHAR(10)
et EMPLOYID CHAR(15)
do CUSTNMBR CHAR(15)

表有显示在下面的索引： 表 索引
tt ActualPC
tt AssignedPC
tt ClientID
et EMPLOYID（主键）
do CUSTNMBR（主键）

tt.ActualPC值不是均匀分布的。
开始，在任何优化被施行前，EXPLAIN语句产生下列信息：

table type possible\_keys key key\_len ref rows Extra
et ALL PRIMARY NULL NULL NULL 74
do ALL PRIMARY NULL NULL NULL 2135
et_1 ALL PRIMARY NULL NULL NULL 74
tt ALL AssignedPC,ClientID,ActualPC NULL NULL NULL 3872
range checked for each record (key map: 35)

因为type对每张表是ALL，这个输出显示MySQL正在对所有表进行一个完整联结！这将花相当长的时间，因为必须检验每张表的行数的乘积次数！对于一个实例，这是74 \* 2135 \* 74 * 3872 = 45,268,558,720行。如果表更大，你只能想象它将花多长时间……

如果列声明不同，这里的一个问题是MySQL(还)不能高效地在列上使用索引。在本文中，VARCHAR和CHAR是相同的，除非他们声明为不同的长度。因为tt.ActualPC被声明为CHAR(10)并且et.EMPLOYID被声明为CHAR(15)，有一个长度失配。

为了修正在列长度上的不同，使用ALTER TABLE将ActualPC的长度从10个字符变为15个字符：

mysql> ALTER TABLE tt MODIFY ActualPC VARCHAR(15);

现在tt.ActualPC和et.EMPLOYID都是VARCHAR(15)，再执行EXPLAIN语句产生这个结果：

table type possible\_keys key key\_len ref rows Extra
tt ALL AssignedPC,ClientID,ActualPC NULL NULL NULL 3872 where used
do ALL PRIMARY NULL NULL NULL 2135
range checked for each record (key map: 1)
et_1 ALL PRIMARY NULL NULL NULL 74
range checked for each record (key map: 1)
et eq_ref PRIMARY PRIMARY 15 tt.ActualPC 1

这不是完美的，但是是好一些了(rows值的乘积少了一个74一个因子)，这个版本在几秒内执行。

第2种改变能消除tt.AssignedPC = et_1.EMPLOYID和tt.ClientID = do.CUSTNMBR比较的列的长度失配：

mysql> ALTER TABLE tt MODIFY AssignedPC VARCHAR(15),
MODIFY ClientID VARCHAR(15);

现在EXPLAIN产生的输出显示在下面：

table type possible\_keys key key\_len ref rows Extra
et ALL PRIMARY NULL NULL NULL 74
tt ref AssignedPC,ClientID,ActualPC ActualPC 15 et.EMPLOYID 52 where used
et\_1 eq\_ref PRIMARY PRIMARY 15 tt.AssignedPC 1
do eq_ref PRIMARY PRIMARY 15 tt.ClientID 1

这“几乎”象它能得到的一样好。

剩下的问题是，缺省地，MySQL假设在tt.ActualPC列的值是均匀分布的，并且对tt表不是这样。幸好，很容易告诉MySQL关于这些：

shell> myisamchk –analyze PATH\_TO\_MYSQL_DATABASE/tt
shell> mysqladmin refresh

现在联结是“完美”的了，而且EXPLAIN产生这个结果：

table type possible\_keys key key\_len ref rows Extra
tt ALL AssignedPC,ClientID,ActualPC NULL NULL NULL 3872 where used
et eq_ref PRIMARY PRIMARY 15 tt.ActualPC 1
et\_1 eq\_ref PRIMARY PRIMARY 15 tt.AssignedPC 1
do eq_ref PRIMARY PRIMARY 15 tt.ClientID 1

注意在从EXPLAIN输出的rows列是一个来自MySQL联结优化器的“教育猜测”；为了优化查询，你应该检查数字是否接近事实。如果不是，你可以通过在你的SELECT语句里面使用STRAIGHT_JOIN并且试着在在FROM子句以不同的次序列出表，可能得到更好的性能。