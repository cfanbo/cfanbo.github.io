---
title: Percona XtraBackup备份mysql数据库 技术手册
author: admin
type: post
date: 2016-11-30T03:58:59+00:00
url: /archives/17265
categories:
 - MySQL
tags:
 - Xtrabackup

---
[https://www.percona.com/doc/percona-xtrabackup/2.4/index.html](https://www.percona.com/doc/percona-xtrabackup/2.4/index.html)

下载地址： [https://www.percona.com/downloads/XtraBackup/LATEST/](https://www.percona.com/downloads/XtraBackup/LATEST/)
一、安装两个必需库

```
sudo yum -y install libdv perl-DBD-MySQL

```

如果libev库在yum源找不到的话，需要在rpmfind.net网站下载自行安装。

```
wget ftp://rpmfind.net/linux/dag/redhat/el6/en/x86_64/dag/RPMS/libev-4.15-1.el6.rf.x86_64.rpm
rpm libev-4.15-1.el6.rf.x86_64.rpm

```

二、安装percona-xtrabackup

```
wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.5/binary/redhat/6/x86_64/percona-xtrabackup-24-2.4.5-1.el6.x86_64.rpm
rpm -ivh percona-xtrabackup-24-2.4.5-1.el6.x86_64.rpm

```

[https://yq.aliyun.com/articles/59272](https://yq.aliyun.com/articles/59272) [https://my.oschina.net/lionel45/blog/691406](https://my.oschina.net/lionel45/blog/691406) [http://blog.itpub.net/29418060/viewspace-1676617/](http://blog.itpub.net/29418060/viewspace-1676617/)