---
title: 'exec: “pkg-config”: executable file not found in %PATH%　的解决办法'
author: admin
type: post
date: 2013-07-23T09:55:10+00:00
url: /archives/14142
categories:
 - 程序开发
tags:
 - zeromq

---
在windows下要用 [golang](http://blog.haohtml.com/tag/golang) 实现操作 [zeromq](http://blog.haohtml.com/tag/zeromq) 消息队列，发现在sublime下进行

 go get -tags zmq_3_x github.com/alecthomas/gozmq

操作的时候，提示

> \# pkg-config –cflags libzmq libzmq libzmq libzmq
> exec: “pkg-config”: executable file not found in %PATH%
> exit status 2

原因是因为没有安装pkg-config．需要手动安装，并设置一下环境变量.pkg-config下载地址： [http://ftp.acc.umu.se/pub/gnome/binaries/win32/dependencies/pkg-config_0.23-3_win32.zip](http://ftp.acc.umu.se/pub/gnome/binaries/win32/dependencies/pkg-config_0.23-3_win32.zip) ( [http://ftp.acc.umu.se/pub/gnome/binaries/win64/dependencies/pkg-config_0.23-2_win64.zip](http://ftp.acc.umu.se/pub/gnome/binaries/win64/dependencies/pkg-config_0.23-2_win64.zip))

如果无法下载，直接打开所在的目录，找到合适的软件包下载．然后将包里bin目录里的pkg-config.exe放在一个目录，在新建一个＂环境变量＂，变量名为 PKG\_CONFIG\_PATH ．值为pkg-config.exe所在的目录即可.

不过我这里好像还有其它的问题．提示错误,暂不知道如何解决.

> \# pkg-config –cflags libzmq libzmq libzmq
> Package libzmq was not found in the pkg-config search path.
> Perhaps you should add the directory containing \`libzmq.pc’
> to the PKG\_CONFIG\_PATH environment variable
> No package ‘libzmq’ found
> Package libzmq was not found in the pkg-config search path.
> Perhaps you should add the directory containing \`libzmq.pc’
> to the PKG\_CONFIG\_PATH environment variable
> No package ‘libzmq’ found
> Package libzmq was not found in the pkg-config search path.
> Perhaps you should add the directory containing \`libzmq.pc’
> to the PKG\_CONFIG\_PATH environment variable
> No package ‘libzmq’ found
> exit status 1
>
> exit status 2