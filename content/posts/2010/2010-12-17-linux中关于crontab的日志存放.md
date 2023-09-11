---
title: linux中关于crontab的日志存放
author: admin
type: post
date: 2010-12-17T12:29:19+00:00
url: /archives/7018
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - crontab

---
默认情况下,crontab中执行的日志写在/var/log下,如:

#ls /var/log/cron*

/var/log/cron /var/log/cron.1 /var/log/cron.2 /var/log/cron.3 /var/log/cron.4

crontab的日志,当crond执行任务失败时会给用户发一封邮件.如果在服务器上发现一个任务没有正常执行,而crond发邮件也失败.通过看mail的日志,看是否是磁盘空间不够造成的

将cornd错误输出和标准输出日志都指向自定义的日志文件:

0 6 \* \* * $HOME/fro\_crontab/createTomorrowTables>>$HOME/for\_crontab/mylog.log 2 >&1

FreeBSD下cron日志文件为 /var/log/cron.

对于crontab的详细介绍请参考: