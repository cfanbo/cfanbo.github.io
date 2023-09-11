---
title: MRTG FOR WINDOWS 安装指南
author: admin
type: post
date: 2009-06-23T12:05:52+00:00
excerpt: |
 MRTG(Multi Router Traffic Grapher)，通常讲是一个监控网络链路流量负载的开源软件，它可以从所有运行SNMP协议的设备上（包括服务器、路由器、交换机等）抓取信息。事实上它不仅可以监控网络设备，任何其它的支持SNMP协议的设备都可以做为MRTG的监控对象，并自动生成包含PNG图形格式的HTML文档，通过HTTP 方式显示给用户。

 官方的安装指导：http://mrtg.cs.pu.edu.tw/doc/mrtg-nt-guide.en.html
url: /archives/1902
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - mrtg
 - 流量监控

---
MRTG(Multi Router Traffic Grapher)，通常讲是一个监控网络链路流量负载的开源软件，它可以从所有运行SNMP协议的设备上（包括服务器、路由器、交换机等）抓取信息。事实上它不仅可以监控网络设备，任何其它的支持SNMP协议的设备都可以做为MRTG的监控对象，并自动生成包含PNG图形格式的HTML文档，通过HTTP 方式显示给用户。

官方的安装指导：http://mrtg.cs.pu.edu.tw/doc/mrtg-nt-guide.en.html

准备安装环境

安装之前，除了MRTG安装程序外，还要下载几个辅助软件。这些软件全部是免费的。
1. 下载MRTG
http://www.mrtg.org

2. 下载ActivePerl
http://www.activestate.com/Products/Download/Download.plex?id=ActivePerl

3. 下载Windows服务安装工具：SERANY.exe 和 INSTSRV.exe
http://www.electrasoft.com/srvany/srvany.htm

安装MRTG

下载了以上软件后就可以开始安装了。事实上在Windows上安装MRTG很简单，因为MRTG是以Perl语言开发的，所以要首先安装一个Perl语言的运行环境出来。

1. 安装ActivePerl

解压ActivePerl的包，在安装目录中找到install.bat文件，运行它即可。在DOS窗口中，安装程序会问一些问题，诸如安装路径、是否要修改环境变量等，可以全部使用缺省设置，一路回车就行了。缺省情况下Perl安装在C:\Perl目录下。完成安装后，打开Windows的环境变量检查一下是否增加了Perl的运行文件路径。

2. 安装MRTG

解压MRTG的包，我用的是MRTG-2.12.2版本。将解压后的目录移到C:\下就行了。

需要注意的地方

(1)、给Windows安装SNMP协议支持
通常由于SNMP是一个建议关闭的协议(因为有安全漏洞)，所以Windows 2003不是缺省安装的。不过MRTG就是要用SNMP协议，有什么办法呢，就装一个吧。在“控制面板->增加/删除程序->Windows 组件安装”中，安装SNMP的组件。(打开”Windows 组件向导”–>在“组件”中，单击“管理和监视工具”（但是不要选中或清除其复选框），然后单击“详细信息”。
选中“简单网络管理协议”复选框，然后单击“确定”。)

(2)、修改SNMP的安全设置

如果被监控的机器上也跑Windows的话，这部分就一定要设置(要在被监控方设置，MRTG所在服务器可以不用设置)，否则永远也收不到SNMP的消息。
打开Services窗口并找到SNMP服务，打开右键菜单，选择属性。在打开的窗口中找到“安全”选项页。在选项页中有两部分设置，上半部分是指 SNMP服务接受哪种Community指示字，缺省情况下Windows 2003不对任何指示字反馈。我一般都设为“public–READ ONLY”。下半部分可以设置可信任的主机名、IP或是IPX名称。

(3)、修改防火墙

如果你安装了防火墙，要记得打开UDP 161端口，否则也会问题多多。

运行MRTG
好了，总算安装完了。现在可以运行一下MRTG了，看看它的庐山真面目。

打开DOS窗口，首先进入C:\mrtg\bin，然后输入以下命令：

perl cfgmaker public@localhost –global “WorkDir: C:\Inetpub\wwwroot\mrtg” –output mrtg.cfg

这条命令是给MRTG建立一个监控配置文件，监控的对象是localhost，就是本地机器。你也可以用IP地址来代替localhost，或者指向其它的监控主机。(注意:上面这行命令中WorkDir: 与C:盘符之间要有空格!!! 另外C:\Inetpub\wwwroot\mrtg这个目录也可以换成其它目录，不过因为mrtg会在这个工作目录下生成统计图表和网页，所以一般指定为某个站点下的目录，以方便直接从网上查看统计数据)

再键入一个命令：

perl mrtg mrtg.cfg

这个命令会在C:\Inetpub\wwwroot\mrtg目录下建立一些HTML和PNG文件，这些文件就是用户通常看到的流量报表了。

使MRTG成为Windows的服务

SERANY.exe和INSTSRV.exe这两个程序是Windows自带的工具的软件。它们可以把任何一个Windows的应用程序安装成为

Windows的一个服务。

(1)、修改注册表

创建一个文本文件，在文件中写入以下内容，并保存为mrtg.reg文件：

Windows Registry Editor Version 5.00
[HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Services\MRTG\Parameters]
“Application”=”c:\\perl\\bin\\wperl.exe”
“AppParameters”=”c:\\mrtg\\bin\\mrtg –logging=eventlog c:\\mrtg\\bin\\mrtg.cfg”
“AppDirectory”=”c:\\mrtg\\bin\\”

(2)、安装服务

把SERANY.exe,instsrv.exe复制MRTG的安装目录下，键入以下命令：

instsrv MRTG c:\mrtg\bin\srvany.exe

双击mrtg.reg文件，把相关信息注册到注册表中。在“控制面板->管理工具->Services”下运行名为MRTG的服务即可。

默认情况下，每5分钟，mrtg收集一次数据(注意：一定要在bin\mrtg.cfg配置文件最后一行加上RunAsDaemon: yes)