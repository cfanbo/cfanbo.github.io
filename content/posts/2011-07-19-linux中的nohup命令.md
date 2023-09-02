---
title: linux/unix中的nohup命令
author: admin
type: post
date: 2011-07-19T06:10:03+00:00
url: /archives/10507
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nohup

---
Unix/Linux下一般比如想让某个程序在后台运行，很多都是使用 & 在程序结尾来让程序自动运行。比如我们要运行mysql在后台：

/usr/local/mysql/bin/mysqld_safe –user=mysql &

但是加入我们很多程序并不象mysqld一样做成守护进程，可能我们的程序只是普通程序而已，一般这种程序使用 &

结尾，但是如果终端关闭，那么程序也会被关闭。但是为了能够后台运行，那么我们就可以使用nohup这个命令，比如我们有个test.php需要在后台运行，并且希望在后台能够定期运行，那么就使用nohup：

> nohup /root/test.php &

提示：

> [~]$ appending output to nohup.out

嗯，证明运行成功，同时把程序运行的输出信息放到当前目录的 nohup.out 文件中去。

**nohup 命令**

**用途**：LINUX命令用法，不挂断地运行命令。

**语法**：nohup Command \[ Arg … \] \[　& \]

**描述**：nohup 命令运行由 Command 参数和任何相关的 [Arg](http://baike.baidu.com/view/1476704.htm) 参数指定的命令，忽略所有挂断（SIGHUP）信号。在注销后使用 nohup 命令运行后台中的程序。要运行后台中的 nohup 命令，添加 & （ 表示“and”的符号）到命令的尾部。

如果不将 nohup 命令的输出重定向，输出将附加到当前目录的 nohup.out 文件中。如果当前目录的 nohup.out 文件不可写，输出重定向到 $HOME/nohup.out 文件中。如果没有文件能创建或打开以用于追加，那么 Command 参数指定的命令不可调用。如果标准错误是一个终端，那么把指定的命令写给标准错误的所有输出作为标准输出重定向到相同的 [文件描述符](http://baike.baidu.com/view/1303430.htm)。

**退出状态**：该命令返回下列出口值：

126 可以查找但不能调用 Command 参数指定的命令。

127 nohup 命令发生错误或不能查找由 Command 参数指定的命令。

否则，nohup 命令的退出状态是 Command 参数指定命令的退出状态。

**nohup命令及其输出文件**

nohup命令：如果你正在运行一个进程，而且你觉得在退出帐户时该进程还不会结束，那么可以使用nohup命令。该命令可以在你退出帐户/关闭终端之后继续运行相应的进程。nohup就是不挂起的意思( no hang up)。

该命令的一般形式为：nohup command &

**使用nohup命令提交作业**

如果使用nohup命令提交作业，那么在缺省情况下该作业的所有输出都被重定向到一个名为nohup.out的文件中，除非另外指定了输出文件：

> nohup command > myout.file 2>&1 &

在上面的例子中，输出被重定向到myout.file文件中。

使用 jobs 查看任务。

使用 fg %n 关闭。

另外有两个常用的ftp工具ncftpget和ncftpput，可以实现后台的ftp上传和下载，这样我就可以利用这些命令在后台上传和下载文件了。

**nohup使用示例:**

> #nohup ping sina.com &
> [1] 28191
> #nohup: ignoring input and appending output to ‘nohup.out’
>
> #jobs
> [1] +  Running     nohup ping sina.com &
>
> #fg 1
> nohup ping sina.com

直接可以看到执行的命令.

对于jobs,还有bg和fg命令的用法请参考: [http://blog.haohtml.com/archives/10516](http://blog.haohtml.com/archives/10516)