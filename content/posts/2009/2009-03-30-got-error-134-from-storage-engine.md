---
title: Got error 134 from storage engine
author: admin
type: post
date: 2009-03-30T04:11:24+00:00
url: /archives/1171
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---

今天将原网站数据导入新系统的时候,发现用户表是空的,程序前几天很正常的,并没有做任何修改,于是将程序的高度模式打开,发现得到错误提示:”Got error 134 from storage engine”,进到mysql里执行select * from tbl_member limit 100,我没有发现错误的,不过将语句若修改为select * from tbl_member limit 100,10时,又出现了这个错误提示信息,怀疑是mysql表损坏,由于备份的时候,mysql处于运行使用状态,并没有停止服务的,所以才产生了这个错误的


于是用 **repair table tablename** 命令修复了次用户表,再次执行上述命令,ok,显示执行成功