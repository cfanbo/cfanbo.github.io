---
title: Smarty中in_array函数的应用
author: admin
type: post
date: 2007-11-20T12:20:30+00:00
url: /archives/203
IM_contentdowned:
 - 1
categories:
 - 程序开发

---
php脚本:

$Action= array(“article”,”soft”,”news”);


$smarty = new Smarty();


$smarty->assign(“Action”,$Action);


模板:

在数组内非数组内的值