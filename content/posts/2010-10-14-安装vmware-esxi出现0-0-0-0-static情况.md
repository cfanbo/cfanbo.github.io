---
title: 安装VMware ESXi出现0.0.0.0 (STATIC)情况
author: admin
type: post
date: 2010-10-14T15:19:59+00:00
url: /archives/6093
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ESX
 - vmware

---
看过了《ESXServer 3i Installable》，信心满满的安装，倒也顺利，谁知道安装好了，发现不能修改ip地址。

首先是“Download tools to manage this host from  （static）”这个肯定很奇怪丫。然后F2进入系统选择 “Configure Management Network ” 修改网络 ，发现根本无法修改地址。重装了也不行。

直到看到了 《ESX Server 3i Installable Setup Guide》 看到这段

One or more of the following Ethernet controllers.

Broadcom NetXtreme 570x gigabit controllers

Intel PRO/1000 adapters

唉。是网卡不识别。

还是要先做好功课，再动手安装。