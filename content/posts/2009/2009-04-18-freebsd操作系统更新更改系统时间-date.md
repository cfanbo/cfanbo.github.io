---
title: FREEBSD操作系统更新更改系统时间 date
author: admin
type: post
date: 2009-04-18T13:15:22+00:00
url: /archives/1247
IM_contentdowned:
 - 1
categories:
 - 服务器

---

修改FreeBSD的系统时间

必须有root权限

# date YYMMDDHHMM

比如要修改时间为2007年4月15日7点52

# date 0704150752

只改时间的话

# date HHMM

使用NTP服务器更新本地时间

# ntpdate time.nist.gov


常用的NTP服务器

time.nist.gov

time.windows.com

chime.utoronto.ca

ntp.pipex.net