---
title: Windows 2000/XP/2003下让APACHE支持ASP
author: admin
type: post
date: 2010-05-05T05:43:13+00:00
url: /archives/3555
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache

---

asp程序还是使用IIS来驱动比较好，怎么说都asp和IIS都是微软的产物，各方面的支持都可以得到保证；而且IASP是JAVA程序，速度会比IIS慢，这里之所以介绍Apache+JDK+IASP支持*.asp,是为大家多提供一条路参考而已！


**1、安装JDK组件支持IASP(如果你的Windows系统中已经安装了JDK，那么可以省略安装JDK)**

**JDK6官方下载地址：**

[http://www.java.net/download/jdk6/6u10/promoted/b32/binaries/jdk-6u10-rc2-bin-b32-windows-i586-p-12_sep_2008.exe](http://www.java.net/download/jdk6/6u10/promoted/b32/binaries/jdk-6u10-rc2-bin-b32-windows-i586-p-12_sep_2008.exe)

**JDK6 API CHM中文参考下载：**

[http://chinesedocument.com/upimg/soft/JDK6API中文参考070114.rar](http://chinesedocument.com/upimg/soft/JDK6API中文参考070114.rar)

**2、安装iASP2.1.01.exe**

**iASP2.1.01.exe下载地址：**

[http://www.china-microsoft.com/html/xitongruanjian/200806/05-53779.html](http://www.china-microsoft.com/html/xitongruanjian/200806/05-53779.html)

按照安装提示做即可。我安装路径为：C:/IASP2101，安装时会提示你JDK的目录，他自己已经找到，下一步即可。


**3、安装完毕后，提示是否现在配置iasp 。当然选择:是。**

**4、配置：**

**第一步：** 代理服务(proxy)选择：instant asp native servlet support

**第二步：** WEB SERVER选择：apache。（可以不管它提示）

**第三步：** 选择apache的配置 文件 ：httpd.conf的位置，请根据自己的安装目录选择，apache版本选择2.X（因为我的是2.2.6版，根据你的apache版本选择）。

**第四步：** proxy配置，如果您有固定ip，添入您的固定ip。如果没有，那就添：127.0.0.1。port: 这是apache与iasp 之间的代理接口，使用默认（9098）即可。

server manager port:远程管理端口，选择默认（9095）即可。


配置完成后，IASP程序会自动在apache的配置文件httpd.conf最后加入了以下语句：


# iASP Setting

LoadModule iasp_module “C:/IASP2101/bin/apache/win32/1.3.20/iasp.dll”

Alias /iasp “C:/IASP2101”

IaspConfig server “C:/IASP2101/properties/server.properties”

IaspConfig rules “C:/IASP2101/properties/rules.properties”


以上语句的功能应该是和安装PHP一样的，为了是Apache支持*.asp程序


**5、重启apache**

执行：开始-> 程序 ->Instant ASP 2.1.01->Install iASP as NT Service 这样，i asp 就被加到了 win2000 server的服务中。


执行：开始-> 程序 ->Instant ASP 2.1.01->Start Instant ASP 这样，i asp 就被立即打开。


配置完毕，apache可以支持 asp 了！


**例子：** index. asp 中写入此句：〈%response.write(金苹果的个人IT博客-htpp://www.goldapple.name/blog！%〉 ，保存到wwwroot的网页根目录。 在浏览 http://localhost/ ，看到 “金苹果的个人IT博客-htpp://www.goldapple.name/blog！” 您的iasp 就安装成功了！