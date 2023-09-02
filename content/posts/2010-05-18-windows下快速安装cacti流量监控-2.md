---
title: Windows下快速安装CACTI流量监控
author: admin
type: post
date: 2010-05-18T01:59:20+00:00
url: /archives/3648
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - CACTI

---
独家：公司最近要对几台上架的服务器进行远程监控，需要提出解决方案。前一段时间曾经在CU上见人们都在讨论CACTI，所以就对照网上的教程进行 了CACTI安装调试，但是可能是自己太菜了，在这个过程中遇到了许多问题，在这里把这个过程记录下来，给像我一样的菜鸟。

CACTI是一套PHP程序，它利用SNMPGET采集数据，使用RRDTOOL绘图引擎绘图，RRDTOOL是MRTG的替代者，它们的作者 是一个人。由于RRDTOOL功能过于强大，所以使用起来命令过于复杂，而CACTI就在这时出现了，它是图形界面，使用简单，使不用直接和 RRDTOOL接触。但是它是以SNMP和RRDTOOL为基础的，所以最好深入学习一下NET-SNMP和RRDTOOL的使用。

好了，废话不多说了，我们来看看在Windows下如何安装CACTI吧。正如我前面说的那样，CACTI是一套PHP系统，所以如果说是安装 调试的话最主要的还的PHP环境的建立。其它的RRDTOOL和Net-Snmp简单应用的话只要安装上就可以，不用做太多的设置。

PHP是一套强大的脚本语言，最初只能应用于Linux下面，随着它的发展，已经能够在Windows下使用了。由于它最初是应用于Linux 下的，所以它安装起来不像Windows的其它软件那样简单，需要进行一些必要的配置，这对使惯Windows的人来说可能一时不能适应。它本身是一套脚 本解释引擎，本身并不具有Web服务器的功能，它是以插件的形式和Apache、IIS等Web服务一起工作的。

Mysql是一套开源的强大的数据库系统，最初也是在Linux上应用，现在也可以在Windows下使用，最新版本有安装、设置向导，使用起 来还是很方便的。

**一、Appserv的安装，及PHP的设置**

在上一篇文章里我详细的写了在Winodws下安装设置Apache、PHP、Mysql，但是感觉这样还是太显麻烦，一样一样装，一样一样 设，太烦琐，而现在大多数网站也都是用的Apache+PHP+Mysql，环境都差不多，那么有没有更简单的方法来搭建这个服务器环境呢?答案肯定是有 的，不然也不会有这篇文章了。

通过在网上的搜索，我发现Appserv这个软件，AppServ 是 Windows下PHP 网页架站工具组合包，泰国的作者将一些网络上免费的架站资源重新包装成单一的安装程序，以方便初学者快速完成架站，AppServ 所包含的软件有：Apache、Apache Monitor、PHP、MySQL、PHP-Nuke、phpMyAdmin，目前最新版本是2.5.8。这个软件安装起来非常方便，一路下一步就可以 非常方便的安装完成，而且安装完成后一个Apache+PHP+Mysql的环境就算搭建好了。而且这个工具还安装了PhpMyAdmin这个Mysql 的管理工具，对于菜鸟来说实在是太方便了。在这里主要需要注意的是如果本机默认的80端口已经在使用了，记着把默认的80端口改成没有使用的， 如：8080，还有就是Mysql的登录密码。

由于这个环境是PHP网站的环境，所以我们还要对PHP进行一些必要的设置，让它符合我们的需求，其实主要就是加几个环境变量。我们打开“开 始”-“控制面板”-“系统”-“高级”-“环境变量”。在“系统变量”选项卡里点添加，在弹出的窗口中变量名输入MIBDIRS，变量值输入 C:AppServphp5extrasmibs，确定就可以了;再找到“path”变量，点编辑，在变量值最后加入PHP的搜索路径，就是你的PHP安 装路径和扩展插件路径，这里是C:AppServphp5和C:AppServphp5ext，所以我加入了“; C:AppServphp5; C:AppServphp5ext”。注意不要加双引号，只添加双引号里面的内容就可以了。

我们还要开启PHP对SNMP、GD、Socket的支持，打开c:windowsphp.ini文件，确保 extension=php\_gd2.dll、extension=php\_mysql.dll、 extension=php_snmp.dll、

extension=php_sockets.dll三个选项前面没有分号。

这时我们要重新启动Windows使刚才所做的设置生效。

 **二、安装CACTI**

系统重新启动以后，我们首先要做的就是在Windows安装Net-Snmp，这个工具安装起来也是很方便的，一路下一步就好了，不用做什么设 置，最好是按照Cacti默认的路径安装，这样设置起CACTI来会省不少事，Cacti默认查找Net-Snmp的路径是C:net-snmp，所以我 们最好将它安装在这个目录下。

而RRDTOOL也已经有Windows下的版本的了，我们只要把它解压就可以了，由于CACTI默认的搜索路径是c:rrdtool，所以我 们把它解压到这个目录就可以了。

最后我们只要把CACTI复制到Web服务器的根目录就可以了，我这里是C:AppServwww，所以我把从网上下载到的CACTI解压到了 这个目录下的CACTI目录，然后打开IE输入：http://localhost，点phpMyAdmin Database Manager Version 2.9.2链接，输入Mysql的用户名和密码，进入PhpMyAdmin后，新建一个数据库“cacti”，新建一个用户“cactiuser”，密码 “cactiuser”，给这个用户完全控制“cacti”数据库权限。然后选择导入数据，把C:AppServwwwcacticacti.sql文件 导入到cacti数据库。到此我们就可以打开IE，输入http://localhost/cacti/install.php来对cacti进行一些简 单的设置，主要是路径的设置。

这样我们就安装成功了CACTI，当然我们还需要进行任务计划的设置等，详细的设置请参见我的上一篇文章“在Windows下安装 CACTI”。

最后附上本次所需要软件的下载地址：

AppServ：http://www.onlinedown.net/soft/35753.htm

CACTI：http://www.cacti.net/downloads

RRDTOOL For Windows：http://www.onlinedown.net/soft/35753.htm

Net-Snmp For Windows：

http://sourceforge.net/project/showfiles.php?group\_id=12694&package\_id=162885&release_id=466298

这些都是软件的最新稳定版本。