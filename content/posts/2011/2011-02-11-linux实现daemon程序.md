---
title: linux实现daemon程序
author: admin
type: post
date: 2011-02-11T03:45:07+00:00
url: /archives/7679
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - c语言
 - daemon

---
**国外相关文档:()**

编写Linux系统下Daemon程序的方法步骤

**一、引言 Daemon程序是一直运行的服务端程序，又称为守护进程。**

本文介绍了在Linux下编写Daemon程序的步骤，并给出了例子程序。

**二、Daemon程序简介**

Daemon是长时间运行的进程，通常在系统启动后就运行，在系统关闭时才结束。一般说Daemon程序在后台运行，是因为它没有控制终端，无法和前台的用户交互。Daemon程序一般都作为服务程序使用，等待客户端程序与它通信。我们也把运行的Daemon程序称作守护进程。

**三、Daemon程序编写规则**

编写Daemon程序有一些基本的规则，以避免不必要的麻烦。

1、首先是程序运行后调用fork，并让父进程退出。子进程获得一个新的进程ID，但继承了父进程的进程组ID。

2、调用setsid创建一个新的session，使自己成为新session和新进程组的leader，并使进程没有控制终端(tty)。

3、改变当前工作目录至根目录，以免影响可加载文件系统。或者也可以改变到某些特定的目录。

4、设置文件创建mask为0，避免创建文件时权限的影响。

5、关闭不需要的打开文件描述符。因为Daemon程序在后台执行，不需要于终端交互，通常就关闭STDIN、STDOUT和STDERR。其它根据实际情况处理。

另一个问题是Daemon程序不能和终端交互，也就无法使用printf方法输出信息了。我们可以使用syslog机制来实现信息的输出，方便程序的调试。在使用syslog前需要首先启动syslogd程序，关于syslogd程序的使用请参考它的man page，或相关文档，我们就不在这里讨论了。

**四、一个Daemon程序的例子** 编译运行环境为Redhat Linux 8.0。

我们新建一个daemontest.c程序，文件内容如下：

> #include
> #include
> #include
> #include
> #include
> #include
> #include
>
> int daemon_init(void)
>
> { pid_t pid;
>
> if((pid = fork()) < 0) return(-1);
>
> else if(pid != 0) exit(0); /\* parent exit \*/
>
> /\* child continues \*/
>
> setsid(); /\* become session leader \*/
>
> chdir(“/”); /\* change working directory \*/
>
> umask(0); /\* clear file mode creation mask \*/
>
> close(0); /\* close stdin \*/
>
> close(1); /\* close stdout \*/
>
> close(2); /\* close stderr \*/
>
> return(0);
> }
>
> void sig_term(int signo)
> {
>
> if(signo == SIGTERM)
>
> /\* catched signal sent by kill(1) command \*/
>
> {
> syslog(LOG_INFO, “program terminated.”);
> closelog();
> exit(0);
> }
>
> }
>
> int main(void)
> {
>
> if(daemon_init() == -1)
> {
> printf(“can’t fork selfn”);
> exit(0);
> }
>
> openlog(“daemontest”, LOG\_PID, LOG\_USER);
>
> syslog(LOG_INFO, “program started.”);
>
> signal(SIGTERM, sig_term); /\* arrange to catch the signal \*/
>
> while(1) { sleep(1); /\* put your main program here \*/ }
>
> return(0);
> }

使用如下命令编译该程序： gcc -Wall -o daemontest daemontest.c编译完成后生成名为daemontest的程序，执行./daemontest来测试程序的运行。

> gcc -Wall -o daemontest daemontest.c
> ./daemontest

使用　ps axj 命令可以显示系统中已运行的daemon程序的信息，包括进程ID、session ID、控制终端等内容。

部分显示内容：

> PPID PID PGID SID TTY TPGID STAT UID TIME COMMAND
>
> 1098 1101 1101 1074 pts/1 1101 S 0 0:00 -bash 1 1581 777 777 ? -1 S 500 0:13 gedit 1 1650 1650 1650 ? -1 S 500 0:00 ./daemontest 794 1654 1654 794 pts/0 1654 R 500 0:00

ps axj 从中可以看到daemontest程序运行的进程号为1650。

我们再来看看/var/log/messages文件中的信息： Apr 7 22:00:32 localhost

daemontest[1650]: program started.

显示了我们在程序中希望输出的信息。

我们再使用kill 1650命令来杀死这个进程，/var/log/messages文件中就会有如下的信息：

Apr 7 22:11:10 localhost daemontest[1650]: program terminated.

使用ps axj命令检查，发现系统中daemontest进程已经没有了。
相关文章：

**五、参考资料**

Advanced Programming in the UNIX Environment W.Richard Stevens

[linux](http://www.chinese%3Ca%20class%3D/) university.net/articles/2153.shtml”>http://www.chinese [linux](http://linux.chinaitlab.com/) university.net/articles/2153.shtml

使用 Zope3 技术建立 Python 服务 (Unix 版本)

daemon.py: daemonize function for Python

机房监控之Python变daemon守护进程

Python Daemon(守护进程）

(翻译)Python标准库的threading.Thread类

python多线程程序的中断(信号)处理

以daemon方式运行一个程序

==============

用 Python写 daemon

\[转\]\[Python 一招鲜系列\] 守护进程一招鲜

python 非阻塞读数据