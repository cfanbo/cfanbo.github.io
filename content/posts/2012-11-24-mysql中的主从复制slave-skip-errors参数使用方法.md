---
title: mysql中的主从复制slave-skip-errors参数使用方法
author: admin
type: post
date: 2012-11-24T14:38:28+00:00
url: /archives/13513
categories:
 - MySQL
tags:
 - 主从复制

---
在主从复制中，难免会遇到一些sql语句错误的问题。这个时候我们需要手动来重新设置中继日志的起始点了，有些麻烦。今天在看“2012华东架构师大会”视频的时候，发现淘宝丁奇的ppt里有这个参数,看名字就知道是让从库跳过一些错误了。以前没有注意过这个参数，这里了解一下这个参数的用法。

—————————————-

环境说明：

mysql>show  slave stsatus\G;

报错信息如下：

……

Last_Errno: 1062

Last_Error: Error ‘Duplicate entry ‘1’ for key ‘PRIMARY” on query…….

……

1062的错误是指一些主键重复的错误，在my.cnf用slave-skip-erros　可以跳过去。这样就避免了由于sql出错导致的从复制失效。

—————————————-

Error\_code: 1032; handler error HA\_ERR\_KEY\_NOT_FOUND;

造成1032错误的根本原因是主从数据库数据不一致,导致同步操作在从库上无法执行.解决办法同上

```
在slave的my.cnf里面写入
slave-skip-errors = 1062
启动后它将会忽略所有类型为1062的错误.
```