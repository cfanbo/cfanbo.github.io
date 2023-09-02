---
title: centos7无ifconfig命令的解决办法
author: admin
type: post
date: 2015-03-04T15:58:02+00:00
url: /archives/15454
categories:
 - 服务器
tags:
 - ifconfig

---
最小化安装了centos7。默认情况下是没有ifconfig这个命令的，可以使用 [ip](blog.haohtml.com/tag/ip) 命令代替ifconfig，使用“**ip addr**”和“**ip link**”命令来查找网卡详情。不过也可以手动安装此命令。

执行以下命令即可。

[shell]yum -y install net-tools[/shell]

[英文] [https://www.unixmen.com/ifconfig-command-found-centos-7-minimal-installation-quick-tip-fix/](https://www.unixmen.com/ifconfig-command-found-centos-7-minimal-installation-quick-tip-fix/) [http://linux.cn/article-3631-1.html](http://linux.cn/article-3631-1.html)