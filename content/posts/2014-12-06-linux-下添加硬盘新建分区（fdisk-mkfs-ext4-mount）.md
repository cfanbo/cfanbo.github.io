---
title: Linux 下添加硬盘/新建分区（fdisk + mkfs.ext4 + mount）
author: admin
type: post
date: 2014-12-06T07:24:16+00:00
url: /archives/15379
categories:
 - 服务器
tags:
 - fdisk

---
此教程只供参考，未进行整理！

使用fdisk命令查看新添加的硬盘

[![QQ截图20141206152231](http://blog.haohtml.com/wp-content/uploads/2014/12/QQ截图20141206152231.jpg)][1]



会看到类似这种页面信息的(说明：这里的图为已经有两个硬盘在使用了， 新添加的硬盘为sdc,尚未使用)

- [第一步：添加硬盘/新建分区（fdisk）](http://www.fikker.com/bigcache2/help/linux-fdisk.html#e1)
- [第二步：格式化分区（mkfs.ext4）](http://www.fikker.com/bigcache2/help/linux-fdisk.html#e2)
- [第三步：加载分区（mount）](http://www.fikker.com/bigcache2/help/linux-fdisk.html#e3)

1、第一步：添加硬盘/新建分区（fdisk）

a、查看当前系统所有硬盘及分区情况：fdisk -l

b、在指定的硬盘（例：/dev/sdb）上创建分区：fdisk /dev/sdb ， 根据提示进行下一步操作，如：查看帮助（h），新建分区（n），删除分区（d），查看分区情况（p）

c、分区成功后，写分区表并退出（w）

注：fdisk 支持硬盘最大尺寸为 2TB，更详细说明请参看 Linux 在线手册（man fdisk）或百度一下。

2、第二步：格式化分区（mkfs.ext4）

对新建分区（例：/dev/sdb1）进行格式化：mkfs.ext4 /dev/sdb1 。

3、第三步：加载分区

a、创建分区挂接目录，例：mkdir /disk-cache-1 和 mkdir /disk-cache-2

b、编辑 /etc/fstab 配置文件，将分区信息写进去。

添加：/dev/sdb1 /fikker ext3 defaults 0 0

[![fstab-fikker-2](http://blog.haohtml.com/wp-content/uploads/2014/12/fstab-fikker-2.png)](http://blog.haohtml.com/wp-content/uploads/2014/12/fstab-fikker-2.png)

c、加载新建分区：mount -a

参考： [http://soft.chinabyte.com/461/7749961.shtml](http://soft.chinabyte.com/461/7749961.shtml)

 [1]: http://blog.haohtml.com/wp-content/uploads/2014/12/QQ截图20141206152231.jpg