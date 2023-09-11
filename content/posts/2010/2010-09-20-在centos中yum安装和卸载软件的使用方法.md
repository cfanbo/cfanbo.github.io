---
title: 在Centos中yum安装和卸载软件的使用方法
author: admin
type: post
date: 2010-09-20T10:43:23+00:00
url: /archives/5784
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos

---
Yum(全 称为 Yellow dog Updater, Modified)是一个在Fedora,Redhat,CentOS中的Shell前端软件包办理器.基於RPM包办理,可以或许从指定的服务器AUTO下载 RPM包而且安装,可以AUTO处理依赖性关系,而且一次安装所有依赖的软体包,无须繁琐地一次次下载、安装.

**安装一个软件时**

yum -y install httpd

 **安装多个相类似的软件时**

yum -y install httpd*

 **安装多个非类似软件时**

yum -y install httpd php php-gd mysql

 **卸载一个软件时**

yum -y remove httpd

 **卸载多个相类似的软件**

yum -y remove httpd*

 **卸载多个非类似软件时**

yum -y remove httpd php php-gd mysql
=======================================

别的还有一个非常棒的用法

假如我要执行**iostat**这个命令来查看CPU与 存储设备状态,可是执行却发现没有这个命令

于是执行yum install **iostat**,结果说找不到该软件,使用下面的措施可以解决

yum search **iostat**就能查到以及iostat相干的安装包了,

别的想安装一个程序,只记得一部门名称,也可以用这个措施来实现安装

yum search png |grep png

就能找到我们想安装的libpng这个名称