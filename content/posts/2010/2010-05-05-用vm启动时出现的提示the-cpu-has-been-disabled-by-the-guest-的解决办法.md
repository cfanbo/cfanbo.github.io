---
title: 用VM启动时出现的提示The CPU has been disabled by the guest…..的解决办法
author: admin
type: post
date: 2010-05-05T08:03:19+00:00
url: /archives/3561
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - vmware

---
虚拟机在安装linux操作系统时出现：The CPU has been disabled by the guest operating system……..

解决方法是*.vmx文件的最后添加两行:

monitor_control.restrict_backdoor = TRUE

monitor_control.enable_svm = TRUE

就OK了