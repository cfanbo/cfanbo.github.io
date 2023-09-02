---
title: 基于Ubuntu平台的nagios快速指南
author: admin
type: post
date: 2010-07-19T04:06:41+00:00
url: /archives/4704
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nagios
 - Ubuntu

---

### 4.6.1. 介绍

本指南试图让你通过简单的指令以在20分钟内在Ubuntu平台上通过对Nagios的源程序的安装来监控本地主机。没有讨论更高级的设置项－只是一些基本操作，但这足以使95%的用户启动Nagios。


这些指令在基于Ubuntu6.10(桌面版)的系统下写成的。


**What You’ll End Up With**

如果按照本指南安装，最后将是这样结果：


1. Nagios和插件将安装到/usr/local/nagios

2. Nagios将被配置为监控本地系统的几个主要服务(CPU负荷、磁盘利用率等)

3. Nagios的Web接口是URL是http://localhost/nagios/


### 4.6.2. 所需软件包

确认你安装好的系统上已经安装如下软件包再继续。


1. Apache2

2. GCC编译器与开发库

3. GD库与开发库


可以用

**apt-get** 命令来安装这些软件包，键入命令：


```
sudo apt-get install apache2
sudo apt-get install build-essential
sudo apt-get install libgd2-dev

```

### 4.6.3. 操作过程

**1)建立一个帐号**

切换为root用户


```
sudo -s

```

创建一个名为 **nagios** 的帐号并给定登录口令


```
/usr/sbin/useradd nagios
passwd nagios

```

在Ubuntu服务器版(6.01或更高版本)，创建一个用户组名为 **nagios**(默认是不创建的)。在Ubuntu桌面版上要跳过这一步。


```
/usr/sbin/groupadd nagios
/usr/sbin/usermod -G nagios nagios

```

创建一个用户组名为 **nagcmd** 用于从Web接口执行外部命令。将nagios用户和apache用户都加到这个组中。


```
/usr/sbin/groupadd nagcmd
/usr/sbin/usermod -G nagcmd nagios
/usr/sbin/usermod -G nagcmd www-data

```

**2)下载Nagios和插件程序包**

建立一个目录用以存储下载文件


```
mkdir ~/downloads
cd ~/downloads

```

下载Nagios和Nagios插件的软件包（访问 [http://www.nagios.org/download/](http://www.nagios.org/download/) 站点以获得最新版本），在写本文档时，最新的Nagios的软件版本是3.0rc1，Nagios插件的版本是1.4.11。


```
wget http://osdn.dl.sourceforge.net/sourceforge/nagios/nagios-3.0rc1.tar.gz
wget http://osdn.dl.sourceforge.net/sourceforge/nagiosplug/nagios-plugins-1.4.11.tar.gz

```

**3)编译与安装Nagios**

展开Nagios源程序包


```
cd ~/downloads
tar xzf nagios-3.0rc1.tar.gz
cd nagios-3.0rc1

```

运行Nagios配置脚本并使用先前开设的用户及用户组：


```
./configure --with-command-group=nagcmd

```

编译Nagios程序包源码


```
make all

```

安装二进制运行程序、初始化脚本、配置文件样本并设置运行目录权限


```
make install
make install-init
make install-config
make install-commandmode

```

现在还不能启动Nagios－还有一些要做的…


**4)客户化配置**

样例 [配置文件](http://nagios-cn.sourceforge.net/nagios-cn/configuration.html#config "5.1. 配置概览") 默认安装在这个目录下 **/usr/local/nagios/etc**，这些样例文件可以配置Nagios使之正常运行，只需要做一个简单的修改…


用你擅长的编辑器软件来编辑这个 **/usr/local/nagios/etc/objects/contacts.cfg** 配置文件，更改email地址 **nagiosadmin** 的联系人定义信息中的EMail信息为你的EMail信息以接收报警内容。


```
vi /usr/local/nagios/etc/objects/contacts.cfg
```

**5)配置WEB接口**

安装Nagios的WEB配置文件到Apache的conf.d目录下


```
make install-webconf

```

创建一个 **nagiosadmin** 的用户用于Nagios的WEB接口登录。记下你所设置的登录口令，一会儿你会用到它。


```
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin

```

重启Apache服务以使设置生效。


```
/etc/init.d/apache2 reload

```

**6)编译并安装Nagios插件**

展开Nagios插件的源程序包


```
cd ~/downloads
tar xzf nagios-plugins-1.4.11.tar.gz
cd nagios-plugins-1.4.11

```

编译并安装插件


```
./configure --with-nagios-user=nagios --with-nagios-group=nagios
make
make install

```

**7)启动Nagios**

把Nagios加入到服务列表中以使之在系统启动时自动启动


```
ln -s /etc/init.d/nagios /etc/rcS.d/S99nagios

```

验证Nagios的样例配置文件


```
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

```

如果没有报错，可以启动Nagios服务


```
/etc/init.d/nagios start

```

**8)登录WEB接口**

你现在可以从WEB方式来接入Nagios的WEB接口了，你需要在提示下输入你的用户名( **nagiosadmin**)和口令，你刚刚设置的，这里用系统默认安装的浏览器，用下面这个超链接


```
http://localhost/nagios/

```

点击“服务详情”的引导超链来查看你本机的监视详情。你可能需要给点时间让Nagios来检测你机器上所依赖的服务因为检测需要些时间。


**9)其他的变更**

如果要接收Nagios的EMail警报，需要安装(Postfix)包


```
sudo apt-get install mailx

```

需要编辑Nagios里的EMail通知送出命令，它位于 **/usr/local/nagios/etc/commands.cfg** 文件中，将里面的’/bin/mail’全部替换为’/usr/bin/mail’。一旦设置好需要重启动Nagios以使配置生效。


```
sudo /etc/init.d/nagios restart

```

配置EMail的报警项超出了本文档的内容，指向你的系统档案用网页查找或是到这个站点 [NagiosCommunity.org wiki](http://www.nagioscommunity.org/wiki) 来查找更进一步的信息，以使Ubuntu系统上可以向外部地址发送EMail信息。


摘自:http://nagios-cn.sourceforge.net/nagios-cn/beginning.html