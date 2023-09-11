---
title: centos 5.4 下面安装mongodb服务端和php客户端扩展…
author: admin
type: post
date: 2010-09-26T02:14:42+00:00
url: /archives/5820
IM_contentdowned:
 - 1
categories:
 - nosql
tags:
 - centos
 - MongoDB

---
**一、安装服务端**

1.下载MongoDB

我下载的是1.6的版本

2.解压并把解压包bin目录中的文件全部移到/usr/local/bin目录下面

3.运行mongo的服务器端程序

> /usr/local/bin/mongod –port <端口> –bind_ip <本机IP> –dbpath <数据库文件存放的位置>

如果要求开机就启动就在

/usr/local/etc/rc.local中加入下面一行内容

> usr/local/bin/mongod –port <端口> –bind_ip <本机IP> –dbpath <数据库文件存放的位置> >/dev/null &

4.测试安装是否成功

运行指令

> ＃/usr/local/bin/mongo –port 4312 –host 127.0.0.1

显示

> MongoDB shell version: 1.6.0
>
> connecting to: 127.0.0.1:4312/test

>help

db.help()                    help on db methods


db.mycoll.help()             help on collection methods


rs.help()                    help on replica set methods


help connect                 connecting to a db help


help admin                   administrative help


help misc                    misc things to know


show dbs                     show database names


show collections             show collections in current database


show users                   show users in current database


show profile                 show most recent system.profile entries with time >= 1ms


use                 set current database


db.foo.find()                list objects in collection foo


db.foo.find( { a : 1 } )     list objects in foo where a == 1


it                           result of the last line evaluated; use to further iterate


exit                         quit the mongo shell


下面的自己慢慢玩了。

**二、安装客户端**

1.去下载相关的扩展包

另外在github上面也有不少相关的资源

2.运行下面命令

pecl install http://pecl.php.net/get/mongo-.tgz

3.进入/etc/php.d

创建mongo.ini

添加下面的内容

extension=mongo.so

4.重启apache

5.写php脚本测试是否成功

#/usr/local/bin/mongo –port 4312 –host 127.0.0.1

>show dbs;

/\*选择数据库\*/

>use test;

/\*查看Collection\*/

>db.getCollectionNames() //显示结果［”system.indexes”, “system.users” ]

//创建一个新Collection

>db.createCollection(‘cartoons’);

>exit;

把 [http://cn2.php.net/manual/en/mongo.tutorial.php](http://cn2.php.net/manual/en/mongo.tutorial.php) 最下方的程序复制在一个文件test.php中

记得改连接的主机和端口

我的改动为改


$m = new Mongo();


为


$m = new Mongo(“127.0.0.1:4312”);


然后在网页上面浏览或是用命令行执行即可看到相关的结果


**三、其他**

详细的文档看

**后记：**

早听说这个数据表，今天拿来玩一下，至于数据的操作（添加，删除，修改，查询）、数据库的备分后续玩的过程中再和大家一起分享，相信入门很简单。

另外从官方来看他的第三方调用方式还是挺多的。居然还有支持javascript（框架），这点倒是很少用。值得玩转一下。