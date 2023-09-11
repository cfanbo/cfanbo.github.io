---
title: varnishncsa（以 NCSA 的格式显示日志）
author: admin
type: post
date: 2011-11-21T05:58:53+00:00
url: /archives/12061
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - varnish
 - varnishncsa

---
**●varnishncsa（以 NCSA 的格式显示日志）**

Author:   Dag-Erling Sm?rgrav

Date: 2010-05-31

Version:  1.0

Manual section: 1

Display varnish logs in apache/NCSA combined log format

**SYNOPSIS**

varnishncsa \[-a\] \[-b\] \[-C\] \[-c\] \[-D\] \[-d\] \[-f\] \[-I regex\] [-i tag]

\[-n varnish_name\] \[-P file\] \[-r file\] \[-V\] \[-w file\] \[-X regex\] [-x tag]

**DESCRIPTION**

Varnishncsa 工具读取共享内存的日志，然后以 apache/NCSA 的格式显示出来。下 面的选项可以用。

-a 当把日志写到文件里时，使用附加，而不是覆盖。

-b 只显示 varnishd 和后端服务器的日志。

-C 匹配正则表达式的时候，忽略大小写差异。

-c 只显示 varnishd 和客户端的日志。

-D 以进程方式运行

-d 在启动过程中处理旧的日志，一般情况下，varnishhist 只会在进程写入日 志后启动。

-f 在日志输出中使用 X-Forwarded-For HTTP 头代替 client.ip。

-I regex 匹配正则表达式的日志，如果没有使用-i 或者-I，那么所有的日志都 会匹配。

-i tag 匹配指定的 tag，如果没有使用-i 或者-I，那么所有的日志都会被匹配。

-n 指定 varnish 实例的名字，用来获取日志，如果没有指定，默认使用主机 名。

-P file   记录 PID 号的文件

-r file   从一个文件读取日志，而不是从共享内存读取。

-w file   把日志写到一个文件里代替显示他们，如果不是用-a 参数就会发生覆盖， 如果 varnishlog 在写日志时，接收到一个 SIGHUP 信号，他会创建一个新的文件， 老的文件可以移走。

-X regex 排除匹配正则表达式的日志。

-x tag 排除匹配 tag 的日志。

**SEE  ALSO**

* varnishd(1)

* varnishhist(1)

* varnishlog(1)

* varnishstat(1)

* varnishtop(1)

**HISTORY**

The  varnishncsa  utility  was  developed  by  Poul-Henning  Kamp  in  cooperation  with Verdens  Gang AS and Linpro AS. This manual page was written by Dag-Erling Sm?rgrav ?.

**COPYRIGHT**

这个文档的版权和 varnish 自身的版权一样，请看 LICENCE。

* Copyright (c) 2006 Verdens Gang AS

* Copyright (c) 2006-2008 Linpro AS

* Copyright (c) 2008-2010 Redpill Linpro AS

* Copyright (c) 2010 Varnish Software AS