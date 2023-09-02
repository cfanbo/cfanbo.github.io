---
title: '[教程]cacti for windows 安装'
author: admin
type: post
date: 2010-07-29T10:18:32+00:00
url: /archives/4869
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - CACTI

---
本文章前提为配置好了apache(iis)+php+mysql这些基本的需要。

安装snmp服务，需要windows光盘或i386目录的文件。在”安全”标签设置好团体字后重新启动snmp服务。主要防火墙对udp 161开放权限一定要做好，不要将161完全暴露在公网上，最好只对特定的IP或子网开放。(可以用netstat -an命令查看udp协议的161端口是否在监听)

**cacti相关软件下载：**

1.cacti下载： [http://www.cacti.net/downloads/](http://www.cacti.net/downloads/)
2.Cygwin下载： [http://www.cygwin.com/setup.exe](http://www.cygwin.com/setup.exe)
3.rrdtool下载(1.2X): [http://www.cacti.net/downloads/rrdtool/win32/](http://www.cacti.net/downloads/rrdtool/win32/)
4.net-snmp下载： [http://sourceforge.net/projects/net-snmp/files/](http://sourceforge.net/projects/net-snmp/files/)(注意下载的是net-snmp binaries的EXE安装文件,这里使用win32下的文件)
5.Spine(原来好像是cactid)下载： [http://www.cacti.net/downloads/spine/packages/Windows/cacti-spine-0.8.7.zip](http://www.cacti.net/downloads/spine/packages/Windows/cacti-spine-0.8.7.zip)

解压后：并编辑配置文件spine.conf里的数据库配置信息．

**开始安装：**

将rrdtool,安装在d:/rrdtool目录里,net-snmp都安装到d:/usr目录里下，Spine安装在D:/cacti-spine-0.8.7e-win32目录里.并全部赋予目录读取运行权限(分别放在独立的文件夹的,以便于管理)；
将cacti 复制到WEB目录下D:/www/cacti/web里.

创建cacti数据库(mysql),并将cacti程序目录的cacti.sql导入库中。
设置cacti配置文件include/config.php,修改对应的数据库连接信息。

设置php.ini

> extension=php_snmp.dll
> extension=php_sockets.dll
>
> safe_mode =Off
> cgi.force_redirect = 0

通过浏览器打开cacti,自动将进入配置区，填写相应的文件的路径。如图：

[![](http://blog.haohtml.com/wp-content/uploads/2010/07/cacti-for-win32.png)][1]

创建计划任务，每5分钟运行d:\PHP\php.exe -q d:\www\cacti\web\poller.php 监控采集

余下的工作就是创建被监控的主机和图形显示菜单，这里就不多说了。

[![](http://blog.haohtml.com/wp-content/uploads/2010/07/cacti-for-win32-1.png)][2]

**说下容易遇到的问题：**

**1.cacti图形不显示**

登入cacti,Console → Settings → General→ RRDTool Utility Version 将它改成RRDTool 1.2x RRDTool Utility Version为1.2X，并赋予C:\WINDOWS\system32\cmd.exe的cacti读取运行权；

**2.cacti图形空白，没有文字**
修改系统默认字体路径**RRDTool Default Font Path**为**c:/windows/fonts/arial.ttf**

**3.执行计划任务报错Cannot find snmp module**

Cannot find module (IP-MIB): At line 0 in (none)
Cannot find module (IF-MIB): At line 0 in (none)
Cannot find module (TCP-MIB): At line 0 in (none)
Cannot find module (UDP-MIB): At line 0 in (none)
Cannot find module (HOST-RESOURCES-MIB): At line 0 in (none)
Cannot find module (SNMPv2-MIB): At line 0 in (none)
Cannot find module (SNMPv2-SMI): At line 0 in (none)
Cannot find module (NOTIFICATION-LOG-MIB): At line 0 in (none)
Cannot find module (UCD-SNMP-MIB): At line 0 in (none)
Cannot find module (UCD-DEMO-MIB): At line 0 in (none)
Cannot find module (SNMP-TARGET-MIB): At line 0 in (none)
Cannot find module (NET-SNMP-AGENT-MIB): At line 0 in (none)
Cannot find module (DISMAN-EVENT-MIB): At line 0 in (none)
Cannot find module (SNMP-VIEW-BASED-ACM-MIB): At line 0 in (none)
Cannot find module (SNMP-COMMUNITY-MIB): At line 0 in (none)
Cannot find module (UCD-DLMOD-MIB): At line 0 in (none)
Cannot find module (SNMP-FRAMEWORK-MIB): At line 0 in (none)
Cannot find module (SNMP-MPD-MIB): At line 0 in (none)
Cannot find module (SNMP-USER-BASED-SM-MIB): At line 0 in (none)
Cannot find module (SNMP-NOTIFICATION-MIB): At line 0 in (none)
Cannot find module (SNMPv2-TM): At line 0 in (none)

设置系统环境变量**MIBDIRS**设为：**C:\cacti\snmp\share\snmp\mibs**

**4.cacti为什么不能监控windows的CPU信息**
windows的cacti功能上还是很弱的，需要第3方插件完成。插件地址：http://www.wtcs.org/informant/download.htm

> 命令:**snmpwalk -v 2c -c public 192.168.0.45 if** 可以采集到那台机ＳerverIP的数据

最后，如果大家监控的主机多的话，多功能有有所要求，请选择在linux主机上安装cacti吧。主要他无论监控linux还是windows效果都是很好的。

如果要监控其它windows服务器的话，请参考：

 [1]: http://blog.haohtml.com/wp-content/uploads/2010/07/cacti-for-win32.png
 [2]: http://blog.haohtml.com/wp-content/uploads/2010/07/cacti-for-win32-1.png