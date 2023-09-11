---
title: memcached 集群单点故障解决方案
author: admin
type: post
date: 2011-11-29T01:55:17+00:00
url: /archives/12145
IM_contentdowned:
 - 1
categories:
 - nosql
tags:
 - 集群
 - memcached

---
magent是一款开源的Memcached代理服务器软件，其项目网址为： [http://code.google.com/p/memagent/](http://code.google.com/p/memagent/)

**一、安装步骤：**
1、编译安装libevent：

```
wget http://monkey.org/~provos/libevent-1.4.9-stable.tar.gz
tar zxvf libevent-1.4.9-stable.tar.gz
cd libevent-1.4.9-stable/
./configure --prefix=/usr
make && make install
cd ../
```

2、编译安装Memcached：

```
wget http://danga.com/memcached/dist/memcached-1.2.6.tar.gz
tar zxvf memcached-1.2.6.tar.gz
cd memcached-1.2.6/
./configure --with-libevent=/usr
make && make install
cd ../
```

3、编译安装magent：

```
mkdir magent
cd magent/
wget http://memagent.googlecode.com/files/magent-0.6.tar.gz
tar zxvf magent-0.6.tar.gz
make
cp magent /usr/bin/magent
cd ../
```

magent的使用方法请参考:,安装magent的make过程中可能会出现错误信息,解决办法请参考:

**二、使用实例：**

```
memcached -m 1 -u root -d -l 127.0.0.1 -p 11211
memcached -m 1 -u root -d -l 127.0.0.1 -p 11212
memcached -m 1 -u root -d -l 127.0.0.1 -p 11213
magent -u root -n 51200 -l 127.0.0.1 -p 12000 -s 127.0.0.1:11211 -s 127.0.0.1:11212 -b 127.0.0.1:11213
```

```
-u  root：以root用户启动
 -n  51200：并发数51200
 -l 127.0.0.1：监听的IP是127.0.0.1
 -p 12000：端口是 12000
 -s 127.0.0.1：127.0.0.1是正在运行的memcached,端口分别为11211和11212,生产中一般为两台物理机器
 -b 127.0.0.1：127.0.0.1是备份的memcached,端口为11213.生产中为物理机器
```

1、分别在11211、11212、11213端口启动3个Memcached进程，在12000端口开启magent代理程序；
2、11211、11212端口为主Memcached，11213端口为备份Memcached；
3、连接上12000的magent，set key1和set key2，根据哈希算法，key1被写入11212和11213端口的Memcached，key2被写入11212和11213端口的Memcached；
4、当11211、11212端口的Memcached死掉，连接到12000端口的magent取数据，数据会从11213端口的Memcached取出；
5、当11211、11212端口的Memcached重启复活，连接到12000端口，magent会从11211或11212端口的Memcached取数据，由于这两台Memcached重启后无数据，因此magent取得的将是空值，尽管11213端口的Memcached还有数据（此问题尚待改进）。

**三、整个测试流程：**

```
[root@centos52 ~]# telnet 127.0.0.1 12000
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
stats
memcached agent v0.4
matrix 1 -> 127.0.0.1:11211, pool size 0
matrix 2 -> 127.0.0.1:11212, pool size 0
END
set key1 0 0 8
zhangyan
STORED
set key2 0 0 8
zhangyan
STORED
quit
Connection closed by foreign host.

[root@centos52 ~]# telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
get key1
END
get key2
VALUE key2 0 8
zhangyan
END
quit
Connection closed by foreign host.

[root@centos52 ~]# telnet 127.0.0.1 11212
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
get key1
VALUE key1 0 8
zhangyan
END
get key2
END
quit
Connection closed by foreign host.

[root@centos52 ~]# telnet 127.0.0.1 11213
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
get key1
VALUE key1 0 8
zhangyan
END
get key2
VALUE key2 0 8
zhangyan
END
quit
Connection closed by foreign host.
```

可以看到,备份11213这个存储了两个key的值.前面的是根据算法自动存储的.

模拟11211、11212端口的Memcached死掉

```
[root@centos52 ~]# ps -ef | grep memcached
root       6589     1   0 01:25 ?         00:00:00 memcached -m 1 -u root -d -l 127.0.0.1 -p 11211
root       6591     1   0 01:25 ?         00:00:00 memcached -m 1 -u root -d -l 127.0.0.1 -p 11212
root       6593     1   0 01:25 ?         00:00:00 memcached -m 1 -u root -d -l 127.0.0.1 -p 11213
root       6609   6509   0 01:44 pts/0     00:00:00 grep memcached
[root@centos52 ~]# kill -9 6589
[root@centos52 ~]# kill -9 6591
[root@centos52 ~]# telnet 127.0.0.1 12000
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
get key1
VALUE key1 0 8
zhangyan
END
get key2
VALUE key2 0 8
zhangyan
END
quit
Connection closed by foreign host.
```

模拟11211、11212端口的Memcached重启复活

```
[root@centos52 ~]# memcached -m 1 -u root -d -l 127.0.0.1 -p 11211
[root@centos52 ~]# memcached -m 1 -u root -d -l 127.0.0.1 -p 11212
[root@centos52 ~]# telnet 127.0.0.1 12000
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
get key1
END
get key2
END
quit
Connection closed by foreign host.
```

重新11211和11212两个实例后,发现里面的存储的内容已经全部没有了(因为数据存储在内存里),现在又可以重新提供memcached服务了.

**相关教程:**

[Linux下安装Memcached](http://blog.haohtml.com/archives/395)