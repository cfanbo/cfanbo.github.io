---
title: redhat下使用dvd作为yum源
author: admin
type: post
date: 2011-09-09T01:56:25+00:00
url: /archives/11347
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - redhat

---
使用rpm包的最大问题就是安装依赖问题，yum就是为了解决这个问题而出现的，但是Redhat AS的更新是要收费的，所以在解决这个收费的问题之前你只能暂时用安装盘作为yum源，实际上，这也是可以做到的:

**1、mount安装盘到/mnt/cdrom**

> #mkdir /mnt/cdrom
> #mount -t auto /dev/cdrom /mnt/cdrom

**2、创建/etc/yum.rep.d/rhel.repo，文件内容如下:**

> [base]
> name=Red Hat Enterprise Linux $releasever – $basearch
> baseurl=file:///mnt/cdrom/
> enabled=1
> gpgcheck=0
> gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release