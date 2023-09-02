---
title: linux ulimit调优
author: admin
type: post
date: 2010-12-16T07:17:57+00:00
url: /archives/6940
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux
 - ulimit

---
1,说明:
ulimit用于shell启动进程所占用的资源.
2,类别:
shell内建命令
3,语法格式:
ulimit \[-acdfHlmnpsStvw\] \[size\]
**4,参数介绍:**
-H 设置硬件资源限制.
-S 设置软件资源限制.
-a 显示当前所有的资源限制.
-c size:设置core文件的最大值.单位:blocks
-d size:设置数据段的最大值.单位:kbytes
-f size:设置创建文件的最大值.单位:blocks
-l size:设置在内存中锁定进程的最大值.单位:kbytes


-m size:设置可以使用的常驻内存的最大值.单位:kbytes
-n size:设置内核可以同时打开的文件描述符的最大值.单位:n
-p size:设置管道缓冲区的最大值.单位:kbytes
-s size:设置堆栈的最大值.单位:kbytes
-t size:设置CPU使用时间的最大上限.单位:seconds
-v size:设置虚拟内存的最大值.单位:kbytes
5.举例
在Linux下写程序的时候，如果程序比较大，经常会遇到“段错误” （segmentation fault）这样的问题，这主要就是由于Linux系统初始的堆栈大小（stack size）太小的缘故，一般为10M。我一般把stack size设置成256M，这样就没有段错误了！命令为：

> ulimit   -s 262140

如果要系统自动记住这个配置，就编辑/etc/profile文件，在 “ulimit -S -c 0 > /dev/null 2>&1”行下，添加“ulimit   -s 262140”，保存重启系统就可以了

Linux对于每个用户，系统限制其最大进程数。为提高性能，可以根据设备资源情况，
设置各linux 用户的最大进程数，下面我把某linux用户的最大进程数设为10000个：

> ulimit -u 10000

对于需要做许多 socket 连接并使它们处于打开状态的 Java 应用程序而言，
最好通过使用 ulimit -n xx 修改每个进程可打开的文件数，缺省值是 1024。
ulimit -n 4096 将每个进程可以打开的文件数目加大到4096，缺省为1024
其他建议设置成无限制（unlimited）的一些重要设置是：

> 数据段长度：ulimit -d unlimited
> 最大内存大小：ulimit -m unlimited
> 堆栈大小：ulimit -s unlimited
> CPU 时间：ulimit -t unlimited
> 虚拟内存：ulimit -v unlimited

我们公司服务器需要调整ulimit的stack size 参数调整为unlimited 无限，使用ulimit -s unlimited时只能在当时的shell见效，重开一个shell就失效了。。于是得在/etc/profile 的最后面添加ulimit -s unlimited 就可以了，source /etc/profile使修改文件生效。

PS：如果你碰到类似的错误提示:
ulimit: max user processes: cannot modify limit: 不允许的操作
ulimit: open files: cannot modify limit: 不允许的操作

为啥root用户是可以的？普通用户又会遇到这样的问题？
看一下/etc/security/limits.conf大概就会明白。
linux对用户有默认的ulimit限制，而这个文件可以配置用户的硬配置和软配置，硬配置是个上限。
超出上限的修改就会出“不允许的操作”这样的错误。

在limits.conf加上

> *        soft    noproc  10240
> *        hard    noproc  10240
> *        soft    nofile  10240
> *        hard    nofile  10240

就是限制了任意用户的最大线程数和文件数为10240。

=============================================

在Linux下面部署应用的时候，有时候会遇上Socket/File: Can’t open so many files的问题；这个值也会影响服务器的最大并发数，其实Linux是有文件句柄限制的，而且Linux默认不是很高，一般都是1024，生产服务器用其实很容易就达到这个数量。下面说的是，如何通过正解配置来改正这个系统默认值。因为这个问题是我配置Nginx+php5时遇到了，所以我将这篇归纳进nginx+apache篇。

查看方法

我们可以用ulimit -a来查看所有限制值

[root@centos5 ~]# ulimit -a

core file size          (blocks, -c) 0

data seg size           (kbytes, -d) unlimited

max nice                        (-e) 0

file size               (blocks, -f) unlimited

pending signals                 (-i) 4096

max locked memory       (kbytes, -l) 32

max memory size         (kbytes, -m) unlimited

open files                      (-n) 1024

pipe size            (512 bytes, -p) 8

POSIX message queues     (bytes, -q) 819200

max rt priority                 (-r) 0

stack size              (kbytes, -s) 10240

cpu time               (seconds, -t) unlimited

max user processes              (-u) 4096

virtual memory          (kbytes, -v) unlimited

file locks                      (-x) unlimited||<

其中 “open files (-n) 1024 “是Linux操作系统对一个进程打开的文件句柄数量的限制(也包含打开的SOCKET数量，可影响MySQL的并发连接数目)。这个值可用ulimit命令来修改,但ulimit命令修改的数值只对当前登录用户的目前使用环境有效,系统重启或者用户退出后就会失效(在布署Nginx+FastCGI我就遇到这个问题，将ulimit -SHn 65535放到/etc/rc.d/rc.local也没起什么作用)

系统总限制是在这里，/proc/sys/fs/file-max。可以通过cat查看目前的值，修改/etc/sysctl.conf 中也可以控制。

另外还有一个，/proc/sys/fs/file-nr，可以看到整个系统目前使用的文件句柄数量。

查找文件句柄问题的时候，还有一个很实用的程序lsof。可以很方便看到某个进程开了那些句柄，也可以看到某个文件/目录被什么进程占用了。

**修改方法**

若要令修改ulimits的数值永久生效，则必须修改配置文档，可以给ulimit修改命令放入/etc/profile里面，这个方法实在是不方便，还有一个方法是修改/etc/sysctl.conf。我修改了，测试过，但对用户的ulimits -a 是不会改变的，只是/proc/sys/fs/file-max的值变了。

我认为正确的做法，应该是修改/etc/security/limits.conf

里面有很详细的注释，比如

* soft   nofile   32768

* hard nofile 65536

就可以将文件句柄限制统一改成软32768，硬65536。配置文件最前面的是指domain，设置为星号代表全局，另外你也可以针对不同的用户做出不同的限制。

注意：这个当中的硬限制是实际的限制，而软限制，是warnning限制，只会做出warning；其实ulimit命令本身就有分软硬设置，加-H就是硬，加-S就是软

默认显示的是软限制，如果运行ulimit命令修改的时候没有加上的话，就是两个参数一起改变。

**生效**

因为我平时工作最多的是部署web环境(Nginx+FastCGI外网生产环境和内网开发环境)，重新登陆即可(reboot其实也行)我分别用root和www用户登陆，用ulimit -a分别查看确认，做这之前最好是重启下ssh服务，service sshd restart。

基本用法也可以参考： [http://blog.haohtml.com/archives/9883](http://blog.haohtml.com/archives/9883)