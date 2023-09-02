---
title: windows下实战VPS虚拟服务器架设教程一
author: admin
type: post
date: 2010-05-04T14:57:41+00:00
url: /archives/3548
IM_data:
 - 'a:6:{s:61:"http://www.nopost.cn/attachments/month_0901/2200916142137.jpg";s:61:"http://www.nopost.cn/attachments/month_0901/2200916142137.jpg";s:61:"http://www.nopost.cn/attachments/month_0901/9200916142416.jpg";s:61:"http://www.nopost.cn/attachments/month_0901/9200916142416.jpg";s:61:"http://www.nopost.cn/attachments/month_0901/h200916143153.jpg";s:61:"http://www.nopost.cn/attachments/month_0901/h200916143153.jpg";s:60:"http://www.nopost.cn/attachments/month_0901/z20091614323.jpg";s:60:"http://www.nopost.cn/attachments/month_0901/z20091614323.jpg";s:60:"http://www.nopost.cn/attachments/month_0901/a20091614327.jpg";s:60:"http://www.nopost.cn/attachments/month_0901/a20091614327.jpg";s:61:"http://www.nopost.cn/attachments/month_0901/a200916143338.jpg";s:61:"http://www.nopost.cn/attachments/month_0901/a200916143338.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - vps

---
网络信息技术的飞速发展，使生活、商业都不可缺少的一部分，然而，网站制作信息技术的深层次的使用我们还处在初级阶段！！ 前言：最近VPS很火热！下面将介绍我架设VPS服务器的一点经验！由于水平有限。疏漏的地方，请高手笑过~在开始之前简单介绍下VPS 的概念 .

一。 虚拟专用服务器（VPS）简介
VPS主机（虚拟专用服务器）（”Virtual Private Server”，或简称 “VPS”）是利用虚拟服务器软件(如微软的Virtual Server、VMware的ESX server、SWsoft 的Virtuozzo)在一台物理服务器上创建多个相互隔离的小服务器。这些小服务器（VPS）本身就有自己操作系统，它的运行和管理与独立服务器完全相 同。虚拟专用服务器确保所有资源为用户独享，给用户最高的服务品质保证，让用户以虚拟主机的价格享受到独立主机的服务品质.

二。VPS主机 用途
VPS虚拟服务器技术可以通过多种不同的方式灵活的分配服务器资源，每个虚拟化服务器的资源都可以有很大的不同，可以灵活的满足各种高端 用户的需求。通过在一台服务器上创建10个左右的VPS主机，可以确保每一个梦幻主机的用户独享VPS资源，其运行和管理完全和独立主机相同。VPS主机 可以为高端用户提供安全、可靠、高品质的主机服务。
可以将它用在以下几个方面：
虚拟主机空间：
VPS主机非常 适合为中小企业、小型门户网站、个人工作室、SOHO一族提供网站空间，较大独享资源，安全可靠的隔离保证了用户对于资源的使用和数据的安全。
电子商务平台：
VPS主机与独立服务器的运行完全相同，中小型服务商可以以较低成本，通过梦幻主机建立自己的电子商务、在线交易平台。
ASP应用平台：
VPS主机特有的应用程序模板，可以快速的进行批量部署，再加上独立主机的品质和极低的的成本是中小型企业进行ASP应 用的首选平台。
数据共享平台：
完全的隔离，无与伦比的安全，使得中小企业、专业门户网站可以使用VPS主机提供数据共享、数 据下在服务。对于大型企业来说，可以作为部门级应用平台。
在线游戏平台：
低廉的价格，优秀的品质，独享的资源使得VPS主机 可以作为在线游戏服务器，为广大的互联网用户提供游戏服务。

我这里只是简单介绍，本文侧重的是架设过程！更多请行查询参考网 址：http://baike.baidu.com/view/698769.htm二。准备工作
1. 必要的网络设备（如交换机等）
2.正常运行的主机一台

服务器配置

操作系统： Windows Server 2003 Enterprise Edition SP1

内存：2G以上（推荐4G以上，毕竟虚拟出来的主机相当于真正的主机！这个不能 太少）

CPU：我的是ADM3000+       推荐更高！越高处理越快！
硬盘：剩余空间40G以上！VPS就在硬 盘上。VPS空间的大小，和这个有绝对关系！太小了。VPS就没啥意义了主板：稳定就行，自行配置吧
网卡：2块以上

软件：

1.架设虚拟机软件：VMware Workstation  我用的是6.5 网上下载的地方很多！

2. 安装虚拟机系统的光盘！你要装什么系统就什么光盘！GHOST也行。前提是能光盘启动！

3.光驱（虚拟光驱也行）

其他：外 网访问VPS的话，要2个以上可用（同一网段[根据接入情况定]）的IP地址。1台VPS最起码要1个地址（不需要外网访问这步可以不管）3. 确认服务器安装好系统。配置完毕！能正常上网！
三。上面的工作全部准备好，就开始实战吧！

1.安装VMware Workstation  我这里默认安装！一路下一步

![[原创]  windows下实战VPS虚拟服务器架设教程（一） - NopostBlog - NoPost - NopostBlog](http://www.nopost.cn/attachments/month_0901/2200916142137.jpg)

![[原创]  windows下实战VPS虚拟服务器架设教程（一） - NopostBlog - NoPost - NopostBlog](http://www.nopost.cn/attachments/month_0901/9200916142416.jpg)

![[原创]  windows下实战VPS虚拟服务器架设教程（一） - NopostBlog - NoPost - NopostBlog](http://www.nopost.cn/attachments/month_0901/h200916143153.jpg)

![[原创]  windows下实战VPS虚拟服务器架设教程（一） - NopostBlog - NoPost - NopostBlog](http://www.nopost.cn/attachments/month_0901/z20091614323.jpg)

![[原创]  windows下实战VPS虚拟服务器架设教程（一） - NopostBlog - NoPost - NopostBlog](http://www.nopost.cn/attachments/month_0901/a20091614327.jpg)

![[原创]  windows下实战VPS虚拟服务器架设教程（一） - NopostBlog - NoPost - NopostBlog](http://www.nopost.cn/attachments/month_0901/a200916143338.jpg)

VMware Workstation安装完成2. 现在重启下主机，进行配置（不管有没提示，推荐重启下）