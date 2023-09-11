---
title: 用wget加快ports下载
author: admin
type: post
date: 2009-04-18T11:14:59+00:00
url: /archives/1243
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ports

---

1.安装wget

#cd /usr/ports/ftp/wget/

#make install clean

2.修改/etc/make.conf

FETCH_CMD=wget -c -t 1

DISABLE_SIZE=yes #这行是必要的，否则…

如果你要wget穿透代理服务器，请加上下面两行

FETCH_ENV=http_proxy=http://proxy2.zsu.edu.cn:3128

FETCH_ENV=ftp_proxy=http://proxy2.zsu.edu.cn:3128

或者使用其他的穿越代理工具例如proxychains 或者socks5(runsocks)

则FETCH_CMD=proxychains wget或者runsocks wget