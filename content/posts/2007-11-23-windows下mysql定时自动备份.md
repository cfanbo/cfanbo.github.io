---
title: windows下mysql定时自动备份
author: admin
type: post
date: 2007-11-23T14:48:30+00:00
url: /archives/209
IM_contentdowned:
 - 1
categories:
 - 数据库

---
对于linux或unix下备份mysql可以说很简介的,但在windows下备份,好像只有用windows自带的计划任务了.

==============
假想环境：
MySQL 安装位置：C:MySQL
论坛数据库名称为：bbs
MySQL root 密码：123456
数据库备份目的地：D:db_backup

程序代码

@echo off
C:MySQLbinmysqladmin -u root –password=123456 shutdown
C:MySQLbinmysqldump –opt -u root –password=123456 bbs > D:db_backupbbs.sql
C:MySQLbinmysqld-nt

将以上代码保存为backup_db.bat
然后使用Windows的“计划任务”定时执行该脚本即可。（例如：每天凌晨5点执行back_db.bat）