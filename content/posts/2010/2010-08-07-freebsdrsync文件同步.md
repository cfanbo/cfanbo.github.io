---
title: FreeBSD+Rsync文件同步
author: admin
type: post
date: 2010-08-07T02:55:29+00:00
url: /archives/5010
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - rsync

---
**一.服务端和客户端安装一样**

> woody-207#cd /usr/ports/net/rsync
> woody-207#make install

**二.配置rsync服务端**

> ****woody207# vi /usr/local/etc/rsyncd.conf

添加以下内容

> [www]
> comment = web server backup
> path = /www
> auth users = woody
> uid = nobody
> gid = nogroup
> secrets file = /usr/local/etc/rsyncd.secrets
> read


**启动rsync的daemon模式**

> vi /usr/local/etc/rc.d/rsyncd
> 修改这一行内容，使用IPV4协议
> command_args=”-4 –daemon”

**系统服务配置**

> ****#echo ‘rsyncd_enable=”YES”’ >> /etc/rc.conf

**启动服务**

> ****woody-207# /usr/local/etc/rc.d/rsyncd start

**检查Rsync daemon启动状态**

> ****woody-207# sockstat | grep rsync
> root rsync 586 3 dgram -> /var/run/logpriv
> root rsync 586 4 tcp4 \*:873 \*:*

**三.RSYNC客户端配置**
1、配置rsyncd.secrets

> woody-207#vi /usr/local/etc/rsyncd.secrets //加入以下内容
>
> RXHOEqat6Dhon4HRsM31 //Rsync Server上的认证密码,不用输入用户名
>
> woody-207#chmod 600 /usr/local/etc/rsyncd.secrets

2、进行第一次同步

> /usr/local/bin/rsync -avzP –delete –password-file=/usr/local/etc/rsyncd.secrets woody@192.168.1.207::www /www/backup/

将上面的同步命令写进shell script里面，加一个#!/bin/sh就可以了。