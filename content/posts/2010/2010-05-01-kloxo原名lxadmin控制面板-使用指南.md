---
title: Kloxo(原名LxAdmin)控制面板 使用指南
author: admin
type: post
date: 2010-05-01T12:33:44+00:00
url: /archives/3532
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - 虚拟主机
 - Linux

---
Kloxo是一个优秀的Web控制面板，有商业版本和免费版本。免费版本的Kloxo允许绑定40个域名，对普通客户来讲40个域名也足够用了。瑞豪开源的VPS提供了预装免费版Kloxo的Linux系统。本文介绍Kloxo控制面板的基本使用方法。

VPS安装好之后，我们会告诉客户Kloxo的登录地址以及admin用户的密码，登录之后就可以开始配置了。

## 升级Kloxo到最新版本

进入Kloxo后要做的第一件事情就是升级Kloxo到最新版本，这是非常必要的，因为老版本可能有bug存在，这些bug有可能导致Kloxo被入侵，而最新版本往往修复了这些bug。

在首页中间的Administration部分，点击Update Home然后就会看到当前的Kloxo是否是最新版本，如果不是最新版本，就点击下面的Update Now按钮进行升级。

## 添加DNS模板

添加DNS模板是必要的，如果不添加DNS模板，将无法添加域名，无法添加新用户。

添加DNS模板，首先点击左侧菜单中的：Resources –> DNS Templates 或者首页中部的Resources –> DNS Templates，然后在新出现的页面中点击Add DNS Template，添加窗口就出现了，在窗口中如下填写：

 * DNS Template Name：随便填写，仅仅是一个名字而已
 * Web Ipaddress：缺省有IP地址，无需填写
 * Mail Ipaddress：缺省有IP地址，无需填写
 * Primary DNS：建议填写208.67.222.222
 * Secondary DNS：建议填写208.67.220.220

填写完成之后点击Add即可。

## 添加用户

本步骤是可选的，不是必须的。Kloxo控制面板缺省只有一个admin用户，这个用户是管理员用户，管理员用户下面可以添加很多域名。也可以创建一些普通用户，每个普通用户下面也可以绑定很多域名。

点击左侧菜单中的Clients或者首页中部的Clients，然后在新页面中点击Add Customer，然后的窗口中：

 * Client Name：用户名
 * Domain Name：这个用户的第一个域名，可以先空着不填，让用户自己登录后自己填写
 * Install Application：缺省安装的网站程序，有Wordpress, Drupal等常用的网站程序，建议不要选择，因为这里安装的都是老版本，不好
 * Password：用户的密码
 * Email Address：用户的email地址，必须填写，当用户忘记密码后可以根据Email找回
 * Send Welcome Message：这个选项要选上
 * Choose Plan：这是要开通的空间的型号，不要管，除非你是卖空间的

然后点击Add，出现新的页面，新页面里的信息不需要修改，继续点击Add即可。然后系统就会给用户的邮箱里面发生邮件，告知登录地址，用户名密码等信息。

## 添加域名

admin用户和普通用户都可以绑定域名，创建普通用户的时候也可以顺便绑定一个域名。

在左侧菜单中点击domains即可进入添加域名的界面，假设我们要添加的域名是 rashost.com ，那么在该界面中Domain Name部分就填写rashost.com；Document Root是域名的文件所在的目录，通常也填写为域名；其他部分不用填写，点击Add即可。

## 上传文件

上传文件可以通过FTP，也可以通过网页上传

在左侧菜单中点击Resources–>File Manager（admin用户需要点击domain–>File Manager），然后进入文件管理器，在文件管理器里面可以点击upload上传文件。

也可以通过FTP上传文件，一般绑定了一个域名之后会自动创建一个FTP用户，FTP用户的名字和域名是相同的，FTP密码就是当前用户的密码。当然也可以另外创建FTP用户，在左侧菜单点击Resources–>FTP Users（admin用户需要点击domain–>FTP Users）就进入管理FTP用户的界面了。

## Email邮箱管理

绑定一个域名之后，以这个域名为后缀的邮箱就开通了。我们仅需要创建一个邮箱帐户就可以了。

点击左侧菜单下部的Mail Accounts进入邮箱帐户管理页面，可以在这里管理邮箱帐户。

假设域名是rashost.com，新创建的邮箱帐号是zzh，那么邮件地址就是zzh@rashost.com。邮箱用户可以通过http://webmail.rashost.com 进入Web邮箱（前提是域名的webmail记录必须指向了VPS的IP）。

## Kloxo的中文汉化

SSH登录到VPS上，执行如下命令：

`cd /usr/local/lxlabs/kloxo/httpdocs/lang/

wget rashost.com/download/kloxo-cn.tar.gz

tar zxf kloxo-cn.tar.gz

chown -R lxlabs: cn`

然后登录Kloxo，在首页点击Appearance，然后点击Language框，选择Chinese，最后点击Update按钮即可

## 定期删除日志脚本

在/etc/cron.daily目录下面创建文件cleankloxolog.sh，修改该文件的权限为755：

`chmod 755 /etc/cron.daily/cleankloxolog.sh`

这个可执行文件每天会被自动执行一次，每次执行都会删除kloxo的日志。

该文件内容如下：

`#!/bin/bash<br />
rm -rf  /home/admin/__processed_stats/*<br />
rm -rf /home/kloxo/httpd/lighttpd/*<br />
rm -rf  /var/log/kloxo/*`

`rm -f /home/httpd/*/stats/*`

## 其他功能

以上简述了一下必要的功能，Kloxo还有很多其他功能，用户可以自己去探索。