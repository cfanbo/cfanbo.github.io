---
title: 合并分区时,出现kern.geom.debugflags=16 提示信息的解决办法
author: admin
type: post
date: 2010-12-26T07:16:18+00:00
url: /archives/7190
IM_contentdowned:
 - 1
categories:
 - 服务器

---
在用sysinstall将原来一个硬盘的两个分区要合并在一个分区的时候,按W进行保存的时候,提示信息 “kern.geom.debugflags=16…”之类的信息,此时退出sysinstall模式,回到命令行状态下,扫行以下命令:

\# sysctl kern.geom.debugflags=16
kern.geom.debugflags: 0->16

再用sysinstall命令里的label合并分区即可.