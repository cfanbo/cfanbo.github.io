---
title: windows下网络测试的好工具:NTttcp和NetCPS
author: admin
type: post
date: 2011-06-15T03:05:14+00:00
url: /archives/9811
IM_data:
 - 'a:2:{s:47:"http://img.zdnet.com.cn/1/565/li7LNKrCHu9fY.jpg";s:73:"http://blog.haohtml.com/wp-content/uploads/2011/06/5819_li7LNKrCHu9fY.jpg";s:46:"http://img.zdnet.com.cn/1/566/litMkxKAdO6Y.jpg";s:72:"http://blog.haohtml.com/wp-content/uploads/2011/06/484d_litMkxKAdO6Y.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - NetCPS
 - NTttcp

---
网上的免费网络测试工具很多，在这里，我将向大家介绍两款优秀的网络测试工具，可以帮助大家便捷的进行网络性能测试。这两款工具即 NTttcp 和 NetCPS。
——————————————————————————————–

在上次专题文章中，我向大家介绍了优秀的开源软件 iperf，以及如何用它来快速进行网络性能测试。之后我收到了很多TechRepublic读者的反馈，向我推荐或询问同类的优秀软件，因此在本期文章中，我再向各位推荐两款类似的工具软件，即微软的Network Performance Tool (NTttcp)以及 NetChain的 NetCPS.

Microsoft的 NTttcp 测试工具可以说是iperf的强化版本，并且针对Windows环境进行了优化。 NTttcp的优势在于它可以根据网络任务调整CPU使用率，并且支持多核CPU。另外，NTttcp针对不同系统平台拥有不同的版本，如32-bit (x86)版本， 64-bit (x64)版本，以及IA-64版本。使用Windows XP, Windows Server 2003, Windows Vista, 以及 Windows Server 2008 系统的用户都可以免费下载NTttcp 。在安装好NTttcp后首先要做的就是将一些文件重命名。

根据 NTttcp安装包中附带的白皮书，用户在安装好NTttcp后，需要将NTttcp_x86.exe ( 32-bit系统)重命名为NTttcps.exe用来执行发送数据的任务，而将其改名为NTttcpr.exe则是用来接收数据的程序。从这点来看，作为一款工具软件，NTttcp显得还有些不够完善。而iperf这样的工具只需要附加简单的参数即可实现发送或者接收数据的功能。巧合的是，iperf和 NTttcp都是采用5001端口作为测试数据收发端口。

**图A**显示了 NTttcp作为接收端时的运行界面。

**图 A**

[![](http://blog.haohtml.com/wp-content/uploads/2011/06/li7LNKrCHu9fY.jpg)][1]



另一个和iperf一样简便快捷的工具是 NetCPS。NetCPS和iperf的功能基本一致，只是没有Jperf这样的基于Java的GUI界面。虽然 NetCPS 已经有超过10年的历史了，但是我仍然觉得它是Windows环境下一款很不错的命令行形式的网络性能测试工具，而且不需要进行安装即可使用。NetCPS 同样也是通过参数实现客户端和服务器端功能的。

**图B**显示了在一个快速网络中的两台电脑上运行NetCPS的界面。

**图 B**

[![](http://blog.haohtml.com/wp-content/uploads/2011/06/litMkxKAdO6Y.jpg)][2]

与iperf 和 NTttcp相比，NetCPS的性能和便利度介于两者之间，完全可以应对大部分网络性能测试的需求。

 [1]: http://blog.haohtml.com/wp-content/uploads/2011/06/li7LNKrCHu9fY.jpg
 [2]: http://blog.haohtml.com/wp-content/uploads/2011/06/litMkxKAdO6Y.jpg