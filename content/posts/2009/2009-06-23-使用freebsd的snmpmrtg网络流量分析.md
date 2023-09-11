---
title: 使用FreeBSD的SNMP+MRTG网络流量分析
author: admin
type: post
date: 2009-06-23T12:11:53+00:00
excerpt: |
 、 安装SNMP
 一般版本的FreeBSD系统SNMP存放在/usr/ports/net/net-snmp下面，但是有的版本不是。有些版本在安装Package的时候，除了要安装Net之外，还要安装Net-mgmt里面的SNMP，安装好之后，SNMP就存放在/usr/ports /net-mgmt/net-snmp下面了。下面就是安装过程：
 # cd /usr/ports/net-mgmt/net-snmp #snmp的存放路径
 # make install clean #安装snmp
url: /archives/1904
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - mrtg
 - 流量监控

---
、 安装SNMP
一般版本的FreeBSD系统SNMP存放在/usr/ports/net/net-snmp下面，但是有的版本不是。有些版本 在安装Package的时候，除了要安装Net之外，还要安装Net-mgmt里面的SNMP，安装好之后，SNMP就存放在/usr/ports /net-mgmt/net-snmp下面了。下面就是安装过程：
\# cd /usr/ports/net-mgmt/net-snmp #snmp的存放路径
\# make install clean #安装snmp
\# ee /etc/rc.conf
snmpd_enable=”YES”
snmpd_flags=”-p /var/run/snmpd.pid”
\# /etc/netstart
\# ee /usr/local/share/snmp/snmpd.conf
rocommunity public
\# /usr/local/etc/rc.d/snmpd.sh start #启动snmp

2、 安装mrtg
mrtg根据不同的版本存放的位置不同，一般存放在/usr/ports/net/net-snmp下面，这里介绍的安装过程种mrtg存放在/usr/ports/net-mgmt/mrtg下面。
\# cd /usr/ports/net-mgmt/mrtg #mrtg的存放路径
\# make install clean #安装mrtg
\# cd /home #以下四个命令是建立MRTG
\# mkdir http #的WEB目录，具体目录可以
\# cd http #根据个人的爱好自己设定
\# mkdir mrtg
\# cd /usr/local/etc/mrtg
\# /usr/local/bin/cfgmaker public@192.168.1.100 > mrtg #创建MRTG的cfg文件
192.168.1.100 ：被监控设备的地址
mrtg ：是要输出的档案
public ：设备设定档的共同的名字(community name) 预设是public，
这个可以在/usr/local/share/snmp/snmpd.conf里面修改
\# ee mrtg
WorkDir: /home/http/mrtg #指向已设定的WEB目录
\# /usr/local/bin/indexmaker –-title ‘标题’ –output
/home/http/mrtg/index.html mrtg #生成index.html文件
\# /usr/local/bin/mrtg /usr/local/etc/mrtg/mrtg #运行mrtg（如果有错误，就
多运行几次）
#ee /etc/crontab #让mrtg每5分钟运行一次
\*/5 \* \* \* root /usr/local/bin/mrtg /usr/local/etc/mrtg/mrtg

3、 安装apache
apache存放在/usr/ports/www/apache2下面
\# cd /usr/ports/www/apache2 #apache2的存放地址
\# make install clean #安装apache2
\# ee /etc/rc.conf
apache2_enable=”YES”
\# /etc/netstart
\# ee /usr/local/etc/apache2/httpd.conf #配置虚拟主机
NameVirtualHost *:80


Options Indexes Includes FollowSymlinks
Allow from all #允许访问

ServerAdmin root@test.com
DocumentRoot /home/http/mrtg
ServerName xxx.xxx.xxx.xxx #安装mrtg的主机地址
DirectoryIndex index.html #前面生成的index.html
ErrorLog /var/log/xxx.xxx.xxx.xxx-error_log
CustomLog /var/log/xxx.xxx.xxx.xxx-access_log common

\# /usr/local/etc/rc.d/apache2.sh start #启动apache
打开http://xxx.xxx.xxx.xxx，就可以看到被监控设备的网络信息了。

4、 设置http://xxx.xxx.xxx.xxx的访问权限
监控流量的网页做好之后，接下来就设置访问这个网页的权限。
1) 修改http.conf ，在和
之间加入一行：
AllowOverride All
意思是在/home/http/mrtg下不同目录的访问权限由该目录下的.htaccess文件来控制，而且不同目录的权限策略可互相覆盖
2) 编辑.htaccess 文件
\# cd /home/http/mrtg
\# mkdir user #建立存放密码文件的文件夹
\# ee .htaccess #访问权限控制文件
AuthUserFile /home/http/mrtg/user/pass #用户密码信息存放文件
AuthType Basic #认证类型为基本型
AuthName “cnseaport”
require valid-user #认证方式
3) 建立用户
\# htpasswd –c /home/http/mrtg/user/pass admin #建立用户admin
New password: #输入用户秘密
Re-type new password: #再次输入密码
Adding password for user admin #添加用户成功信息
可以建立多个用户
4) 重新启动apache，再次访问http://xxx.xxx.xxx.xxx，这时应该出现一个
身份认证窗口，你需要输入用户名和密码才能访问这个页面。