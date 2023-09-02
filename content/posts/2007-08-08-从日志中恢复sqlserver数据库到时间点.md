---
title: 从日志中恢复SQLServer数据库到时间点
author: admin
type: post
date: 2007-08-08T16:06:42+00:00
url: /archives/82
IM_data:
 - 'a:1:{s:77:"http://www.mx68.com/WebDeveloper/UploadFiles_5122/200603/2006310142248361.jpg";s:76:"http://blog.haohtml.com/wp-content/uploads/2009/06/4f43_2006310142248361.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 数据库

---

      DB2中可以使得数据库回复到指定的时间点，SQL Server数据库的Recovery Model为full 或者Bulk copy的时候，是可以从日志来恢复数据库的。实际上日志中记录的一条一条的transact sql语句，恢复数据库的时候会redo这些sql语句。

前提条件：myBBS是数据库test中的一个表,

          数据库test的Recovery Model为Full,Auto Close,Auto Shrink两个选项未选中。

          数据库test的data files和log files均为默认的自动增长状态。

![](http://www.mx68.com/WebDeveloper/UploadFiles_5122/200603/2006310142248361.jpg)

A：2004/10/13,16:00进行数据库备份，backup database test to disk=’d:\db\1600.bak’ with init

B：2004/10/14,13:00对数据库进行了update,delete等操作；

C：2004/10/15,18:00使用delete mybbs where id>300时，语句误写成delete mybbs,因而删除了表mybbs中的所有数据。

现在在C点,C点对数据库进行了误操作，我们希望数据库能够恢复到C之前的状态，比如恢复到10月15日17:59分的状态。

要恢复数据库B点，使用的是A点备分的数据库1600.bak；而使用的日志备分是最新的备分1820.logs；因而进行如下操作：

–备分日志：BACKUP LOG test TO DISK=’d:\1820.logs’ WITH INIT

–恢复数据库1600.bak,使用WITH NORECOVERY参数：RESTORE DATABASE test from disk=’d:\db\1640.bak’ WITH NORECOVERY

–使用日志恢复数据库到10月15日17:59分：

RESTORE LOG test

        FROM disk=’d:\1820.logs’ WITH RECOVERY,STOPAT=’10/15/2004 17:59′

上面的三条Transact SQL语句的对应过程:

      1.恢复数据库到A点；

      2.执行A-B之间的log记录，把数据库恢复到B点.

       这样就恢复数据库到了指定的时间点。如果恢复不成功，可能的原因是：1.未使用正确的备分数据库；2.数据库选项选中了Auto Shrink.