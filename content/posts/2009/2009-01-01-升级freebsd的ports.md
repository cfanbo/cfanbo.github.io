---
title: 升级FreeBSD的ports
author: admin
type: post
date: 2009-01-01T03:04:30+00:00
excerpt: |
 |
 Xinsoft-BSD# cp /usr/share/examples/cvsup/ports-supfile /root
 Xinsoft-BSD# vi /etc/make.conf

 # added by root [Xinoft] 2006-02-05 03:52:11
 # for cvsup
 # Block_CVSUP :: begin

 SUP_UPDATE= yes
 SUP= /usr/local/bin/cvsup
url: /archives/782
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ports

---
Xinsoft-BSD# cp /usr/share/examples/cvsup/ports-supfile /root

 Xinsoft-BSD# vi /etc/make.conf

# added by root [Xinoft] 2006-02-05 03:52:11

 # for cvsup

 # Block_CVSUP :: beginSUP_UPDATE= yes

 SUP= /usr/local/bin/cvsup

 SUPFLAGS= -g -L 2# cvsup[1-9].tw.FreeBSD.org

 SUPHOST= [ftp.freebsdchina.org][1]

SUPFILE= /usr/share/examples/cvsup/stable-supfile

 PORTSSUPFILE= /root/ports-supfile

 DOCSUPFILE= /usr/share/examples/cvsup/doc-supfileMASTER_SITE_BACKUP?=

[ftp://ibm.tju.edu.cn/pub/FreeBSD/ports/distfiles/${DIST_SUBDIR}/][2]
[ftp://ftp.freebsd.org.cn/pub/FreeBSD/ports/distfiles/${DIST_SUBDIR}/][3]
[ftp://ftp.freebsdchina.org/pub/FreeBSD/ports/distfiles/${DIST_SUBDIR}/][4]
# Block_CVSUP :: endXinsoft-BSD# cd /usr/ports/

 Xinsoft-BSD# make update

 [1]: ftp://ftp.freebsdchina.org/
 [2]: ftp://ibm.tju.edu.cn/pub/FreeBSD/ports/distfiles/$%7BDIST_SUBDIR%7D//
 [3]: ftp://ftp.freebsd.org.cn/pub/FreeBSD/ports/distfiles/$%7BDIST_SUBDIR%7D//
 [4]: ftp://ftp.freebsdchina.org/pub/FreeBSD/ports/distfiles/$%7BDIST_SUBDIR%7D/