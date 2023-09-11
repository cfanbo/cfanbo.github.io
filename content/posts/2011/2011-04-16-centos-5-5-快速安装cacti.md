---
title: '[教程]Centos 5.5 快速安装cacti'
author: admin
type: post
date: 2011-04-16T16:10:37+00:00
url: /archives/9269
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - CACTI
 - centos

---
**一、准备工作**

环境：Centos 5.4 x86_64
所需软件：

> http
> Php
> Php-mysql
> Php-snmp
> Mysql
> Perl-DBD-MySQL
> Php-pdo
> rrdtool
> Net-snmp
> Net-snmp-libs
> Net-snmp-utils

#下载相关软件

> cd /usr/local/src/
>
> wget http://www.cacti.net/downloads/cacti-0.8.7e.tar.gz

**二、环境介绍**
主监控机是Centos 5.4 x86_64
主监控机IP=10.0.0.52

**三、安装配置**
（1）在主监控机上安装apache+php+gd的web环境,推荐编译安装，不再赘述，本处方便起见用yum装了

> yum install php php-mysql php-snmp mysql mysql-server net-snmp net-snmp-libs net-snmp-utils php-pdo perl-DBD-MySQL

（2）在主监控机上安装rrdtool,rrdtool依赖的包过多，所以选择增加源，然后用yum安装
#增加源

> vi /etc/yum.repos.d/CentOS-Base.repo

#在文件末尾增加以下部分

> [dag]
>
> name=Dag RPM Repository for Red Hat Enterprise Linux
>
> baseurl=http://apt.sw.be/redhat/el$releasever/en/$basearch/dag
>
> gpgcheck=1
>
> gpgkey=http://dag.wieers.com/rpm/packages/RPM-GPG-KEY.dag.txt
>
> enabled=1

> yum install rrdtool

（3）配置snmp
vi /etc/snmp/snmpd.conf
#将下边这行中的default

com2secnotConfigUser default public


#改为127.0.0.1

com2secnotConfigUser 127.0.0.1 public


#将下边这行中的systemview

access notConfigGroup “” any noauth exact systemview none none

#改为all

access notConfigGroup “” any noauth exact all none none


#将下边这行的注释“#”号去掉
view all included .1 80

#重启snmpd服务

> service snmpd restart

（4）安装cacti
#把解压后的包移动到你的相应的web目录

> tar xvf cacti-0.8.7e.tar.gz
>
> mv cacti-0.8.7e /var/www/html/cacti

> （5）在数据库中建库、授权、导入数据库结构
> #注意导入cacti.sql时该文件的路径
> mysql -p

> mysql> create database cacti;
>
> mysql> grant all privileges on cacti.* to cacti@localhost identified by ‘cacti’ with grant option;
>
> mysql> grant all privileges on cacti.* to cacti@127.0.0.1 identified by ‘cacti’ with grant option;
>
> mysql> use cacti;
>
> mysql> source /var/www/html/cacti/cacti.sql;

#配置cacti以连接数据库

> vi /var/www/html/cacti/include/config.php

（6）浏览器下配置
#用浏览器打开 http://10.0.0.52/cacti ，会显示 cacti的安装指南，设置好就不会再出现了。
#点击 “Next”
#选择“New Install”，点击“Next”
#指定 rrdtool、 php、 snmp 工具的 Binary 文件路径，确保所有的路径都是显示“ FOUND”，没有 “NOT FOUND”的，点击 Finish 完成安装。
#Cacti 默认的用户名与密码是 **admin**，输入用户名与密码，点击 login
#为了安全的原因，第一次登录成功后，cacti 会强制要求你更改一个新的 password ，输入新密码并确认密码，点击 save ,进入 cacti 控制台界面：
#点击 graphs ，查看cacti 监控本机的图表：

（7）增加入一个计划任务，使得 cacti 每五分钟生成一个监控图表。

> crontab -e

> #加入如下内容。注意poller.php的路径

> */5 * * * * php /var/www/html/cacti/poller.php > /dev/null 2>&1

#确保 /var/www/html/cacti/rra/目录存在
#如果暂时未看到图表，可以手工执行，生成图表

> #php /var/www/html/cacti/poller.php > /dev/null 2>&1

**（8）使用 Cacti 监控 Linux 主机**
#在被监控的linux主机上安装net-snmp

> yum install net-snmp
>
> vi /etc/snmp/snmpd.conf

> #更改以下部分
>
> #将下边这行中的default
>
>

>

> com2secnotConfigUser default public
>

>

>
> #改为10.0.0.52（cacti）服务器的地址)
>
>

>

> com2sec notConfigUser 10.0.0.52 public
>

>

>
> #将下边这行中的systemview
>
>

>

> access notConfigGroup “” any noauth exact systemview none none
>

>

>
> #改为all
>
>

>

> access notConfigGroup “” any noauth exact all none none
>

>

>
> #将下边这行的注释“#”号去掉
>
>

>

> view all included .1 80
>

>

**重启snmpd服务**

> service snmpd restart

（9）如果出现问题请注意一下snmp协议的版本，都用version 1是一种解决方法
如果都用version 1,需要把所有监控机和被监控机的snmpd.conf改一下

> #vi /etc/snmp/snmpd.conf
> #将下边这行
>
>

>

> view systemview included .1.3.6.1.2.1.1
>

>

>
> #改为
>
> view systemview included .1.3.6.1.2.1

配置是否成功可以使用snmpwalk命令测试,用法见：

如果在运行 poller.php脚本的时候，提示错误

运行cacti的问题Cannot connect to MySQL server on ‘localhost’.Please make sure you have specified a valid MySQL database name in ‘include/config.php’

的请，请参考： [http://blog.haohtml.com/archives/13582](http://blog.haohtml.com/archives/13582)