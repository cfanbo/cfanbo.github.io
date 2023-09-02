---
title: nagios访问apache权限问题解决
author: admin
type: post
date: 2011-08-02T10:00:14+00:00
url: /archives/10846
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nagios

---

It appears as though you do not have permission to view information for any of the services you requested…


打开cgi.cfg配置文件，里面有个参数:

use_authentication=1

为了保障系统的安全性，nagios设置了这个参数，默认为1,改为0即可


1.装了几次，换了几个版本的系统，脑袋都大了，终于解决了


nrpe在 ./configure时提示


checking for SSL… configure: error: Cannot find ssl libraries


把openssl-devel装上就可以了

2.nagios web界面提示


It appears as though you do not have permission to view information for any of the services you requested…


打开cgi.cfg配置文件，里面有个参数:

use_authentication=1

为了保障系统的安全性，nagios设置了这个参数，默认为1,改为0即可。


3.Service Commands 中Enable notifications for this service时报错


Sorry Dave, I can’t let you do that…

It seems that you have chosen to not use the authentication functionality of the CGIs.

I don’t want to be personally responsible for what may happen as a result of allowing unauthorized users to issue commands to Nagios,so you’ll have to disable this safeguard if you are really stubborn and want to invite trouble.


Read the section on CGI authentication in the HTML documentation to learn how you can enable authentication and why you should want to.

修改cgi.cfg文件

修改use_authentication=1 (默认) ，如果没有的话,就手动添加，然后重启nagios服务。

4.is not allowed to connect to this MySQL server

server(nagios服务端192.168.0.132)

#/usr/local/nagios/libexec/check_mysql -H 192.168.0.207 -u root -p xukixu

此时可能会出现错误：Host ‘192.168.0.132’ is not allowed to connect to this MySQL server

因此只要在客户端做个mysql授权用户访问即可

2、client(客户端192.168.0.207)

#mysql -uroot -pabcd

mysql>grant all privileges on *.* to [root@192.168.0.132](mailto:root@192.168.0.132) identified by ‘abcd;

mysql>flush privileges;

mysql>quit;