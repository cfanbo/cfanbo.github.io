---
title: 如何用Squid Windows版架设二级代理服务器
author: admin
type: post
date: 2010-05-29T07:45:47+00:00
url: /archives/3706
categories:
 - 服务器
tags:
 - Squid

---
一、Windows版Squid的下载与安装

下载windwosNT版本的squid下载地址：

http://squid.acmeconsulting.it/download/squid-2.6.STABLE13-bin.zip

1．把squid-2.6.STABLE13-bin.zip解压缩，把里面的squid文件夹拷到c:\下(squid默认的是c: \squid)

2．squid\etc目录下把

squid.conf.default拷贝一份重新命名为 squid.conf

cachemgr.conf.default拷贝一份重新命名为cachemgr.conf

mime.conf.default 拷贝一份重新命名为mime.conf

3．用文本编辑器打开squid.conf，需要修改的地方：

找到 http_port 3128在后面增加一行

http_port 80 transparent

找 到#cache_peer sib2.foo.net sibling 3128 3130 [proxy-only]在后面增加一行

cache_peer 192.168.1.8 parent 7001 0 no-query originserver

找到# TAG: visible_hostname在后面增加一行

visible_hostname volcano(任意命名)

找到 http_access deny all在其前面加#将这一行注释掉，然后增加一行

http_access allow all

4． 从命令行到c:\squid\sbin目录下执行

squid -i（将squid服务加入到服务里面）

squid -z

安装完成

5．从服务里启动squid

访问squid服务器:

http://192.168.1.2(你 的squid服务器IP地址)>>>指向http://192.168.1.8:7001(web服务器地址)

如果 把#http\_access deny all打开把http\_access allow all注释掉，你的访问就会被拒绝

你需要配置 一下：找到下面两行

#acl our_networks src 192.168.1.0/24 192.168.2.0/24

#http\_access allow our\_networks

打开注释，修改你的内网ip(段)可以设为192.168.1.0/24一个也可以如上面的一样 设一段IP

二、squid.conf配置文件

cache_mgr ghxu@zju.edu.cn #设置管理员邮箱,无关紧要

visible_hostname ibi #设置虚拟主机名,似乎squid2.5这个版本需要

\# 设置这一项,2.4却不需要

cache_peer 10.10.2.53 parent 6666 3130 login=account:passwd default no-query

#设置上级代理,其中10.10.2.53是我们校内的代 理地址,6666是他的端口号,

#account,passwd则是上网帐号密码(当然我不会把我们真实的帐号贴出来)

#hierarchy_stoplist cgi-bin ? #注释掉这一行,不然不能访问带有”?”

#的url

#acl QUERY urlpath_regex cgi-bin ? #这两行没有具体测试,应该和cgi请求有关

#no_cache deny QUERY

acl all src 0.0.0.0/0.0.0.0

acl manager proto cache_object

acl localhost src 127.0.0.1/255.255.255.255

acl SSL_ports port 443 563

acl Safe_ports port 80 # http

acl Safe_ports port 21 # ftp

acl Safe_ports port 443 563 # https, snews

acl Safe_ports port 70 # gopher

acl Safe_ports port 210 # wais

acl Safe_ports port 1025-65535 # unregistered ports

acl Safe_ports port 280 # http-mgmt

acl Safe_ports port 488 # gss-http

acl Safe_ports port 591 # filemaker

acl Safe_ports port 777 # multiling http

acl CONNECT method CONNECT

acl lan-a src 10.49.41.150-10.49.41.190/32 #对ip进行控制,这行定义了一个ip

#段为组 lan-a

http_access allow lan-a #这里控制组lan-a的ip可以使用squid代理

acl lan-b src 10.141.96.0/24 #同样设置了一个ip段,ip地址前三位是

#10.141.96的所有ip,其实 就是我们寝室楼的ip段

http_access allow lan-b

http_access allow manager localhost

http_access deny manager

http\_access deny !Safe\_ports

http\_access deny CONNECT !SSL\_ports

http_access allow localhost

http_access deny all

icp_access allow all

never_direct allow all #这一行解决无法登陆的问题.