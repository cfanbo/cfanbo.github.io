---
title: 'nagios出现NRPE: Unable to read output问题的解决办法'
author: admin
type: post
date: 2011-08-08T07:03:59+00:00
url: /archives/10929
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - nagios

---
在服务器部署好nagios，分别在客户端安装好，但是其中几台系统不是自己安装，里面的环境不太了解。

在nagios服务器端使用nrpe检查出现

### NRPE: Unable to read output

在监控机上运行check_nrpe -H IP

可以查看到客户端的nrpe信息，说明监控机与被监控机的nrpedaemon通信是正常。

在网上查找了一下，也没个具体的原因等。

**根据问题查找得出一些分析的注意地方：**

1、检查客户端nrpe的权限是否可读，可被nagios执行。

2、检查nrpe.cfg里面commands命令路径是否正确。

**常见的一些nrpe的错误信息解决方法：**

在监控机上，执行：

#[root@localhost][1] libexec]# /opt/nagios/libexec/check_nrpe -H IP

CHECK_NRPE: Error – Could not complete SSL handshake.

**解决方案：**

在被监控机nrpe.cfg中，增加监控主机的地址：

#NOTE: This option is ignored if NRPE is running under either inetd or xinetd

**allowed_hosts=127.0.0.1,IP**

注意两个地址以逗号隔开。并关闭超级守护进程xinetd.

 [1]: mailto:#root@localhost