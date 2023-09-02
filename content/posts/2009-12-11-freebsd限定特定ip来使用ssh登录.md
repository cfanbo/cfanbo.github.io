---
title: Freebsd限定特定IP来使用ssh登录
author: admin
type: post
date: 2009-12-11T01:39:40+00:00
url: /archives/2707
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ssh

---

法1.

#ee /etc/hosts.allow

在ALL : ALL : allow的前面加上

sshd : your IP : allow

sshd : ALL : deny

就OK了。

法2.

修改/etc/ssh/sshd_config

加入

Allowusers admin@172.16.2.188

意思为

只允许admin从172.16.2.188登陆

法3.

防火墙