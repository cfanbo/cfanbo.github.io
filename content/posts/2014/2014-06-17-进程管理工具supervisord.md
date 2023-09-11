---
title: 进程管理工具Supervisord
author: admin
type: post
date: 2014-06-17T06:47:50+00:00
url: /archives/15145
categories:
 - 系统架构
tags:
 - crontab
 - Supervisord

---
## Supervisord 简介

上面已经介绍了Go目前是有两种方案来实现他的daemon，但是官方本身还不支持这一块，所以还是建议大家采用第三方成熟工具来管理我们的应用程序，这里我给大家介绍一款目前使用比较广泛的进程管理软件： [Supervisord](http://supervisord.org/)。Supervisord是用Python实现的一款非常实用的进程管理工具。supervisord会帮你把管理的应用程序转成daemon程序，而且可以方便的通过命令开启、关闭、重启等操作，而且它管理的进程一旦崩溃会自动重启，这样就可以保证程序执行中断后的情况下有自我修复的功能。

> 我前面在应用中踩过一个坑，就是因为所有的应用程序都是由Supervisord父进程生出来的，那么当你修改了操作系统的文件描述符之后，别忘记重启Supervisord，光重启下面的应用程序没用。当初我就是系统安装好之后就先装了Supervisord，然后开始部署程序，修改文件描述符，重启程序，以为文件描述符已经是100000了，其实Supervisord这个时候还是默认的1024个，导致他管理的进程所有的描述符也是1024.开放之后压力一上来系统就开始报文件描述符用光了，查了很久才找到这个坑。

## Supervisord安装

Supervisord可以通过`sudo easy_install supervisor`安装，当然也可以通过Supervisord官网下载后解压并转到源码所在的文件夹下执行`setup.py install`来安装。

- 使用easy_install必须安装setuptools打开 [`http://pypi.python.org/pypi/setuptools#files`](http://pypi.python.org/pypi/setuptools#files)，根据你系统的python的版本下载相应的文件，然后执行 `sh setuptoolsxxxx.egg`，(或者执行python setup.py install命令)这样就可以使用easy_install命令来安装Supervisord。


## Supervisord配置

```
echo_supervisord_conf > /etc/supervisord.conf
```

Supervisord默认的配置文件路径为/etc/supervisord.conf，通过文本编辑器修改这个文件，下面是一个示例的配置文件：

```
;/etc/supervisord.conf
[unix_http_server]
file = /var/run/supervisord.sock
chmod = 0777
chown= root:root

[inet_http_server]
# Web管理界面设定
port=*:9001
username = admin
password = yourpassword

[supervisorctl]
; 必须和'unix_http_server'里面的设定匹配
serverurl = unix:///var/run/supervisord.sock

[supervisord]
logfile=/var/log/supervisord/supervisord.log ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=50MB       ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10          ; (num of main logfile rotation backups;default 10)
loglevel=info               ; (log level;default info; others: debug,warn,trace)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
nodaemon=true              ; (start in foreground if true;default false)
minfds=1024                 ; (min. avail startup file descriptors;default 1024)
minprocs=200                ; (min. avail process descriptors;default 200)
user=root                 ; (default is current user, required if root)
childlogdir=/var/log/supervisord/            ; ('AUTO' child log dir, default $TEMP)

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

; 管理的单个进程的配置，可以添加多个program
[program:blogdemon]
command=/data/blog/blogdemon
autostart = true
startsecs = 5
user = root
redirect_stderr = true
stdout_logfile = /var/log/supervisord/blogdemon.log

```

## Supervisord管理

```
/usr/bin/supervisord -c /etc/supervisord.conf
```

Supervisord安装完成后有两个可用的命令行supervisor和supervisorctl，命令使用解释如下：

- supervisord 初始启动Supervisord，启动、管理配置中设置的进程。

- supervisorctl stop programxxx，停止某一个进程(programxxx)，programxxx为[program:blogdemon]里配置的值，这个示例就是blogdemon。

- supervisorctl start programxxx，启动某个进程

- supervisorctl restart programxxx，重启某个进程

- supervisorctl stop all，停止全部进程，注：start、restart、stop都不会载入最新的配置文件。

- supervisorctl reload，载入最新的配置文件，并按新的配置启动、管理所有进程。