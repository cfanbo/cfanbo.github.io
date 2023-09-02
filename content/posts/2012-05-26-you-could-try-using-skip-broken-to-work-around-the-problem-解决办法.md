---
title: You could try using –skip-broken to work around the problem 解决办法
author: admin
type: post
date: 2012-05-26T14:06:15+00:00
url: /archives/13056
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos

---

–> Missing Dependency: libevent-1.4.so.2()(64bit) is needed by package mysql-proxy-0.5.1-2.el5.x86_64 (epel)

Error: Missing Dependency: libevent-1.4.so.2()(64bit) is needed by package mysql-proxy-0.5.1-2.el5.x86_64 (epel)

You could try using –skip-broken to work around the problem

You could try running: package-cleanup –problems

package-cleanup –dupes

rpm -Va –nofiles –nodigest

The program package-cleanup is found in the yum-utils package.


解决：


[root@oracle10g bin]# yum –skip-broken update