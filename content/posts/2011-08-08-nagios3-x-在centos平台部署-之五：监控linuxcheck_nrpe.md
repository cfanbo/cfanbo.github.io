---
title: Nagios监控客户端CentOS设置(check_nrpe)
author: admin
type: post
date: 2011-08-08T06:29:59+00:00
url: /archives/10920
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - nagios

---
上一节我们在FreeBSD上实现了了安装nagios(),下面我们要监控一台linux(centos)的主机.

Note: It is possible to execute Nagios plugins on remote Linux/Unix machines through SSH. There is a
check\_by\_ssh plugin that allows you to do this. Using SSH is more secure than the NRPE addon, but it also imposes a larger (CPU) overhead on both the monitoring and remote machines. This can become an issue when you start monitoring hundreds or thousands of machines. Many Nagios admins opt for using using the NRPE addon because of the lower load it imposes.
**一、实现原理**
The NRPE addon is designed to allow you to execute Nagios plugins on remote Linux/Unix machines. The main reason for doing this is to allow Nagios to monitor “local” resources (like CPU load, memory usage, etc.) on remote machines. Since these public resources are not usually exposed to external machines, an agent like NRPE must be installed on the remote Linux/Unix machines.

[![](http://blog.haohtml.com/wp-content/uploads/2011/08/nagios-nrpe.jpg)](http://blog.haohtml.com/wp-content/uploads/2011/08/nagios-nrpe.jpg)

a、Nagios 执行 check_nrpe 插件，check_nrpe 去检测services 定义的服务。

b、通过 SSL，check_nrpe 连接被监控主机的 NRPE daemon

c、NRPE 运行本地的各种插件去检测本地和远端的服务和状态(check_disk,..etc)

d、最后，NRPE 把检测的结果传给主机端的 check_nrpe，check_nrpe 再把结果送到 Nagios状态队列中。

e、Nagios 依次读取队列中的信息，再把结果显示出来。

**二、被监控主机安装Nrpe**

**1、create account information**

[root@ypd ~]# adduser nagios


**2、compile and install the plugins**

> wget http://mcshell.org/nagios-plugins-1.4.13.tar.gz
>
> ./configure
>
> make
>
> make install
>
> chown nagios.nagios /usr/local/nagios
>
> chown -R nagios.nagios /usr/local/nagios/libexec

**3、Install the NRPE daemon**

> wget [http://mcshell.org/nrpe-2.8.tar.gz](http://mcshell.org/nrpe-2.8.tar.gz)
>
> tar xzf nrpe-2.8.tar.gz
>
> cd nrpe-2.8
>
>
> ./configure
>
> make all

3.2 Install the NRPE plugin (for testing), daemon, and sample daemon config file.

> make install-plugin
>
> make install-daemon
>
> make install-daemon-config

3.3 Install the NRPE daemon as a service under xinetd.


修改nrpe.cfg文件,修改allowed_hosts段如下


> **allowed_hosts=127.0.0.1,IP**

其中IP为监控主机的ip地址.


4 **、启用NRPE服务**

> /usr/local/nagios/bin/nrpe -c ../etc/nrpe.cfg -d
>
>
> 查看端口是否启用监听
>
>
> netstat -an | grep 5666

要注意监控端和被监控端的nrpe.cfg的commands段要一样，否则可能会＂NRPE: Command ‘check_disk’ not defined＂之类的错误,也有可能在nagios后台里提示＂host status information not found＂，无法发现要监控的机器．


**四、监控主机测试**

这时在监控主机端执行命令


#/usr/local/nagios/libexec/check_nrpe -H 监控机器的IP地址 -c check_load


OK – load average: 0.35, 0.37, 0.41|load1=0.350;15.000;30.000;0; load5=0.370;10.000;25.000;0; load15=0.410;5.000;20.000;0;


如果显示”NRPE: Unable to read output”之类的错误的请,解决办法见: [http://blog.haohtml.com/archives/10929](http://blog.haohtml.com/archives/10929),也有可能出现上面提示的错误信息的．


注意:

1.他们之间是通过端口 5666 进行的通讯,所以有问题请先检查两端机器的端口是否在监听,然后再进行其它错误的排除.

2.在安装的时候要注意安装的路径,查看nrpe.cfg文件末尾的command[check_XXX]段的命令,确认监控端和受控端是否同时存在,将命令复制过来一份就可以了.

相关教程: [http://bbs.linuxtone.org/thread-2201-1-1.html](http://bbs.linuxtone.org/thread-2201-1-1.html)