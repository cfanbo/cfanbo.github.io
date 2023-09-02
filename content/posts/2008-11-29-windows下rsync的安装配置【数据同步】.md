---
title: windows下rsync的安装配置【数据同步】
author: admin
type: post
date: 2008-11-29T00:59:14+00:00
excerpt: |
 之前有转载了一篇《rsync中文手册，使用rsync实现网站镜像和备份》，介绍的是Linux下的安装配置，不过使用流程还是一样的。

 rsync的配置环境
 软件平台:windows2003
 软件版本:cwRsync_2.0.10_Installer cwRsync_Server_2.0.10_Installer
 硬件平台:dell2950 cpu1.6G*4 内存:4G 硬盘:1G*6 RAID5

 ===安装===

 在WINDOWS环境下安装rsync要安装服务端和客户端

 服务器端安装:运行cwRsync_Server_2.0.10_Installer
 客户端安装:运行cwRsync _2.0.10_Installe

 安装步骤和安装服务器端是一样的这里就不详细描述
url: /archives/654
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - rsync
 - Windows

---
    之前有转载了一篇 [《rsync中文手册，使用rsync实现网站镜像和备份》](http://www.indang.net/yinyou/2008/92.html)，介绍的是Linux下的安装配置，不过使用流程还是一样的。

**rsync的配置环境**
软件平台:windows2003
软件版本:cwRsync\_2.0.10\_Installer cwRsync\_Server\_2.0.10_Installer
硬件平台:dell2950 cpu1.6G\*4 内存:4G 硬盘:1G\*6 RAID5

**===安装===**

在WINDOWS环境下安装rsync要安装服务端和客户端

服务器端安装:运行cwRsync\_Server\_2.0.10_Installer
客户端安装:运行cwRsync \_2.0.10\_Installe

安装步骤和安装服务器端是一样的这里就不详细描述

**===配置===**
配置和我们在linux下面的配置一样，在安装目录中找到rsync.conf文件进行配置:

Rsync.conf文件:

pid file = /var/run/rsyncd.pid

lock file = /var/run/rsync.lock

log file = /var/log/rsyncd.log

uid = administrator

gid = administrator

use chroot = no

max connections =4

syslog facility = local5

[test]

path =/cygdrive/d/wlk

comment=/cygdrive/d/wlk comment = BACKUP CLIENT IS SOLARIS 8 E250
ignore errors # 可以忽略一些无关的IO错误
read only = yes # 只读
list = no # 不允许列文件
auth users = inburst # 认证的用户名，如果没有这行，

则表明是匿名
secrets file = etc/inburst.pas # 认证文件名

在server端生成一个密码文件etc/inburst.pas

打开记事本

inburst:hack

保存在安装路径下面的etc文件加下面文件明保存为inburst.pas

在服务中把RsyncServer启动，启动类型修改为自动这样服务器端就安装设置好了

**从client端进行测试**
下面这个命令行中-vzrtopg里的v是verbose，z是压缩，r是recursive，topg都是保持文件原有属性如属主、时间的参数。–progress是指显示

出详细的进度情况，–delete是指如果服务器端删除了这一文件，那么客户端也相应把文件删除，保持真正的一致。
后面的中，

    inburst是指定密码文件中的用户名，之后的::inburst这一inburst是模块名，也就是在/etc/rsyncd.conf中自定义的名称。最后的/tmp是备份到本地的目录名。
    在这里面，还可以用-e ssh的参数建立起加密的连接。可以用–password-file=/password/path/file来指定密码文件，这样就可以在脚本中使用而无需交互式地输入验证密码了，这里需要注意的是这份密码文件权限属性要设得只有属主可读。

在客户端运行CMD

rsync -av 10.0.0.16::401 /cygdrive/h/401