---
title: rsync从linux到linux的文件同步备份
author: admin
type: post
date: 2010-08-07T02:45:25+00:00
url: /archives/5000
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - rsync

---
**一、环境**

需要备份文件的服务器（服务器端）：192.168.1.201 （RHEL 5）

接收备份文件的服务器（客户端）：192.168.1.202 （CENTOS 5）

**二、安装配置**

1.服务器端的配置

A、采用系统默认安装的rsync 编辑/etc/rsyncd.conf文件，如果没有则新建一个。 vi /etc/rsyncd.conf #\[globale] strict modes= yes #check passwd file port= 873 #default port logfile= /var/log/rsyncd.log pidfile= /var/run/rsyncd.pid max connections= 4 #[modules\] \[testlink\] #备份模块 uid= root gid= root path= /usr/local/apache/htdocs/testlink/upload_area #要备份的目录 read only= no host allow= \* auth users= wwyhy secrets file= /etc/rsyncd.scrt [bugfree] #备份模块 uid= root gid= root path= /usr/local/apache/htdocs/bugfree/BugFile #要备份的目录 read only= no host allow= \* auth users= wwyhy secrets file= /etc/rsyncd.scrt [redmine] #备份模块 uid= root gid= root path= /usr/local/redmine-0.8.1/files #要备份的目录 read only= no host allow= * auth users= wwyhy secrets file= /etc/rsyncd.scrt

B、 添加一个密码文件 vi /etc/rsyncd.scrt 内容如下： wwyhy:123456 #(自己设置)

C、改变权限为600 chmod 600 /etc/rsyncd.scrt

D、启动服务(如开有防火墙请允许873端口通过) rsync –daemon –config=/etc/rsyncd.conf &

2.配置客户端 客户端我则自己编译安装的rsync-3.0.3.tar.gz的

A、安装： tar -zxvf rsync-3.0.3.tar.gz cd rsync-3.0.3 ./configure make make install B、添加密码文件 vi /etc/rsyncd.scrt (没有就新建) 内容如下： wwyhy:123456 (文件与客户端文件内容一样) C、改文件权限为600 chmod 600 /etc/rsyncd.scrt

**三、开始备份**

可以在客户端通过man rsync指令来查看备份指令,我们用脚本来自动执行备份 列：rsync -avz –password-file=密码文件路径 username@需要备份的主机IP::备份里的模块名称 接收备份文件的路径 在/root建一个脚本文件 vi backup

添加内容如下： #1.192.168.1.201上的testlink附件备份指令 rsync -avz –password-file=/etc/rsyncd.scrt wwyhy@192.168.1.201::testlink /home/wangwei/testlink/upload_area #2.192.168.1.201上的bugfree附件备份指令 rsync -avz –password-file=/etc/rsyncd.scrt wwyhy@192.168.1.201::bugfree /home/wangwei/bugfree/BugFile #3.192.168.1.201上的redmine附件备份指令 rsync -avz –password-file=/etc/rsyncd.scrt wwyhy@192.168.1.201::redmine /home/wangwei/redmine-0.8.1/files

chmod u+x backup

每晚2.30自动执行 vi /etc/crontab

30 2 \* \* * root /root/backup