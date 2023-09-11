---
title: CentOS(RedHat)安装Adobe Flash Player插件 For firefox全过程
author: admin
type: post
date: 2011-01-11T05:43:21+00:00
url: /archives/7476
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - firefox

---
随便打开一个带Flash的网站，提示需要安装插件，使用firefox自带功能安装失败（图1所示）。
浏览器默认下载安装的插件失败之后，点“手动安装”会自动跳转到Adobe Flash Player下载页面：

或者直接先打开Adobe Flash Player下载页面：

[http://www.adobe.com/shockwave/download/download.cgi?P1_Prod_Version=ShockwaveFlash](http://www.adobe.com/shockwave/download/download.cgi?P1_Prod_Version=ShockwaveFlash)

选择”.rpm For Linux“ 显示并下载：

下载完后执行安装：

> [root@CentOS Desktop]# rpm -ivh flash-plugin-9.0.124.0-release.i386.rpm
>
> Preparing…########################################### [100%]
>
> 1:flash-plugin ########################################### [100%]

[root@CentOS Desktop]#

安装成功后重新重动系统既可。