---
title: MongoDB非正常关闭后修复记录 mongod –repair
author: admin
type: post
date: 2012-04-27T05:32:25+00:00
url: /archives/12805
IM_contentdowned:
 - 1
categories:
 - nosql
tags:
 - MongoDB

---
mongod没有后台执行，在终端连接非正常断开后，再次执行mongod报错，如下所示：

[root@localhost mongodb]# ./bin/mongod
./bin/mongod –help for help and startup options
Thu Nov 17 22:42:49
Thu Nov 17 22:42:49 warning: 32-bit servers don’t have journaling enabled by default. Please use –journal if you want durability.
Thu Nov 17 22:42:49
Thu Nov 17 22:42:49 [initandlisten] MongoDB starting : pid=3257 port=27017 dbpath=/data/db/ 32-bit host=localhost
Thu Nov 17 22:42:49 [initandlisten]
Thu Nov 17 22:42:49 [initandlisten] ** NOTE: when using MongoDB 32 bit, you are limited to about 2 gigabytes of data
Thu Nov 17 22:42:49 [initandlisten] **       see http://blog.mongodb.org/post/137788967/32-bit-limitations
Thu Nov 17 22:42:49 [initandlisten] **       with –journal, the limit is lower
Thu Nov 17 22:42:49 [initandlisten]
Thu Nov 17 22:42:49 [initandlisten] db version v2.0.1, pdfile version 4.5
Thu Nov 17 22:42:49 [initandlisten] git version: 3a5cf0e2134a830d38d2d1aae7e88cac31bdd684
Thu Nov 17 22:42:49 [initandlisten] build info: Linux domU-12-31-39-01-70-B4 2.6.21.7-2.fc8xen #1 SMP Fri Feb 15 12:39:36 EST 2008 i686 BOOST\_LIB\_VERSION=1_41
Thu Nov 17 22:42:49 [initandlisten] options: {}
\***\***\***\*****
Unclean shutdown detected.
Please visit http://dochub.mongodb.org/core/repair for recovery instructions.
\***\***\***\****
Thu Nov 17 22:42:49 [initandlisten] **exception in initAndListen: 12596 old lock file, terminating**
Thu Nov 17 22:42:49 dbexit:
Thu Nov 17 22:42:49 [initandlisten] shutdown: going to close listening sockets…
Thu Nov 17 22:42:49 [initandlisten] shutdown: going to flush diaglog…
Thu Nov 17 22:42:49 [initandlisten] shutdown: going to close sockets…
Thu Nov 17 22:42:49 [initandlisten] shutdown: waiting for fs preallocator…
Thu Nov 17 22:42:49 [initandlisten] shutdown: closing all files…
Thu Nov 17 22:42:49 [initandlisten] closeAllFiles() finished
Thu Nov 17 22:42:49 dbexit: really exiting now



**修复方法：**

这算是一个Mongod 启动的一个常见错误，非法关闭的时候，lock 文件没有干掉，第二次启动的时候检查到有lock 文件的时候，就报这个错误了。

解决方法：进入 mongod 上一次启动的时候指定的 data 目录 ** ****–dbpath=/data/mongodb**删除掉该文件:

**rm /data/mongodb/mongo.lock**

再执行:

** ./mongod  –repair**

启动：

/usr/local/src/mongodb-linux-x86_64-2.0.2/bin/mongod –port=27017 –pidfilepath=/var/run/mongod.pid –dbpath=/data/mongodb –directoryperdb –nojournal –noauth

OK，问题解决。

正确关闭mongod 的方法：进入mongo shell

use admin

db.shutdownServer()

也可以按照文档粗暴的杀掉它，它内部应该有KILL信号处理程序。

killall mongod请不要 kill -9 ，会造成文件数据混乱丢失 repair 也无力回天。