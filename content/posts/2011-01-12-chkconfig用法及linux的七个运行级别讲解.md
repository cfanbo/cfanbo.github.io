---
title: linux的七个运行级别及chkconfig的用法
author: admin
type: post
date: 2011-01-12T01:12:52+00:00
url: /archives/7488
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux

---
所谓运行级别简单点来说，运行级别就是操作系统当前正在运行的功能级别。级别是从0到6，具有不同的功能。这些级别定义在/ect/inittab文件中。这个文件是init程序寻找的主要文件，最先运行的服务是那些放在/ect/rc.d目录下的文件。

**一、Linux的运行级别：
**
Linux下的7个运行级别：

0:系统停机状态，系统默认运行级别不能设置为0，否则不能正常启动，机器关闭。
1:单用户工作状态，root权限，用于系统维护，禁止远程登陆，就像Windows下的安全模式登录。
2:多用户状态，没有NFS支持。
 **3**:完整的多用户模式，有NFS，登陆后进入控制台命令行模式。
4:系统未使用，保留一般不用，在一些特殊情况下可以用它来做一些事情。例如在笔记本电脑的电池用尽时，可以切换到这个模式来做一些设置。
 **5**:X11控制台，登陆后进入图形GUI模式，XWindow系统。
6:系统正常关闭并重启，默认运行级别不能设为6，否则不能正常启动。运行init6机器就会重启。

**标准的Linux运行级别为3或5**运行级别原理：
1.在目录/etc/rc.d/init.d下有许多服务器脚本程序，一般称为服务(service)
2.在/etc/rc.d下有7个名为rcN.d的目录，对应系统的7个运行级别
3.rcN.d目录下都是一些符号链接文件，这些链接文件都指向init.d目录下的service脚本文件，命名规则为K+nn+服务名或S+nn+服务名，其中nn为两位数字。
4.系统会根据指定的运行级别进入对应的rcN.d目录，并按照文件名顺序检索目录下的链接文件：对于以K(Kill)开头的文件，系统将终止对应的服；对于以S(Start)开头的文件，系统将启动对应的服务
5.查看运行级别用：runlevel
6.进入其它运行级别用：initN，如果init3则进入终端模式，init5则又登录图形GUI模式
7.另外init0为关机，init6为重启系统

标准的Linux运行级别为3或5，如果是3的话，系统就在多用户状态；如果是5的话，则是运行着XWindow系统。不同的运行级别有不同的用处，也应该根据自己的不同情形来设置。例如，如果丢失了root口令，那么可以让机器启动进入单用户状态来设置。在启动后的lilo提示符下输入：
init=/bin/shrw

这样就可以使机器进入运行级别1，并把root文件系统挂为读写。它会路过所有系统认证，让你使用passwd程序来改变root口令，然后启动到一个新的运行级。

**二、chkconfig用法**
chkconfig命令可以用来检查、设置系统的各种服务

使用语法：
chkconfig\[–add\]\[–del\]\[–list\]\[系统服务\]或chkconfig\[–level<等级代号>\]\[系统服务\][on/off/reset]

参数用法：
–add:增加所指定的系统服务，让chkconfig指令得以管理它，并同时在系统启动的叙述文件内增加相关数据。
–del:删除所指定的系统服务，不再由chkconfig指令管理，并同时在系统启动的叙述文件内删除相关数据。
–level<等级代号>:指定读系统服务要在哪一个执行等级中开启或关毕。

使用范例：
chkconfig –list 列出所有的系统服务
chkconfig –add httpd 增加httpd服务
chkconfig –del httpd 删除httpd服务
chkconfig –level httpd 2345 on 把httpd在运行级别为2、3、4、5的情况下都是on（开启）的状态。

chkconfig命令提供了一种简单的方式来设置一个服务的运行级别。例如，为了设置MySQL服务器在运行级别3和4上运行，你必须首先将MySQL添加为受chkconfig管理的服务：

> cp  /usr/local/mysql/share/mysql/mysql.server      /etc/init.d/mysqld
> chkconfig –add mysqld

现在，我们在级别3和5上设定服务为“on”

> chkconfig –level 35 mysqld on

在其他级别上设为off

> chkconfig –level 01246 mysqld off

为了确认你的配置被正确的修改了，我们可以列出服务将会运行的运行级别，如下所示：

> #chkconfig –list mysqld
> mysqld 0:off 1:off 2:off 3:on 4:off 5:on 6:off

本文来自CSDN博客，转载请标明出处： [http://blog.csdn.net/MONKEY_D_MENG/archive/2010/05/10/5573580.aspx](http://blog.csdn.net/MONKEY_D_MENG/archive/2010/05/10/5573580.aspx)