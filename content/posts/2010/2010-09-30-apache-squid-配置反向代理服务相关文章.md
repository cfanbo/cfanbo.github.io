---
title: apache squid 配置反向代理服务相关文章
author: admin
type: post
date: 2010-09-30T01:46:19+00:00
url: /archives/5895
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - apache
 - Squid

---
 * [apache + squid 配置反向代理服务][1]环境 ：ubuntu 9.04（linux 2.6.28-15-generic） apache版本：Apache/2.2.11 squid版本：Squid3.0/STABLE8 本机IP ：192.168.1.102并在/etc/hosts里添加www.abc.com的伪域名以便测试 安装方式 ：apt－get安装（源码安装同） 配置文件： apache：（/etc/apach…
 * [Linux下Squid3.0反向代理的安装与配置][2]1. Squid3.0的安装是很简单的： ./configure –prefix=/usr/local/squid make make install chown -R nobody.nobody /usr/local/squid/var/ /usr/local/squid/sbin/squid -z Squid3.0的配置也不复杂： 假设我们有两台Apache服务器需要反向代理：www.avnads.c…
 * [squid3.0快速缓存实现][3]一、编译安装 #tar zxvf squid-3.0.STABLE11.tar.gz //稳定版 #cd squid-3.0.STABLE11 #./configure –prefix=/usr/local/squid \ –enable-arp-acl \ –enable-linux-netfilter \ –enable-pthreads \ –enable-err-language=Simplify_Chinese \ –enable-d…

 * [对外网用户的squid代理+认证(51)][4]FreeBSD6.2+Squid2.6架设对外网用户的squid代理+认证服务器 架设一台代理，提供对外网用户的代理请求，端口仍然为3128，加入Squid认证功能。这样可以保证只提供给某些你信任的用户该服务。架设过程和架设对内网用户提供服务的过程基本相同，只是在编译安装sq…
 * [squid3.0反向代理 apache+squid][5]squid3.0反向代理 apache+squid2008-03-19 16:10apache(81端口)+squid(80端口)（apache和squid跑在同一个机器上面 要实现反向代理）我将我的外网域名用abc.com代替了 apache简单配置如下： Listen 81 NameVirtualHost \* VirtualHost \* Directory /usr/local/…
 * [squid、apache安装配置步骤][6]本人一台web服务器，日流量约10万，上面有好几个虚拟主机，近日装上 Squid 2.6进行WEB加速， Squid 和 Apache 均在同一台服务器上面，效果非常明显，看到论坛上好多人问如何配置 squid 2.6支持,虚拟主机 现在将安装过程贴出和大家一起分享，给菜鸟们一个学习…
 * [用squid加速apache][7]早就看过用squid加速apache的文章，就是懒的去玩，今天闲来郁闷，突然想玩玩，所以就有了本文（本文不算是原创，都是建立他人的基础上凑起来的，算是整理吧！） 系统：redhatas4 apache：httpd-2.0.52-9.ent squid：squid-2.5.STABLE6-3.4E.3 1.安装 安装squ…
 * [基于squid面向apache作反向代理][8]Squid代理Apach的配置 目的： 让squid反向代理apache服务器，提高客户端访问速度。 计划： 将apache的80端口换成8080，再把squid的默认端口改成80，实现客户端对squid的访问。 过程： 1安装Apache服务器 进入压缩文件解压tar xjf httpd-2.0.54.tar.bz2 进入…
 * [用squid加速apache][9]系统：redhat as 4 apache ：httpd-2.0.52-9.ent squid ：squid-2.5.STABLE6-3.4E.3 1.安装 安装squid很简单： # yum -y install squid 配置squid 修改：/etc/squid/squid.conf成下面的 http\_port 80 icp\_port 0 acl QUERY urlpath\_regex cgi-bin no\_cache d…

 [1]: http://www.haohtml.com/server/services/44403.html
 [2]: http://www.haohtml.com/server/services/44402.html
 [3]: http://www.haohtml.com/server/services/44401.html
 [4]: http://www.haohtml.com/server/services/44400.html
 [5]: http://www.haohtml.com/server/services/44399.html
 [6]: http://www.haohtml.com/server/services/44392.html
 [7]: http://www.haohtml.com/server/services/44391.html
 [8]: http://www.haohtml.com/server/services/44390.html
 [9]: http://www.haohtml.com/server/services/44389.html