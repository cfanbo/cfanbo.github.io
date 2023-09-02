---
title: ADODB中GetOne($sql)与GetRow($sql)的区别
author: admin
type: post
date: 2007-09-25T13:56:19+00:00
url: /archives/150
IM_contentdowned:
 - 1
categories:
 - 程序开发

---
**GetOne($sql)**Executes the SQL and returns the first field of the first row as an array. The recordset and remaining rows are discarded for you automatically. If an error occur, false is returned.        执行SQL指令，并且以阵列的方式回传第一笔记录的第一个栏位。资料集及其余的记录将会被自动清除，如果发生错误，就回传 false 值。译者注：这个功能在验证某笔记录在不在特别有用，可以减少系统记忆体及资源的用量。**GetRow($sql)**执行SQL指令，并且以阵列的方式回传第一笔记录。资料集及其馀的记录将会被自动清除，如果发生错误，就回传 false 值。其中GetOne($sql)为了检测某一条记录是否存在时,特别有用,(如,用户在注册前,可以检测用户名是否已经被占用,比较适合GetOne($ql)).如果此时需要除检测该记录是否存在,并保存该记录的信息,就要用到GetRow($sql)了,如用户登陆时,如果没有找到该用户的信息,则登陆失败,否则保存用户的注册信息到SESSION或COOIKE中,转到用户控制面板.