---
title: 用Rsync从Linux到Windows远程备份
author: admin
type: post
date: 2008-11-29T00:49:02+00:00
excerpt: |
 rsync是Linux系统下的数据镜像备份工具，从软件的命名上就可以看出来了——remote sync。rsync支持大多数的类Unix系统，无论是Linux、Solaris还是BSD上都经过了良好的测试。rsync的最新版本可以从http://rsync.samba.org/rsync/获得。它的特性如下：
 1、可以镜像保存整个目录树和文件系统。
 2、可以很容易做到保持原来文件的权限、时间、软硬链接等等。
 3、无须特殊权限即可安装。
 4、优化的流程，文件传输效率高。
 5、可以使用rcp、ssh等方式来传输文件，当然也可以通过直接的socket连接。
 本文介绍了如何使用rsync服务从Linux到Windows进行远程备份。
 一、配置服务器端
 首先我们需要配置rsync，打开配置文件/etc/xinetd.d/rsyncd.conf（如果没有请创建它），修改相应的配置项，并增加以下内容：
 uid = nobody # 备份以什么身份进行，用户ID
 gid = nobody # 备份以什么身份进行，组ID
 #注意这个用户ID和组ID，如果要方便的话，可以设置成root，这样rsync几乎就可#以读取任何文件和目录了，但是也带来安全隐患。建议设置成只能读取你要备
 #份的目录和文件即可。
url: /archives/644
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - 远程备份
 - Linux
 - rsync
 - Windows

---
rsync是Linux系统下的数据镜像备份工具，从软件的命名上就可以看出来了——remote sync。rsync支持大多数的类Unix系统，无论是Linux、Solaris还是BSD上都经过了良好的测试。rsync的最新版本可以从http://rsync.samba.org/rsync/获得。它的特性如下：
1、可以镜像保存整个目录树和文件系统。
2、可以很容易做到保持原来文件的权限、时间、软硬链接等等。
3、无须特殊权限即可安装。
4、优化的流程，文件传输效率高。
5、可以使用rcp、ssh等方式来传输文件，当然也可以通过直接的socket连接。
本文介绍了如何使用rsync服务从Linux到Windows进行远程备份。
一、配置服务器端
首先我们需要配置rsync，打开配置文件/etc/xinetd.d/rsyncd.conf（如果没有请创建它），修改相应的配置项，并增加以下内容：
uid = nobody # 备份以什么身份进行，用户ID
gid = nobody # 备份以什么身份进行，组ID
#注意这个用户ID和组ID，如果要方便的话，可以设置成root，这样rsync几乎就可#以读取任何文件和目录了，但是也带来安全隐患。建议设置成只能读取你要备
#份的目录和文件即可。
max connections = 4　# 最大连接数为4
[www] # 指定认证的备份模块名
path = /www # 需要备份的目录
comment = BACKUP WWW　 # 注释
ignore errors　# 忽略一些无关的IO错误
read only = yes　# 设置为只读
list = no # 不允许列文件
auth users = wwwuser　 # 认证的用户名，如果没有这行，则表明是匿名
hosts allow=220.122.133.31　 #允许连接服务器的主机IP地址
secrets file = /etc/wwwuser.pass # 认证文件名，用来存放密码
这一段我们修改完成。
注意：如果同时还需要备份其它目录的话，可以直接在配置文件的后面继续增加配置内容，例如：
[database]
path = /var/lib/mysql
……
这样就可以同时备份多个目录了。
然后为备份模块设置密码文件，如上例的密码文件为/etc/wwwuser.pass，使用编辑器创建这个文件，并输入用户名称和密码：
vi /etc/wwwuser.pass
输入以下内容：
wwwuser:123456
这样，为备份模块www的用户wwwuser设置了密码123456。注意，出于安全目的，这个文件的属性必需是只有属主可读，否则rsync将拒绝运行。我们可以设置它的属性为600：
chmod 600 /etc/wwwuser.pass
设置rsync服务在系统启动时自动启动运行，可以通过ntsysv来设置：

[![](http://blog.haohtml.com/wp-content/uploads/2008/11/20080807180632876-282x300.jpg)][1]

最后在服务器端我们需要启动rsync服务：
service xinetd restart
至此，服务器端配置完毕。

二、配置客户端
为了在Windows环境使用rsync工具，我们需要去下载cwRsync工具，这是一个rsync for windows的版本。
下载安装完成之后的目录结构类似下图所示：

[![](http://blog.haohtml.com/wp-content/uploads/2008/11/a1-300x267.jpg)][2]

现在我们可以在Windows环境下运行rsync工具了，举例使用下面的命令连接服务器并开始备份目录和文件：

```
rsync -vzrtopg --progress --delete wwwuser@xx.xx.xx.xx::www .\bak
```

应该可以看到：
password:
要求输入密码的提示，正确输入密码后就应该看到开始备份了。当然，也有可能出现类似下面的错误信息：

应该可以看到：
password:
要求输入密码的提示，正确输入密码后就应该看到开始备份了。当然，也有可能出现类似下面的错误信息：

[![](http://blog.haohtml.com/wp-content/uploads/2008/11/b1-300x82.jpg)](http://blog.haohtml.com/wp-content/uploads/2008/11/b1.jpg)

引起这种错误有几种可能性，一是你没有输入正确的用户名或密码，二是你的服务器端存储密码的文件没有正确的权限，也就是你的密码文件不是类似这样子的权限：-rw——-1 root root
在备份完成之后，我们可以看到类似下图所示的状态：



[![](http://blog.haohtml.com/wp-content/uploads/2008/11/c.jpg)](http://blog.haohtml.com/wp-content/uploads/2008/11/c.jpg)

 [1]: http://blog.haohtml.com/wp-content/uploads/2008/11/20080807180632876.jpg
 [2]: http://blog.haohtml.com/wp-content/uploads/2008/11/a1.jpg