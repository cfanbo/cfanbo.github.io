---
title: rsync在windows与windows服务器之间的同步设置
author: admin
type: post
date: 2008-11-29T00:40:14+00:00
excerpt: |
 一、windows与windows同步
 1.准备两台机器：
 server-----192.168.0.201
 client-----192.168.0.202

 2.下载windows版的rsync工具
 具体软件下载链接我也忘了，不过在google应该可以搜索到。
 我也将它上传到CU上……


 文件: cwRsync_2.0.10_Installer.zip
 大小: 2953KB
 下载: 下载

 文件: cwRsync_Server_2.0.10_Installer.zip
 大小: 2821KB
 下载: 下载

 server端：cwRsync_Server_2.0.10_Installer.zip
 client端：cwRsync_2.0.10_Installer.zip
url: /archives/642
IM_data:
 - 'a:1:{s:42:"http://blog.chinaunix.net/fileicon/zip.gif";s:63:"http://blog.haohtml.com/wp-content/uploads/2009/06/bd67_zip.gif";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - rsync
 - Windows

---

一、windows与windows同步

1.准备两台机器：

server—–192.168.0.201

client—–192.168.0.202

2.下载windows版的rsync工具

具体软件下载链接我也忘了，不过在google应该可以搜索到。

我也将它上传到CU上……

![](http://blog.chinaunix.net/fileicon/zip.gif)
文件:

cwRsync_2.0.10_Installer.zip

大小:

2953KB

下载:
[下载](http://blogimg.chinaunix.net/blog/upfile/070917224721.zip)![](http://blog.chinaunix.net/fileicon/zip.gif)
文件:

cwRsync_Server_2.0.10_Installer.zip

大小:

2821KB

下载:
[下载](http://blogimg.chinaunix.net/blog/upfile/070917224837.zip)

server端：cwRsync_Server_2.0.10_Installer.zip

client端：cwRsync_2.0.10_Installer.zip

3.安装 与配置

SERVER：

(1)安装cwRsync_Server_2.0.10_Installer.zip

在开始程序中打开“start a unix bash shell”程序:

进入一个类似cmd的终端,输入如下命令：

＄/bin/activate-user.sh

输入l

输入administrator

后面全按回来结束

(2)启动opensshd

打开“控制面板”－－＞“管理工具”－－＞“服务”：

找到一个opensshd的服务，启动它

(3)配置rsyncd.conf配置文件

编辑C:\Program Files\cwRsyncServer\rsyncd.conf，内容如下：

use chroot = false

strict modes = false

hosts allow = *

log file = rsyncd.log

pid file = rsyncd.pid

# Module definitions

# Remember cygwin naming conventions : c:\work becomes /cygwin/c/work

[rsync]

path = /cygdrive/f/rsync   (此处路径代表f:\rsync目录)

read only = yes

transfer logging = yes

secrets file = /cygdrive/f/rsyncd.secrets

(4)启动rsync服务

打开“控制面板”－－＞“管理工具”－－＞“服务”：

找到一个RsyncServer的服务，启动它

到此server端配置结束，接下来配置client端 。

CLIENT：

(1)安装client端软件包：cwRsync_2.0.10_Installer.zip

(2)打开cmd，执行如下操作，测试服务端是否正常 启动服务 了：

cd C:\Program Files\cwRsync\bin

telnet 192.168.0.201 22

telnet 192.168.0.201 873

若上述测试成功，此时可执行同步计划：

rsync -vzrtopg –progress –delete 192.168.0.201::rsync /cygdrive/d/test

或者是：

rsync -vzrtopg –progress –delete 192.168.0.201：/cygdrive/d/rsync /cygdrive/d/test

(此时，会提示输入密码，用户名为administrator，密码则为192.168.0.201的管理员登录密码)

至此，安装配置windows到windows间的同步已经OK

如果定时同步server上的文件，可将其加入任务计划中。

二、windows作为server时与linux间的同步

1、准备机器，此时使用windows作为server

server—192.168.0.201 (windows)

client—192.168.0.132 (linux)

2、经过上文的操作，此时可简化操作了

进入linux主机client同步server:

#rsync -vzrtopg –progress –delete 192.168.0.201::rsync /test

三、linux作为server时与windows间的同步

1、准备机器，此时使用linux作为server

server—192.168.0.132 (linux)

client—192.168.0.202 (windows)

2、安装与配置linux主机的rsync

(1)查看linux上是否安装rsync:

#rpm -qa|grep rsync

若无则安装，或者使用tar编译安装

#rpm -ivh rsync-2.6.8-3.1.rpm

(2)打开rsync服务

#chkconfig xinetd on

#chkconfig rsync on

(3)创建 rsyncd.conf 文件

#touch /etc/rsyncd.conf

#vi /etc/rsyncd.conf(内容如下：)

uid = nobody

gid = nobody

max connections = 4

[www]

path = /www

comment = BACKUP WWW

ignore errors

read only = yes

list = no

auth users = wwwuser

hosts allow=192.168.0.202

secrets file = /etc/wwwuser.pass

(4)启动基于xinetd进程的rsync服务

#/etc/init.d/xinetd start

3、配置windows的rsync客户端

(1)安装client端的rsync包

(2)打开cmd,执行同步计划：

cd C:\Program Files\cwRsync\bin

rsync -vzrtopg –progress –delete root@192.168.0.132::www /cygdrive/d/test

(此时须输入root用户的密码，就可进行同步了。)

至此，全部配置完成。

注：

要使用加密的同步，可使用……

rsync -e ‘ssh -p 2002’ -vzrtopg –progress –delete root@192.168.0.132::www /cygdrive/d/test