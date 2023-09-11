---
title: adodb教程:产生 Update 及 Insert 的SQL指令
author: admin
type: post
date: 2007-08-18T10:57:25+00:00
url: /archives/97
IM_contentdowned:
 - 1
categories:
 - 程序开发

---
[ADODB](/?tag=adodb) 1.31版起，新增了两个资料集函数：GetUpdateSQL()及GetInsertSQL()。这允许你在执行了像”Select * FROM table query Where…”这样的查询函数後，建立一个 $rs->fields复本，改变这些栏位，然後自动产生出更新或是新增的SQL指令。以下我们展示如何运用这些函数，我们将存取一个资料表，带有下列栏位：(ID,FirstName,LastName,Created)。在这些函数被执行前，你需要藉由一个对资料表的查询指令(select)来初始化一个资料集。 #==============================================

 #  GetUpdateSQL() 及 GetInsertSQL() 范例码

 #==============================================

 include(‘ADOdb.inc.php’);

 include(‘tohtml.inc.php’);#==========================

**# 以下的程式代码为测试新增状态**

$conn = &ADONewConnection(“mysql”);  # 建立一个连结
$conn->debug=1;
$conn->PConnect(“localhost”, “admin”, “”, “test”); # 连结到 MySQL, 资料库名称为 test

$sql = “Select * FROM ADOXYZ Where id = -1”;

 # 从资料库中查询出一个空的资料集

$rs = $conn->Execute($sql); # 执行查询，并取得一个空的资料集 **初始化记录集,以便下面保存,**

$record = array(); # 初始化一个阵列，以便存放记录资料供新增用

\# 设定记录中的栏位值
$record[“firstname”] = “Bob”;

 $record[“lastname”] = “Smith”;

 $record[“created”] = time();

\# 传入空的资料集及栏位资料阵列到GetInsertSQL函数中，以执行功能
\# 这个函数将会依传入的资料，回传一个全格式的 Insert SQL指令

$insertSQL = $conn->GetInsertSQL($rs, $record);$conn->Execute($insertSQL); # 将记录挿入资料库中

#==========================
\# **以下的程式码测试更新状态**

$sql = “Select * FROM ADOXYZ Where id = 1”;
\# 选择一笔记录以便更新

$rs = $conn->Execute($sql); # 执行这个查询，并取得一个存在的记录来更新

$record = array(); \# 初始化一个阵列，以存放要更新的资料

\# 设定栏位里的值
$record[“firstname”] = “Caroline”;$record[“lastname”] = “Smith”; # 更新 Caroline的姓由 Miranda 变成 Smith

\# 传入这个只有单一记录的资料集以及含有资料的阵列到 GetUpdateSQL函数里
\# 函数将会回传一个具有正确 Where 条件的 Update(更新) SQL 指令

$updateSQL = $conn->GetUpdateSQL($rs, $record);$conn->Execute($updateSQL); \# 更新资料库中的记录
$conn->Close();
?>