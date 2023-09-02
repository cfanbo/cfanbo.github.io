---
title: ServU和ServU-Plus结合对ftp用户进行数据库(Mysql)验证
author: admin
type: post
date: 2009-09-14T03:24:31+00:00
excerpt: |
 基本步骤：
 1.
 下载Mysql for windows的版本，目前最新的为mysql-4.0.20d-win。下载并安装启动。
 2. 在mysql.com网站下载对应的mysql-odbc驱动程序，安装在windows 2000/NT/advance server操作系统.
 3. 在操作系统中，点击控制面板－＞管理工具－＞数据源（ODBC），添加对MySQL ODBC的支持。
url: /archives/2367
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ftp
 - mysql
 - php

---
基本步骤：
1.

下载Mysql for windows的版本，目前最新的为mysql-4.0.20d-win。下载并安装启动。

2.      在mysql.com网站下载对应的mysql-odbc驱动程序，安装在windows 2000/NT/advance server操作系统.

3.      在操作系统中，点击控制面板－＞管理工具－＞数据源（ODBC），添加对MySQL ODBC的支持。

4.      使用servU-Plus插件程序，ServUPlus是Serv-U的一个插件，其主要功能就是捕捉Serv-U的事件，然后做适当的功能增强、扩展。解压后出现目录结构如下：

[\]根目录

Readme.txt      自述文件

MySQL_SQL.txt      MySQL的数据结构

MSSQL_SQL.txt      MSSQL的数据结构

Update.txt      升级说明

[\ServU]目录

dbexpmysql.dll            访问MySQL的DLL(可选)

dbexpmss.dll            访问MSSQL的DLL(可选)

libmySQL.dll            数据库接口DLL

MIDAS.DLL            数据库接口DLL

ServUPlus.dll            扩充功能库

ServUPlus.ini            配置文件

ServUPlus_Man.exe      管理主程序

5.      安装条件

1)      理论ServU 3.1以上，建议ServU 4.1.0.0或以上（因为这个版本修正了对DLL的支持，以及很多BUG）

2) ServU上面安装MySQL

6.      建立数据库

建立数据库的SQL语句:

SQL: MySQL_SQL.txt

MSSQL: MSSQL_SQL.txt

7.      修改配置文件(ServUPlus.ini)

[DataServer]            //[数据库部分]

Type=1                  //数据库类型1：MySQL，2：MS SQL Server

Host=127.0.0.1            //IP

User=root            //用户

Pass=                  //密码

Database=ServUPlus      //数据库

AutoRetry=1            //是(1)否(0)自动尝试连接

RetryTime=60            //尝试连接的间隔时间(秒)

[Option]            //[其他]

User_Cache=60            //缓存时间(秒)

NameAddStr=sisha_      //用户名前面增加的标识(暂时无用)

RatiosType=1            //(0)为下载完毕才扣下载量，(1)为按照实际下载量扣。(推荐使用1)

LockDomain=0            //是(1)否(0)锁定域(4.1.0.0以下不能使用)


[FilterCommand]            //[过滤命令]

ListR=0                  //是(1)否(0)过滤LIST -alR命令


[IPRule]            //[IP限制规则]

Max=3                  //用户自定义IP允许的个数，-1为不必输入，0为不限制个数

Depth=2                  //规则的位数，0为不限制


[Log]                  //[Log记录]

DL_OK=1                  //是(1)否(0)记录下载文件成功

DL_ERR=1            //是(1)否(0)记录下载文件失败

UL_OK=1                  //是(1)否(0)记录上传文件成功

UL_ERR=1            //是(1)否(0)记录上传文件失败


[SFVCheck]            //[SFV校验]

SFVEnable=1            //是(1)否(0)激活SFV检测

DelOtherMsg=1            //是(1)否(0)删除空文件(文件名为-*-)

HideTmpFile=1            //是(1)否(0)隐藏临时文件

AddMsg=sisha            //这个就是你加入的标识，随便起一个即可，比如起名叫sisha，然后你上传SFV后会显示：-[#####—–.50%]-[5.of.10]-[ServUPlus.******]-[sisha]-，就在方括号内

MsgUpFile=1            //是(1)否(0)标识上传中的文件

SkipUpFile=1            //是(1)否(0)跳过检测上传中的文件

SkipCompleteSFV=1      //是(1)否(0)跳过已经检测过的SFV文件。

SkipFileMax=10            //如果被检测的文件大于10 byte，则跳过检测，0为不限制。(建议用)

MsgSkipFile=1            //是(1)否(0)标识跳过检测的文件(限制了文件大小才显示)

LimitCheckPath=1      //是(1)否(0)限制要检测的目录(限制了，就只会检测以下的目录)

CheckPath1=E:\

CheckPath2=F:\            //这个是要检查的目录(包括其子目录)，也就是其他用户可以上载的目录，如有多个目录要检查…用CheckPath3=XXX…CheckPath4=XXXX如此类推


[Msgs]                  //[自定义信息]

GroupTooMany=Your group is too many users – please try again later.

AccountTooMany=Too many users – please try again later.

AccountExpired=Your account expired.

8.      安装扩充功能库(ServUPlus.dll)

以 Serv-U 4.1.0.0 为例：

1) 关闭 Serv-U（单击停止服务器 -> 立即停止）。

2) 将ServU目录下面的5个文件及ServUPlus.ini文件放在上 Serv-U 的安装目录下（不能放在其它目录）。

3) 修改 ServUDaemon.ini，添加以下设置（Serv-U 在启动时自动调用）：

[EXTERNAL]

ClientCheckDLL1=ServUPlus.dll

EventHookDLL1=ServUPlus.dll

9.      重新启动 Serv-U(单击开始服务器)，如果安装成功，您会在看到如下信息，表示 ServUPlus.dll 已成功加载。

Mon 15Jul02 12:48:45 – Serv-U FTP Server v4.0 (4.1.0.0) – Copyright (c) 1995-2002 Cat Soft, All Rights Reserved – by Rob Beckers

Mon 15Jul02 12:48:45 – Cat Soft is an affiliate of Rhino Software, Inc.

Mon 15Jul02 12:48:46 – Loaded external DLL ServUPlus.dll

…………………..

Mon 15Jul02 12:48:50 – Valid registration key found

Mon 15Jul02 12:48:50 – Loaded external DLL ServUPlus.dll

【Loaded external DLL ServUPlus.dll】有了这两行才说明安装成功

最好也要查看一下当前目录下的ServUPlus_LOG，如果出现：

2004-7-16 10:14:46 – Serv-U正在启动…

2004-7-16 10:14:46 – 成功载入ServUPlus.INI！

2004-7-16 10:14:46 – 成功连接数据库，插件启动正常！

说明servU-Plus已经正常加载了配置文件

10.      使用方法，以两组用户为例：

一组具有完全权限：包括目录建立，删除，更名；文件的上传，下载，删除，更名操作

另一组只有下载的权限

在servU服务器中，建立对应的两个真实用户，分别为upload,download,建立后可以禁用此两个用户，但是帐户必须存在。（做为虚拟用户映射所用）


11.      打开servu-Plus管理程序，界面如下，添加虚拟组

图一

图二

12.      完成后就可以用FTP客户端软件进行连接和控制。