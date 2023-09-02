---
title: Snapshot appears to have been created more than one day into the future!
author: admin
type: post
date: 2010-12-26T06:19:02+00:00
url: /archives/7188
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - portsnap

---
本地刚装完freebsd7.0,连上ssh，portsnap fetch extract一下，提示：

> Snapshot appears to have been created more than one day into the future!
> (Is the system clock correct?)
> Cowardly refusing to proceed any further.

原来是安装时系统时间不正确
执行代码:

> ntpdate pool.ntp.org

然后再执行portsnap fetch即可.