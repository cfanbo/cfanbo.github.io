---
title: 'iisapp 命令 弹出 iisschlp.wsc [88,25] 属性值无效 progid'
author: admin
type: post
date: 2011-03-22T02:24:53+00:00
url: /archives/8067
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - iis
 - iisapp

---
在执行iisapp.vbs时，可能会提示如下错误：

Windows Script Component – file://C:WINDOWSsystem32iisschlp.wsc
[88,25] 属性值无效 : progid

不要汗，解决也挺简单。

原因是为了所谓的ASP安全，卸载了 shell.applaction 组件，也就是 wshom.ocx

重新注册即可正常运行 iisapp.vbs

注册命令：

> regsvr32 wshom.ocx

用完以后,可以再把这个组件卸载掉:

> regsvr32 /u /s weboffice.ocx

有关iisapp命令的用法请参考: