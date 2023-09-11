---
title: 编译php时make ZEND_EXTRA_LIBS=’-liconv’这句命令的作用
author: admin
type: post
date: 2010-10-13T15:48:51+00:00
url: /archives/6047
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos

---
#make _ZEND\_EXTRA\_LIBS_=’-liconv’

#make install

可能是因为机器没有安装libiconv之类的库，怕编译出错，所以不为php加入iconv模块吧。