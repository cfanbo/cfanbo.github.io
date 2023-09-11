---
title: '[教程]freebsd中使用rsync同步文件'
author: admin
type: post
date: 2010-08-07T02:53:40+00:00
url: /archives/5007
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - rsyn

---
rsync是类unix系统下的数据镜像备份工具，从软件的命名上就可以看出来了——remote sync
它的特性如下：
可以镜像保存整个目录树和文件系统。
可以很容易做到保持原来文件的权限、时间、软硬链接等等。
无须特殊权限即可安装。
优化的流程，文件传输效率高。
可以使用rcp、ssh等方式来传输文件，当然也可以通过直接的socket连接。
支持匿名传输，以方便进行网站镜象。

测试环境freebsd6.3 server:192.168.1.3 client:192.168.1.4
 **1、server端配置(备份源服务器)**
 安装rsync
#cd /usr/ports/net/rsync
#make install clean
 安装成功后编辑rsync的配置文件
#vi /usr/local/etc/rsyncd.conf
加入以下内容


[test]                       #rsync区段的设定名称
comment = test rsync backup #注释
path = /var/www/htdocs/      #需要同步的数据所在路径
auth users = test            #连接rsync服务的帐号
uid = nobody                 #采用什么身份进行文件读取
gid = nogroup                #同上，必须是有读取path权限的用户、组
secrets file = /usr/local/etc/rsyncd.secrets #指定存放帐号密码的文件的位置
read only = yes              #只读

创建帐号密码文件
#vi /usr/local/etc/rsyncd.secrets
格式：帐号:密码 （每行一组，帐号和密码用:号分开）
test:123456

保存后需要将权限修改为600，属主为root
#chmod 600 /usr/local/etc/rsyncd.secrets

启动服务：(daemon前面为两个-)
#rsync -4 –daemon

 查看rsync是否正确运行
#sockstat |grep rsync 正常的话会有以下显示
root     rsync      1572 3 dgram -> /var/run/logpriv
root     rsync      1572 4 tcp4   \*:873                 \*:*

**2、client服务器配置(备份目的地服务器)**
安装rsync
#cd /usr/ports/net/rsync
#make install clean

创建存放备份的目录
#makdir /backup
创建存放密码的文件
#vi /usr/local/etc/rsyncd.secrets
只写密码即可
123456

运行备份命令
#rsync -avzp –delete  /backup –password-file=/usr/local/etc/rsyncd.secrets

正常的话rsync即可将server端的/var/www/htdocs下的所有目录和文件同步到client的/backup目录下，这只是rsync最简单的应用，还可以加入crontab中设定定时备份等，另外还有一些高级应用还需要我再仔细研究一下，先写到这里。

**设置主从服务器定定时自动同步**

rsync.sh文件是cron要执行的脚本文件。/usr/local/etc/rsync.secrets是保存主服务器密码的文件。
a> 创建rsync.sh文件
**#vi /usr/rsync.sh**
写入:
**/usr/local/bin/rsync -vzrtopg –-progress test@192.168.1.3::test /backup –-password-file=/usr/local/etc/rsync.secrets**
退出保存。
b> 创建rsync.secrets密码文件roger#vim rsync.secrets写入：1q2w3e退出保存并修改权限：
**#chmod 600 /usr/rsync.secret**
配置cron服务
**#crontab -e**
添加一行：
**\*/1 \* \* \* * /usr/rsync.sh**
//即每分钟同步一次退出保存。设置完毕，以后每隔一分钟将自动进行同步。

**如果要停止服务用:**
#/usr/local/etc/rc.d/rsyncd stop

**PS: 参数说明**

-a, –archive archive mode, equivalent to -rlptgoD
//档案模式
-v, –verbose
//详细模式
-z, –compress compress file data
//压缩文件
-P equivalent to –partial –progress
//显示进度
–delete
This tells rsync to delete any files on the receiving side
that
aren@#t on the sending side.
//保持远程机器的文件同步性
-e ssh use SSH connection
//使用SSH连接,保证数据安全