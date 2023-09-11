---
title: Linux的bg和fg命令
author: admin
type: post
date: 2011-07-19T06:16:25+00:00
url: /archives/10516
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - bg
 - fg

---
我们都知道，在 Windows 上面，我们要么让一个程序作为服务在后台一直运行，要么停止这个服务。而不能让程序在前台后台之间切换。而 Linux 提供了 fg 和 bg 命令，让我们轻松调度正在运行的任务。

假设你发现前台运行的一个程序需要很长的时间，但是需要干其他的事情，你就可以用 Ctrl-Z ，挂起这个程序，然后可以看到系统提示（方括号中的是作业号）：

> [1]+ Stopped /root/bin/rsync.sh

然后我们可以把程序调度到后台执行：（bg 后面的数字为作业号）

> #bg 1
> [1]+ /root/bin/rsync.sh &

用 jobs 命令查看正在运行的任务：

> #jobs
> [1]+ Running /root/bin/rsync.sh &

如果想把它调回到前台运行，可以用

> #fg 1
> /root/bin/rsync.sh

这样，你在控制台上就只能等待这个任务完成了。

fg、bg、jobs、&、ctrl + z都是跟系统任务有关的，虽然现在基本上不怎么需要用到这些命令，但学会了也是很实用的.


一。& 最经常被用到
这个用在一个命令的最后，可以把这个命令放到后台执行
二。ctrl + z
可以将一个正在前台执行的命令放到后台，并且暂停
三。jobs
查看当前有多少在后台运行的命令
四。fg
将后台中的命令调至前台继续运行
如果后台中有多个命令，可以用 fg %jobnumber将选中的命令调出，%jobnumber是通过jobs命令查到的后台正在执行的命令的序号(不是pid)
五。bg
将一个在后台暂停的命令，变成继续执行
如果后台中有多个命令，可以用bg %jobnumber将选中的命令调出，%jobnumber是通过jobs命令查到的后台正在执行的命令的序号(不是pid)