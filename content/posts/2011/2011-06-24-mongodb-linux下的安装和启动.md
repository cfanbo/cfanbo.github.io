---
title: MongoDB Linux下的安装和启动
author: admin
type: post
date: 2011-06-24T06:18:00+00:00
url: /archives/9969
IM_contentdowned:
 - 1
categories:
 - nosql
tags:
 - MongoDB

---
这里用的是64位版本.使用时请检查相应操作系统的版本是32位还是64位.

**1>设置mongoDB目录**

> cd /home/apps
> mkdir /home/apps

**2>下载mongodb**

> wget http://fastdl.mongodb.org/linux/mongodb-linux-x86_64-1.6.3.tgz

**3>解压缩文件**

> tar xzf mongodb-linux-x86_64-1.6.3.tgz

**4>启动服务**

> ./mongodb-linux-x86_64-1.6.3/bin/mongod -dbpath=/data/mongodb/db -logpath=/data/mongodb/log

**5>将mongoDB服务加入随机启动**

> vi /etc/rc.local

使用vi编辑器打开配置文件，并在其中加入下面一行代码

> /home/apps/mongodb/bin/mongod –dbpath /data/mongodb/db –port 27017 –logpath /data/mongodb/log –logappend &

**6>连接mongoDB**

> ./mongodb-linux-x86_64-1.6.3/bin/mongo DBName

讲到这儿，mongoDB的在Linux下安装已完成，本地连接mongoDB也已成功，这时我们就要考虑到另外一个问题了，局域网如何来连接mongoDB呢？局域网中windows机器如何来连接Linux机器中的mongoDB呢？

其实做法一样很简单：./mongodb-linux-x86_64-1.6.3/bin/mongo 192.168.10.234/DBName 即可。

不过此处就需要注意了，我们需要在centOS上打开mongoDB的端口号，接下来讲讲如何在centOS上打开指定端口。

我们打开配置文件 /etc/sysconfig/iptables，在该文件中添加如下内容：

> -A RH-Firewall-l-INPUT -P tcp -m tcp –dport mongoDB端口号 -j ACCEPT

然后重启服务

> service iptables restart

此时，你已可以开始通过局域网来访问centOS上部署的mongoDB

**7>测试Mongodb**

在控制台中：


> $ nohup ./mongodb-xxxxxxx/bin/mongod &
>
> $ ./mongodb-xxxxxxx/bin/mongo
>
> > db.foo.save( { a : 1 } )
>
> > db.foo.find()

结果是：


> { “_id” : ObjectId(“4cd181a31415ffb41a094f43”), “a” : 1 }

顺便再增加一点centOS与windows互访的知识，譬如，我们想把原来在windows机器上的mongoDB产生的文件移植到centOS中，当然可以用移动存储设备来拷贝，但是我这里讲的是Linux（centOS）如何来访问windows共享目录，命令如下：

> mount -t cifs //ip/共享目录名称 /mnt/sharefile -o username=,password=

上面的命令即将windows的共享目录映射为linux上的/mnt/sharefile目录