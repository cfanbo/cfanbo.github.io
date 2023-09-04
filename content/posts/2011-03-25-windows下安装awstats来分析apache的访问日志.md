---
title: '[教程]windows下安装awstats来分析apache的访问日志'
author: admin
type: post
date: 2011-03-24T20:08:31+00:00
url: /archives/8138
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - awstats

---
下面的教程是在windows2003服务器下安装配置的,由于awstats是由perl程序写的,所以需要我们在安装awstats以前,需要安装ActivePerl( [http://www.activestate.com/](http://www.activestate.com/)),这里安装的为 v5.10.1版本.安装路径为d:\perl,记得要启动httpd.conf文件里的LoadModule cgi\_module modules/mod\_cgi.so模块.

**一.下载软件包**

从官方网站( [http://awstats.sourceforge.net/](http://awstats.sourceforge.net/))下载最新的awstats压缩包(也可以下载.exe的安装文件),这里下载的是awstats-7.0.zip压缩包.将其解压到D:\site\awstats-7.0目录里.
**二.初始化配置环境**

打开D:/site/awstats-7.0/tools文件夹,双击执行awstats_configure.pl,根据提示输入自己apache的安装环境和httpd.conf文件所在的位置,以下为我本机的环境,如图所示:

[![](https://blogstatic.haohtml.com//uploads/2023/09/awstats_install_1.png)][1]

[![](https://blogstatic.haohtml.com//uploads/2023/09/awstats_install_2.png)][2]

回车后,提示

[![](https://blogstatic.haohtml.com//uploads/2023/09/awstats_install_3.png)][3]

这里直接输入”n”,我们手动来配置就可以了,接着是两次回车就可以了.

这时,在httpd.conf文件里会自动添加以下配置信息:

> \## Directives to allow use of AWStats as a CGI
> #
> Alias /awstatsclasses “D:/site/awstats-7.0/wwwroot/classes/”
> Alias /awstatscss “D:/site/awstats-7.0/wwwroot/css/”
> Alias /awstatsicons “D:/site/awstats-7.0/wwwroot/icon/”
> ScriptAlias /awstats/ “D:/site/awstats-7.0/wwwroot/cgi-bin/”
>
> #
> \# This is to permit URL access to scripts/files in AWStats directory.
> #
>
> Options None
> AllowOverride None
> Order allow,deny
> Allow from all
>



**三.添加站点配置文件**

修改awstats.pl文件里的perl路径,用记事本打开D:\site\awstats-7.0\wwwroot\cgi-bin\awstats.pl文件,将第一行 **#!/usr/bin/perl** 修改为

> #!d:/perl/bin/perl.exe

不修改没有办法执行的.
添加新站点www.haohtml.com的配置文件,复制awstats.model.conf文件(D:\site\awstats-7.0\wwwroot\cgi-bin),改名为awstats.www.haohtml.com.conf,修改配置文件里的LogFile,SiteDomain,LogType三个指令.这里分析的是web日志,所以修改LogType=W,修改默认的语言为中文,修改语言一项，Lang=”auto”,将“auto”改为”cn“,让awstats以中文方式工作。

> LogType=W
> LogFile=”d:/apache2.2/logs/www/access\_%YYYY\_%mm_%dd.log”
> SiteDomain=”www.haohtml.com”

这里一定要注意日志的格式,要保证让虚拟主机www.haohtml.com的日志文件名格式和这里的格式一样才可以.

另外默认情况下是不允许通过网页直接更新日志分析信息的,这里修改一下AllowToUpdateStatsFromBrowser的值,改为1

> AllowToUpdateStatsFromBrowser=1

默认配置CustomLog的日记格式是common，改为combined，后者是awstats推荐的方式可以用来分析客户端浏览器的类型以及访问来源等。例如：

> CustomLog “|bin/rotatelogs.exe D:/Apache2.2/logs/www/access\_%Y\_%m_%d.log 86400 480” combined

这个日志配置让apache每天生成一个新的日志文件，其中%Y%m%d是年月日。一般修改的文件为httpd.conf和extra/httpd-vhosts.conf两个文件.

重启apache,输入 [http://localhost/awstats/awstats.pl?config=www.haohtml.com](http://localhost/awstats/awstats.pl?config=www.haohtml.com),就可以看到网站日志查看界面了.

[![](https://blogstatic.haohtml.com//uploads/2023/09/awstats.pl_www.haohtml.com_.gif)][4]

如果有添加新站点,只需要按上面的操作再次复制一个,修改一下相关配置信息就可以了.为了管理方面这里提供了另一个方面的管理方法,使用配置文件包含的功能，所以我们可以配置一个通用配置，比如：awstats.common.conf

然后其他站点的配置设置为：可以通过后面的选项覆盖和缺省不一致的配置。

> awstats.bbs.haohtml.com.conf
> **Include “awstats.common.conf”**
> LogFile=”d:/apache2.2/logs/bbs/access\_%YYYY\_%mm_%dd.log”
> SiteDomain=”bbs.haohtml.com”
>
> awstats.www.haohtml.com.conf
> **Include “awstats.common.conf”**
> LogFile=”d:/apache2.2/logs/www/access\_%YYYY\_%mm_%dd.log”
> SiteDomain=”www.haohtml.com”

**四,安全**

一般管理员为了安全起见,是不允许让外面随便查看这些信息的,这里我们需要做一安全设置.

awstats本身并没有对访问进行任何限制，因此我们必须通过apache的机制来实现，在httpd.conf末尾增加配置如下：

>
> Order deny,allow
> AuthType Basic
> AuthName “Restricted Files”
> AuthUserFile conf/awstats_passwd
> require user awstats_admin
>

使用apache自带的工具htpasswd来生成一个用户名和口令

> {apache}/bin/htpasswd -c {apache}/conf/awstats_passwd awstats_admin

重复输入两次密码即可.

重启apache，这样以后每次访问awstats页面都要求输入正确的用户名(awstats_admin)和口令。

[![](http://blog.haohtml.com/wp-content/uploads/2011/03/awstats_apache_htpasswd.gif)][5]

一般情况下配置完成后，我们需要来更新一下日志,在命令行下执行

awstats.pl -config=www.haohtml.com -update

或者通过浏览器打开 [http://www.haohtml.com/awstats/awstats.pl?config=www.haohtml.com](http://www.haohtml.com/awstats/awstats.pl?config=www.haohtml.com) 更新日志(AllowToUpdateStatsFromBrowser=1).

**五．更新日志**

下面我们来设置一下让系统在指定时间点自动来更新日志，这样我们就省去了人工手动来更新日志了.在linux或者Unix下我们一般是用crontab来实现的，在下我们只能利用计划任务来实现此功能了．

创建批处理文件AwstatsUpate.bat，内容为 ：

> D:\site\awstats-7.0\wwwroot\cgi-bin\ awstats.pl -update -config=www.haohtml.com
> D:\site\awstats-7.0\wwwroot\cgi-bin\ awstats.pl -update -config=bbs.haohtml.com
> ……

我们在计划任务里指定在每晚的23:55分来执行此bat文件即可．

这样，AWStats即可使用了，当然，AWStats可以实现很多丰富的功能，要想更灵活地配置，多看看HELP文件，很详细的。



[1]: http://blog.haohtml.com/wp-content/uploads/2011/03/awstats_install_1.png
[2]: http://blog.haohtml.com/wp-content/uploads/2011/03/awstats_install_2.png
[3]: http://blog.haohtml.com/wp-content/uploads/2011/03/awstats_install_3.png
[4]: http://blog.haohtml.com/wp-content/uploads/2011/03/awstats.pl_www.haohtml.com_.gif
[5]: http://blog.haohtml.com/wp-content/uploads/2011/03/awstats_apache_htpasswd.gif