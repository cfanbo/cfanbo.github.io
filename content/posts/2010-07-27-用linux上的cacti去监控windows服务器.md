---
title: 用linux上的cacti去监控windows服务器
author: admin
type: post
date: 2010-07-27T06:35:14+00:00
url: /archives/4841
IM_data:
 - 'a:7:{s:60:"http://www.haohtml.com/uploads/allimg/100727/14345C392-2.jpg";s:71:"http://blog.haohtml.com/wp-content/uploads/2011/03/3e9c_14345C392-2.jpg";s:59:"http://www.haohtml.com/uploads/allimg/100727/14345C5B-3.jpg";s:70:"http://blog.haohtml.com/wp-content/uploads/2011/03/dc27_14345C5B-3.jpg";s:61:"http://www.haohtml.com/uploads/allimg/100727/1434563026-4.gif";s:72:"http://blog.haohtml.com/wp-content/uploads/2011/03/d346_1434563026-4.gif";s:61:"http://www.haohtml.com/uploads/allimg/100727/1434561638-5.jpg";s:72:"http://blog.haohtml.com/wp-content/uploads/2011/03/4477_1434561638-5.jpg";s:61:"http://www.haohtml.com/uploads/allimg/100727/1434562453-6.jpg";s:72:"http://blog.haohtml.com/wp-content/uploads/2011/03/7f3e_1434562453-6.jpg";s:59:"http://www.haohtml.com/uploads/allimg/100727/14345B1N-7.jpg";s:70:"http://blog.haohtml.com/wp-content/uploads/2011/03/d647_14345B1N-7.jpg";s:60:"http://www.haohtml.com/uploads/allimg/100727/14345611S-8.jpg";s:71:"http://blog.haohtml.com/wp-content/uploads/2011/03/cbf1_14345611S-8.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - CACTI
 - Linux

---

另篇相同的教程: [http://blog.haohtml.com/index.php/archives/4850](http://blog.haohtml.com/index.php/archives/4850)

以前一直用cacti或者mrtg来监控交换机流量，很少用来监控服务器，最近突然有个任务需要监控windows服务器，一般刚装好的cacti，里面的监控设置都是基于交换机和linux的，没有专门监控windows的选择，于是研究了一下，和大家分享一下经验。另外我的cacti是安装的debian linux上，有些安装命令不适合其他linux上，请大家注意。


操作系统：debian 5

**1.安装mysql**

apy-get install mysql-server-5.0

安装时会提示你输入mysql root密码


**2.安装apache和php**

apt-get install apache2 libapache2-mod-php5 php5 php5-gd php5-mysql php5-cli php5-common php5-snmp php-net-socket


php5-gd是关系到绘图

php5-mysql和数据库有关系

php-net-socket这个有时候cacti需要

**3.安装cacti**

apt-get install cacti rrdtool snmp

安装时会要求输入刚才你设置的mysql root密码，然后会自动建立个cacti库，同时也需要输入密码.


以上cacti就安装完毕了，非常的简单明了吧，debian就是这点好，优点就是安装软件快，不需要你去下什么rpm包之类的，一句话全搞定.


**cacti的设置**

1.首先把监控windows的脚本导入到cacti

附件里有个Cacti_SNMP_INFORMANT_STD_W32_Metrics.zip的包，里面包含的文件就是脚本文件，其中snmp_informant_.xml开头的文件是需要放到cacti服务端的snmp_queries目录下，如果你的debian 的话，目录地址是/usr/share/cacti/resource/snmp_queries/。cacti_data_query开头的文件全都通过cacti页面导入。

[![](http://blogx.haohtml.com/wp-content/uploads/2010/07/1434563418-0.jpg)](http://blog.haohtml.com/wp-content/uploads/2010/07/1434563418-0.jpg)

2.在Devices中新建立个服务器，填写名称和ip地址，还有snmp信息，最下面的地方add如下东西：

[![](https://blogstatic.haohtml.com//uploads/2023/09/1434563a8-1.jpg)](http://blog.haohtml.com/wp-content/uploads/2010/07/1434563a8-1.jpg)

3.然后点最上面的Create Graphs for this Host ，选择Graphs Types，添加需要监控的项目。



这样基本上就可以了，然后设置windows服务器


**客户端windows安装snmp**

这里需要说明的是，除了windows自带安装的snmp之外，还需要安装SNMP Informant-STD 1.6

软件下载地址： [http://www.wtcs.org/informant/download.htm](http://www.wtcs.org/informant/download.htm)

只要安装好就可以了，不需要任何设置，当然之前你自带的snmp需要设置一下，一个是设置public，一个是监控你snmp的服务器IP地址，也就是cacti的机器IP地址，设置好后记得重起一下snmp服务，这点很重要，然后检查一下服务器是不是开放了udp 161端口，还有防火墙是不是开放了这端口。


![](http://www.haohtml.com/uploads/allimg/100727/14345C5B-3.jpg)

相关连接： [http://forums.cacti.net/about29832.h…hlight=Windows](http://forums.cacti.net/about29832.html&highlight=Windows) 这个是cacti网站上的关于这脚本使用的方法，如果英文好的话可以详细研究一下

貌似写的很仓促，我不太喜欢写文档呵呵，如果有问题，可以联系我well_wong@hotmail.com


上传的附件![文件类型: zip](http://www.haohtml.com/uploads/allimg/100727/1434563026-4.gif)[Cacti_SNMP_INFORMANT_STD_W32_Metrics.zip](http://www.linuxsir.org/bbs/attachment.php?s=7a0a60f2cdfb7c41805e04432d418772&attachmentid=51147&d=1236752610) (39.1 KB, 176 次查看)
