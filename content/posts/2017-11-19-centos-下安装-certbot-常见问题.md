---
title: centos 下安装 certbot 常见问题
author: admin
type: post
date: 2017-11-19T03:46:30+00:00
url: /archives/17491
categories:
 - 系统架构
tags:
 - certbot
 - https

---
上一篇( [https://blog.haohtml.com/archives/17422](https://blog.haohtml.com/archives/17422))我们介绍了centos下安装certbot的方法，但有时间服务器环境不一样，总会遇到一些问题，常见问题如下：

centos7.5下安装certbot常见问题

一、出错”ImportError: ‘pyOpenSSL’ module missing required functionality. Try upgrading to v0.14 or newer.“
解决办法：

```
sudo pip uninstall pyOpenssl
sudo pip install pyOpenSSL==0.14.0

```

查看版本：

```
pip show pyOpenssl

```

一、出错信息为“certbot AttributeError: ‘module’ object has no attribute ‘SSL\_ST\_INIT’”

解决办法：

```
pip uninstall pyOpenSSL
pip install pyOpenSSL==16.2.0

```