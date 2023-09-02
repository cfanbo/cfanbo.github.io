---
title: 'redhat下关于Public key for *.rpm is not installed 的解决方法'
author: admin
type: post
date: 2011-03-21T11:58:09+00:00
url: /archives/8063
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - redhat

---
今天在用yum 安装httpd的时候出现了一下错误：
**_warning: rpmts_HdrFromFdno: Header V3 DSA signature: NOKEY, key ID e8562897
update/gpgkey                                            | 1.8 kB     00:00



Public key for apr-util-1.2.7-11.el5_5.1.i386.rpm is not installed

在网上找到了解决方法：

此时要导入rpm的签名信息即可

以root登录，执行下面命令

> \# rpm –import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

根据我的Linux版本是CentOS 5.4

于是我执行下面命令

> **_#rpm –import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5_**

** __**问题终于得到解决！