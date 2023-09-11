---
title: Mongdb监控 (MMS)
author: admin
type: post
date: 2012-05-07T03:27:01+00:00
url: /archives/12858
IM_data:
 - 'a:1:{s:55:"http://photo.l99.com/bigger/02/1332162468834_m9mf67.jpg";s:55:"http://photo.l99.com/bigger/02/1332162468834_m9mf67.jpg";}'
IM_contentdowned:
 - 1
categories:
 - nosql
tags:
 - Mongdb

---
MMS (MongoDB Monitoring Service) is a hosted application created by [10gen](http://10gen.com/) for monitoring MongoDB deployments. MMS Collects statistics on all key server and hardware indicators and presents this data through an intuitive web interface

**先简单说下原理：**

1、在mms服务器上添加mongodb服务器的ip,端口，user，password.
2、在mongodb服务器所在的内网空闲机器上安装定制的agent脚本。
3、agent脚本从mms获取你的服务器Ip及端口等信息，然后连接到mongodb服务器获取必要的监控数据。
4、agent脚本将监控信息上传到mms服务器，我们登陆后就可以查看到相应的信息了。

主页地址如下

１. [https://mms.10gen.com/help/install.html](https://mms.10gen.com/help/install.html)

在Hosts项上点击添加，此处hostname填写内网IP，port为mongodb端口，user/password为用户名和密码。

[![](http://blog.haohtml.com/wp-content/uploads/2012/05/mongodb_host.jpg)][1]



首先得检查一下服务器上的python版本

测试服务器上python版本为2.4

需要安装pymongo,simplejson,hashlib,hmac等包

**可以先安装 python的setuptools.**

sudo apt-get install python-setuptools

然后依次安装：easy_install simplejson hmac hashlib

**再安装 pymongo**

可能需要安装依赖

先执行

apt-get install build-essential python-dev

yum 命令

yum install gcc python-devel

安装 easy_install pymongo

sudo easy_install -U pymongo



如果不用easy_install

则 pymongo：http:http://api.mongodb.org/python/
simplejson:http://pypi.python.org/pypi/simplejson/2.1.0
hashlib:http://pypi.python.org/pypi/hashlib/20081119
hmac:http://pypi.python.org/pypi/hmac



下载 mms-agent

[https://mms.10gen.com/settings/10gen-mms-agent.zip](https://mms.10gen.com/settings/10gen-mms-agent.zip)

解压并进入目录中

执行 nohup python agent.py >/tmp/agent.log 2>&1 & 即可

建议直接将python 升级至最新版本。2.4版本中需要解决各种依赖问题。太过繁琐。

登陆mms.10gen.com，登陆后查看Hosts 可看到相关mongodb信息。

done.



附开启防火墙出口443，

>  iptables -t filter -A OUTPUT -o eth1 -p tcp -m state –state NEW -m tcp –dport 443 -j ACCEPT
> service iptables restart



 [1]: http://blog.haohtml.com/wp-content/uploads/2012/05/mongodb_host.jpg