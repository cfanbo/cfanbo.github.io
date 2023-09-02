---
title: ubuntu开启SSH服务
author: admin
type: post
date: 2010-07-08T03:22:26+00:00
url: /archives/4505
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux
 - ssh
 - Ubuntu

---
网上有很多介绍在Ubuntu下开启SSH服务的文章，但大多数介绍的方法测试后都不太理想，均不能实现远程登录到Ubuntu上，最后分析原因是都没有真正开启ssh-server服务。最终成功的方法如下：

**sudo apt-get install openssh-server**

Ubuntu缺省安装了openssh-client,所以在这里就不安装了，如果你的系统没有安装的话，再用apt-get安装上即可。

然后确认sshserver是否启动了：

**ps -e |grep ssh**

如果只有ssh-agent那ssh-server还没有启动，需要/etc/init.d/ssh start，如果看到sshd那说明ssh-server已经启动了。

ssh-server配置文件位于/ etc/ssh/sshd_config，在这里可以定义SSH的服务端口，默认端口是22，你可以自己定义成其他端口号，如222。然后重启SSH服务：

**sudo /etc/init.d/ssh resar**

ssh连接：ssh xjtu129@202.117.15.165