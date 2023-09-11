---
title: CentOS上DirectAdmin安装教程
author: admin
type: post
date: 2010-12-02T09:09:43+00:00
url: /archives/6822
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - DirectAdmin
 - 虚拟主机

---

[![](http://blog.haohtml.com/wp-content/uploads/2010/12/directadmin.gif)](http://blog.haohtml.com/wp-content/uploads/2010/12/directadmin.gif)

DirectAdmin是一款付费的虚拟主机管理软件，通常简称为DA，DA比Cpanel功能上简单，但是内存占用也更少些，更重要的是价格也更便宜，一般自己用或者搞合租DA算是很合适的。

安装前首先确保已经购买了DirectAdmin的授权，购买授权后会有Client ID，License ID，也需要在DA官网上或者DA销售商那里提交你的VPS或者服务器的IP和系统信息。

1、安装CentOS的相关组件的命令如下：

> yum update -y
> yum install gcc-c++ gcc make automake wget flex -y

2、安装DirectAdmin需要干净的系统，所以在装之前要卸载掉httpd、php、mysql。

> yum remove httpd\* php\* mysql* -y

3、下载DirectAdmin安装脚本文件，执行命令：

> wget [http://directadmin.com/setup.sh](http://directadmin.com/setup.sh)

4、为DirectAdmin安装脚本文件添加执行权限，执行命令：

> chmod +x setup.sh

5、执行DirectAdmin安装脚本文件：

> ./setup.sh

运行DirectAdmin安装文件，然后在下面填入相关的信息。其中hostname最好最好使用如：linode.vpser.net 之类的二级域名不要使用顶级域名。

DirectAdmin’s setup has a few more things you need to fill:
Please enter your Client ID :                   //输入你的Client ID
Please enter your License ID :                    //输入你的License ID
Please enter your hostname (server.domain.com)                         //输入一个主机名，如：linode.vpser.net
It must be a Fully Qualified Domain Name
Do \*not\* use a domain you plan on using for the hostname:
eg. don’t use domain.com. Use server.domain.com instead.
Do not enter http:// or www
Enter your hostname (FQDN) :
Is this correct? (y,n) :                          //提示上面是否正确，正确请输入y
Is eth0 your network adaptor with the license IP? (y,n) :             //输入y
Is xx.xx.xx.xx the IP in your license? (y,n) :                           //确认IP是否是License上注册的IP
DirectAdmin will now be installed on: Enterprise 5
Is this correct? (must match license) (y,n) :               //输入y
You now have 2 options for your apache/php setup.
1: customapache: older, more tested. Includes Apache 1.3, php 4 and frontpage.
2: custombuild 1.1: newer, less tested. Includes any Apache version, php 4, 5, or both in cli and/or suphp. Frontpage not available with Apache 2.x.
Post any issues with custombuild to the forum: [http://www.directadmin.com/forum/forumdisplay.php?f=61](http://www.directadmin.com/forum/forumdisplay.php?f=61)
Enter your choice (1 or 2):        //一般选择2就行，使用php5

经过这些步骤,DirectAdmin的安装已经完成了。

安装完成后会提示：

DirectAdmin的用户名密码及DirectAdmin的管理后台地址等。

一般基于OpenVZ的VPS需要在使用前打开/usr/local/directadmin/conf/directadmin.conf这个文件，确认其中的ethernet_dev的值修改为：venet0:0 ，具体已ifconfig 为准。

执行：service directadmin restart 重启DirectAdmin，用http://IP:2222 登录DirectAdmin后台。