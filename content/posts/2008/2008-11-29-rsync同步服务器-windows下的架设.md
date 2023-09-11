---
title: rsync同步服务器 windows下的架设
author: admin
type: post
date: 2008-11-29T00:54:04+00:00
excerpt: |
 sync是linux下优秀的服务器同步备份软件，是个开源项目，用起来感觉非常的好，现在也有很多服务器是windows的，好在rsync也有windows下的版本，否则很多人将无法享受这么好的软件了。

 下面讲下windows下rsync的架设步骤。

 rsync特性简介

 rsync是类unix系统下的数据镜像备份工具，从软件的命名上就可以看出来了——remote sync。它的特性如下：

 1、可以镜像保存整个目录树和文件系统。
 2、可以很容易做到保持原来文件的权限、时间、软硬链接等等。
 3、无须特殊权限即可安装。
 4、优化的流程，文件传输效率高。
 5、可以使用rcp、ssh等方式来传输文件，当然也可以通过直接的socket连接。
 6、支持匿名传输。
 2. 安装
url: /archives/650
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - rsync
 - Windows

---
sync是linux下优秀的服务器同步备份软件，是个开源项目，用起来感觉非常的好，现在也有很多服务器是windows的，好在rsync也有windows下的版本，否则很多人将无法享受这么好的软件了。

下面讲下windows下rsync的架设步骤。

**rsync特性简介**

rsync是类unix系统下的数据镜像备份工具，从软件的命名上就可以看出来了——remote sync。它的特性如下：

1、可以镜像保存整个目录树和文件系统。
2、可以很容易做到保持原来文件的权限、时间、软硬链接等等。
3、无须特殊权限即可安装。
4、优化的流程，文件传输效率高。
5、可以使用rcp、ssh等方式来传输文件，当然也可以通过直接的socket连接。
6、支持匿名传输。
2. 安装
**rsync的配置环境
** 软件平台：windows2003
软件版本：cwRsync\_2.0.10\_Installer cwRsync\_Server\_2.0.10_Installer
硬件平台：dell2950 cpu1.6G\*4 内存:4G 硬盘：1G\*6 RAID5

2． 安装

在WINDOWS环境下安装rsync要安装服务端和客户端

服务器端安装:运行cwRsync\_Server\_2.0.10_Installer
客户端安装:运行cwRsync \_2.0.10\_Installe

安装步骤和安装服务器端是一样的这里就不详细描述

**3. 配置**
配置和我们在linux下面的配置一样，在安装目录中找到rsync.conf文件进行配置：

Rsync.conf文件：

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

在服务中把RsyncServer启动，启动类型修改为自动
这样服务器端就安装设置好了

**从client端进行测试**
下面这个命令行中-vzrtopg里的v是verbose，z是压缩，r是recursive，topg都是保持文件原有属性如属主、时间
的参数。–progress是指显示

出详细的进度情况，–delete是指如果服务器端删除了这一文件，那么客户端也相应把文件删除，保持真正的一致。
后面的中，

inburst是指定密码文件中的用户名，之后的::inburst这一inburst是模块名，也就是在/etc/rsyncd.conf中自定义
的名称。最后的/tmp是备份
到本地的目录名。
在这里面，还可以用-e ssh的参数建立起加密的连接。可以用–password-file=/password/path/file来指定密码文
件，这样就可以在脚本中使

用而无需交互式地输入验证密码了，这里需要注意的是这份密码文件权限属性要设得只有属主可读。

在客户端运行CMD

rsync -av 10.0.0.16::401 /cygdrive/h/401

**常见问题:**
Q：如何通过ssh进行rsync，而且无须输入密码？
A：可以通过以下几个步骤
1. 通过ssh-keygen在server A上建立SSH keys，不要指定密码，你会在~/.ssh下看到identity和identity.pub文件
2. 在server B上的home目录建立子目录.ssh
3. 将A的identity.pub拷贝到server B上
4. 将identity.pub加到~[user b]/.ssh/authorized_keys
5. 于是server A上的A用户，可通过下面命令以用户B ssh到server B上了
e.g. ssh -l userB serverB
这样就使server A上的用户A就可以ssh以用户B的身份无需密码登陆到server B上了。
Q：如何通过在不危害安全的情况下通过防火墙使用rsync?
A：解答如下：
这通常有两种情况，一种是服务器在防火墙内，一种是服务器在防火墙外。
无论哪种情况，通常还是使用ssh，这时最好新建一个备份用户，并且配置sshd仅允许这个用户通过RSA认证方式进入。
如果服务器在防火墙内，则最好限定客户端的IP地址，拒绝其它所有连接。
如果客户机在防火墙内，则可以简单允许防火墙打开TCP端口22的ssh外发连接就ok了。
Q：我能将更改过或者删除的文件也备份上来吗？
A：当然可以：
你可以使用如：rsync -other -options -backupdir = ./backup-2000-2-13 …这样的命令来实现。
这样如果源文件:/path/to/some/file.c改变了，那么旧的文件就会被移到./backup-2000-2-13/path/to/some/file.c，
这里这个目录需要自己
手工建立起来
Q：我需要在防火墙上开放哪些端口以适应rsync？
A：视情况而定
rsync可以直接通过873端口的tcp连接传文件，也可以通过22端口的ssh来进行文件传递，但你也可以通过下列命令改变它的端口：
rsync –port 8730 otherhost:: 或者 rsync -e ‘ssh -p 2002’ otherhost:
Q：我如何通过rsync只复制目录结构，忽略掉文件呢？
A：rsync -av –include ‘\*/’ –exclude ‘\*’ source-dir dest-dir
Q：为什么我总会出现”Read-only file system”的错误呢？
A：看看是否忘了设”read only = no”了
Q：为什么我会出现[‘@ERROR][1]: invalid gid’的错误呢？
A：rsync使用时默认是用uid=nobody;gid=nobody来运行的，如果你的系统不存在nobody组的话，就会出现这样的错误，可以试试gid = nogroup或者其它
Q：绑定端口873失败是怎么回事？
A：如果你不是以root权限运行这一守护进程的话，因为1024端口以下是特权端口，会出现这样的错误。你可以用–port参数来改变。
Q：为什么我认证失败？
A：从你的命令行看来：
你用的是：
> bash$ rsync -a 144.16.251.213::test test
> Password:
> @ERROR: auth failed on module test
> I dont understand this. Can somebody explain as to how to acomplish this.
> All suggestions are welcome.
应该是没有以你的用户名登陆导致的问题，试试rsync -a  test

 [1]: mailto:'@ERROR