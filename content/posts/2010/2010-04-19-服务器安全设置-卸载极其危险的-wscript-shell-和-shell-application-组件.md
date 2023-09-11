---
title: 服务器安全设置.卸载极其危险的 Wscript.Shell 和 shell.application 组件
author: admin
type: post
date: 2010-04-19T04:30:06+00:00
url: /archives/3435
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - 安全

---

载极其危险的 Wscript.Shell 和 shell.application 组件，这2 个组件的主要作用是asp调用exe程序。

几乎所有正常的网站都用不到，而要黑服务器却几乎都需要调用这个组件来执行操作

运 行：regsvr32 /u c:\winnt\system32\wshom.ocx　即可卸载 Wscript.Shell

运 行：regsvr32 /u c:\winnt\system32\shell32.dll　即可卸载 shell.application

如果是window2000/20003则将winnt改为windows再运行即可