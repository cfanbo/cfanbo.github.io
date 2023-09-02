---
title: freebsd ssh 服务器登录失败问题的解决
author: admin
type: post
date: 2009-01-01T11:03:29+00:00
url: /archives/780
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ssh

---
编辑/etc/ssh/sshd_config 保证设置以下参数：

PermitRootLogin yes
PasswordAuthentication yes
UseDNS no
LoginGraceTime 0