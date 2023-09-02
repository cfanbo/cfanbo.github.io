---
title: centos下安装Nagios服务器监控软件
author: admin
type: post
date: 2010-07-11T14:26:29+00:00
url: /archives/4565
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - nagios

---
nagios可以对服务器进行全面的监控，包括服务（apache、mysql、ntp、dns、disk、qmail和sshd等等）的状态，服务器的状态（up、down等

等）。它是一个完全GPL协议的开源软件包，包含有nagios主程序和它的各个插件，配置非常灵活，可以监视的项目很多，可以自定义shell脚

本进行监控服务，非常适合大型网络。

nagios的包含主动监控和被动监控。

主动检查是通过监控中心的主机发出请求，让运行在远程主机上的nrpe守护进程收集信息，然后报告它，它通过web接口把数据显示在页面上。

 **它的工作原理如下：**

被动监控是当远程被监控主机处于防火墙之内的时候，只有远程主机可以访问到监控中心，防火墙之内可以设置另外一个监控中心，远程监控

中心的nagios收集服务器信息以后，和nsca报告，由naca客户端报告naca的服务器端，然后报告监控中心的nagios，通过web接口显示监控结果。

Nagios是一个监视系统和网络的应用程序。它监视你所指定主机和服务，当监视的内容变好或者变坏时发出警告。Nagios最初是被设计在Linux

平台上运行的，然而现在在其他平台上也运行良好。

 **Nagios的特性包括：
**
监视网络服务（SMTP, POP3, HTTP, NNTP, PING, 等等）

监视主机资源（处理器负载、磁盘空间等）

容许用户开发自己的插件去检查自定义的项目；

通过使用“父主机”，定义网络主机的分层，容许探测主机down掉或者不可到达。

可以定义在主机或服务运行期间，事件发生以后如何处理和解决方式；

自动记录错误日志；

支持冗余监视；

可选web接口，通过web页面查看当前网络状态，提示和报告故障历史，日志文件等；

**Nagios的系统要求：**

Linux、Unix等

apache

GD库（1.63以上）

zlib

pnglib

jpeglib

basic icons

等.

Nagios的安装过程(CENTOS5)

前提:apache服务已经正常运行.

nagios的安装比较简单，复杂的是设置和配置参数的设定。不过你要放松一点，毕竟我们要搞定它，不是吗？那就开始吧：

1：获得最新的安装包，http://www.nagios.org/download

2：以root身份登录服务器，本实验用版本2.5的GZ包

1）nagios，版本2.5：

2）获得nagios插件，版本1.4.3：

3）获得图库文件：

4）NRPE，版本2.5.2

5）NSCA，版本2.6

3：切换到root用户：

sudo su

4：解压缩

tar zxvf nagios-2.5.tar.gz

5：建立运行nagios的用户：

adduser nagios

6：建立安装nagios的文件夹，并使这个文件夹的所有者为nagios:nagios

> mkdir /usr/local/nagios
> chown nagios.nagios /usr/local/nagios

7：确认web服务器的用户

可能会通过web接口执行一些命令，必须确定web服务器以哪个用户运行的，通常为：apache：

> grep “^User” /etc/httpd/conf/httpd.conf

8：建立命令文件组

这个新的组会包括apache的用户和nagios的用户

> groupadd nagcmd
>
> usermod apache -G nagcmd
>
> usermod nagios -G nagcmd
>
> ———————————-
>
> cat /etc/group
>
> nagcmd:*:9007:apache,nagios
>
> ———————————-

8：运行配置脚本并安装nagios

> cd nagios-2.5
>
> ./configure –prefix=/usr/local/nagios –with-gd-lib=/usr/local/lib –with-gd-inc=/usr/local/include
>
> make all
>
> make install
>
> make install-init
>
> make install-commandmode
>
> make install-config

9：安装nagios-plugins

> tar zxvf nagios-plugins-1.4.3.tar.gz
>
> cd nagios-plugins-1.4.3
>
> ./configure –prefix=/usr/local/nagios-plugins
>
> make all
>
> make install

安装完成以后在/usr/local/nagios-plugins-plugins会产生一个libexec的目录，将该目录全部移动到/usr/local/nagios目录下即可。

> mv /usr/local/nagios-plugins-plugins/libexec/ /usr/local/nagios/

10：imagepak-base.tar.gz的安装

> tar –xvzf imagepak-base.tar.gz

解压以后是base目录

> mv base/ /usr/local/nagios/share/images/logos/

**现在开始配置：**

1：配置web接口

> vi /etc/httpd/conf/httpd.conf

添加如下内容：

> ScriptAlias /nagios/cgi-bin /usr/local/nagios/sbin
>
>
>
> Options ExecCGI
>
> AllowOverride None
>
> Order allow,deny
>
> Allow from all
>
> AuthName “Nagios Access”
>
> AuthType Basic
>
> AuthUserFile /usr/local/nagios/etc/htpasswd.users
>
> Require valid-user
>
>
>
> Alias /nagios /usr/local/nagios/share
>
>
>
> Options None
>
> AllowOverride None
>
> Order allow,deny
>
> Allow from all
>
> AuthName “Nagios Access”
>
> AuthType Basic
>
> AuthUserFile /usr/local/nagios/etc/htpasswd.users
>
> Require valid-user
>
>

修改完毕，保存文件，并重启apache：

> /usr/local/apahce2/bin/apachectl restart

2：配置apache的BASIC认证：

生成认证密码：

> #htpasswd –c /usr/local/nagios/etc/htpasswd.users nagios

开始配置nagios：

cd /usr/local/nagios/etc/

在/usr/local/nagios/etc下是nagios的配置模板文件-sample,把.cfg-sample文件全部拷贝成.cfg

例如:cp nagios.cfg-sample nagios.cfg

全部拷贝完成即可.

vi minimal.cfg

注释所有command：

注释的方法是在每一个定义语句前面添加”#“

修改cgi.cfg

修改use_authentication=1为use_authentication=0,即不用验证.不然有一些页面不会显示。

现在检查配置文件是否有语法错误：

> /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

如果正确，会显示以下结果：

Total Warnings: 0

Total Errors: 0

否则，需要根据提示进行修改配置文件。

配置文件等会再弄。

现在启动nagios

> /usr/local/nagios/bin/nagios -d /usr/local/nagios/etc/nagios.cfg

现在打开网页：

> http://localhost/nagios/

一定会让你大吃一惊，呵呵，我的服务器和服务状态都清楚的看到了。

现在我们的nagios中只有一个，那就是它自己，localhost.

———————-

**监测windows 客户端：**

在nagios的libexec下有check\_nt这个插件,它就是用来检查windows机器的服务的,其功能类似于上一章讲的check\_nrpe.不过还需要搭配另外一个软件NSClient,它则类似于NRPE

NSClient与nrpe最大的区别就是:

–被监控机上安装有nrpe,并且还有插件,最终的监控是由这些插件来进行的.当监控主机将监控请求发给nrpe后,nrpe调用插件来完成监控.

–NSClient则不同,被监控机上只安装NSClient,没有任何的插件.当监控主机将监控请求发给NSClient后,NSClient直接完成监控,所有的监控是由NSClient完成的.

这也说明了NSClient的一个很大的问题,不灵活,没有可扩展性.它只能完成自己本身包含的监控操作,不能由一些插件来扩展.好在NSClient已经做的不错了,基本上可以完全满足我们的监控需要.

1）安装NSClient

NSClient++-0.3.7-Win32

2）配置NSC.ini文件

[Settings]

‘allowed\_hosts’选项的注释去掉,并且加上运行nagios的监控主机的IP.如allowed\_hosts=127.0.0.1/32,192.168.0.22 以逗号相隔.

如果写成192.168.0.0/24则表示该子网内的所有机器都可以访问.如果这个地方是空白则表示所有的主机都可以连接上来.注意是[Settings]部分的,因为[NSClient]部分也有这个选项.

必须保证[NSClient]的’port’选项并没有被注释,并且它的值是’12489’,这是NSClient的默认监听端口

3）启动NSCLIENT服务NSClient++

并在cmd里面执行netstat –an可以看到已经开始监听tcp的12489端口,防火墙打开tcp的12489端口。

4）对监控主机的配置

接下来就是要配置监控主机了.与之前的nrpe的过程类似,在监控主机上做的就3件事情

1.安装监控windows的插件(已经默认安装了,check_nt)

2.定义命令

3.定义要监控的项目

定义命令

vi /usr/local/nagios/etc/commands.cfg

增加下面的内容:

########################################################################

#

# 2007.9.6 add by yahoon

# CHECK_NT

# check windows hosts info

#

########################################################################

define command{

command_name    check_nt


command_line    $USER1$/check_nt -H $HOSTADDRESS$ -p 12489  -v $ARG1$ $ARG2$


}

如果NSClient设置了连接需要密码,则应写成如下格式

$USER1$/check_nt -H $HOSTADDRESS$ -p 12489 -s PASSWORD -v $ARG1$ $ARG2$

具体含义参考check_nt命令的用法

增加监控项目

vi /usr/local/nagios/etc/services.cfg

下面这个服务是监控NSClient的版本

define service{

host_name               yahoon


service_description     check-version         check_command           check_nt!CLIENTVERSION


max_check_attempts      5

normal_check_interval   3


retry_check_interval    2

check_period            24×7


notification_interval   10


notification_period     24×7


notification_options    w,u,c,r


contact_groups          sagroup


}

同样的可以增加如下服务(为了篇幅,我只给出最关键的check_command这一项)

1)监控windows服务器运行的时间

check\_command check\_nt!UPTIME

2)监控Windows服务器的CPU负载,如果5分钟超过80%则是warning,如果5分钟超过90%则是critical

check\_command check\_nt!CPULOAD!-l 5,80,90

3)监控Windows服务器的内存使用情况,如果超过了80%则是warning,如果超过90%则是critical.

check\_command check\_nt!MEMUSE!-w 80 -c 90

4)监控Windows服务器C:\盘的使用情况,如果超过80%已经使用则是warning,超过90%则是critical

check\_command check\_nt!USEDDISKSPACE!-l c -w 80 -c 90

注:-l后面接的参数用来指定盘符

5)监控Windows服务器D:\盘的使用情况,如果超过80%已经使用则是warning,超过90%则是critical

check\_command check\_nt!USEDDISKSPACE!-l d -w 80 -c 90

6)监控Windows服务器的W3SVC服务的状态,如果服务停止了,则是critical

check\_command check\_nt!SERVICESTATE!-d SHOWALL -l W3SVC

7)监控Windows服务器的Explorer.exe进程的状态,如果进程停止了,则是critical

check\_command check\_nt!PROCSTATE!-d SHOWALL -l Explorer.exe

**相关教程:**

Nagios监控windows主机教程请参考:

FreeBSD下安装nagios教程: