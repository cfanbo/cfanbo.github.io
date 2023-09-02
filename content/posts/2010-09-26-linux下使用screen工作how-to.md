---
title: Linux下使用screen工作How-to
author: admin
type: post
date: 2010-09-25T16:39:13+00:00
url: /archives/5803
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux

---
通过ssh在Linux终端下工作，有一个很烦的事情就是，如果需要执行一个长时间的命令（例如拷贝一个大文件，或者做DDL）时，如果终端意外断开（网络或者别的原因），一般命令就会终止，当然你可以使用nohup命令，这里提供另一个办法：使用[screen][1]。

一般，我们创建一个screen会话，然后连接会话并在会话下工作，这时候，我们可以随时挂起会话，去做别的事情，而且这个挂起的会话会一直在后台执行。而后又可以重新连接会话。下面是一个简单的How-to：

1. How-to

1.1 创建一个screen会话

screen -dmS supu

该命令，创建一个名为supu的会话，当时并不立刻进入会话。

1.2 连入会话

screen -r supu

连入会话后，就可以做任何想做的工作了。

1.3 挂起该终端

如果你在会话中，做了某个需要等很久的操作，或者你需要离开一段时间，这时就需要执行挂起操作了：

(ctrl+a) + D 先按下Ctr+a然后按D键（screen捕获ctrl+a，后面跟一个命令键D，可以通过ctrl+a ?查看更多）

1.4 其他相关

而后，可以重新使用-r参数回到会话；在会话中，用exit可以退出并关闭这个会话；还可以使用screen -ls命令来查看当前的全部会话状态。

2. 一些名词

Attached和Detached

一般screen -ls可以看到多个会话状态，例如：

```

  [admin@my174 ~]$ screen -ls  There are screens on:          22872.supu      (Detached)          18283.pts-3.my174       (Attached)  2 Sockets in /var/run/screen/S-admin.

```

Detached表示会话处于挂起状态，Attached表示有终端在连接会话。

“22872.supu”这是会话名。22872是一个唯一会话ID，后面supu是自定义的会话名，可以使用screen -r 22872等同于screen -r supu。

Enjoy！

参考：[linux 技巧：使用 screen 管理你的远程会话][2] | man screen

来源:

 [1]: http://www.gnu.org/software/screen/
 [2]: http://www.ibm.com/developerworks/cn/linux/l-cn-screen/