---
title: CentOS访问Windows共享文件夹的两种方法
author: admin
type: post
date: 2011-06-09T09:27:13+00:00
url: /archives/9743
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos

---
**1 在地址栏中输入下面内容:**

> smb://Windows IP/Share folder name

smb为Server Message Block协议的简称，是一种IBM协议，运行在TCP/IP协议之上。

从Windows 95开始，Microsoft Windows都提供了Server和Client的SMB协议支持，Microsoft为Internet提供了SMB开源版本，及CIFS(Common Internet File System)，通用文件系统。

**2 将Windows的共享文件夹挂载到本地**

在终端中输入命令：

> mount -t cifs -o username=”Admin”,password=”” //192.168.1.1/ShareFolder /mnt/MyShare

注意命令行中的空格和逗号，空密码也可以。

此命令就是将192.168.1.1上的共享文件夹ShareFolder 挂载到本地的/mnt/MyShare文件夹，执行完，就可在MyShare里看到ShareFolder里的内容。

删除挂载用命令：umount /mnt/MyShare

摘自：