---
title: Apache 的信息查看模块——Server-Status
author: admin
type: post
date: 2009-05-19T02:23:02+00:00
excerpt: |
 |
 前提:启用httpd.conf配置文件里的两个模块:|
 LoadModule status_module modules/mod_status.so
 LoadModule info_module modules/mod_info.so

 本文我们将讨论使用 mod_status 和 mod_info to 来告诉你目前服务器的工作情况
 我可以得到什么样的信息？
 使用 mod_status，你可以知道谁在你的服务器上看些什么东西，以及有多少人连在Web 服务
url: /archives/1415
IM_data:
 - 'a:1:{s:60:"http://file.cs001.net/files/1/gallery/blog/Server-Status.jpg";s:73:"http://blog.haohtml.com/wp-content/uploads/2009/05/177d_Server-Status.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache

---
前提:启用httpd.conf配置文件里的两个模块:|
**LoadModule status\_module modules/mod\_status.so
LoadModule info\_module modules/mod\_info.so**

本文我们将讨论使用 mod\_status 和 mod\_info to 来告诉你目前服务器的工作情况
 **我可以得到什么样的信息？**
使用 mod_status，你可以知道谁在你的服务器上看些什么东西，以及有多少人连在Web 服务器上。还有其他可能你的客户不关心的信息，但是对于你，一个站点管理员来说，却是十分有用的信息。

**客户喜欢这些资料**
我不知道你的客户都是怎样的人物，但是我的客户喜欢我提供的信息。每天一次的信息还不够，因为到一天结束时才知道就太晚了。所以他们喜欢知道现在正在发生的事情。

**mod\_info 和 mod\_status**
这两个模块可以提供十分有用的信息，而且十分方便。
mod_status 能准确地告诉你，你的服务器正在“想”什么。你可以知道有哪些人在浏览您的网站，有多少子进程在运行，以及这些进程在干吗。

如果你使用缺省方法安装的 Apache 的话，应该已经安装了mod_status ，唯一要做的就是在配置文件(httpd.conf) 中加入下面几行（其实，只要注释掉就可以了）

 **\# mod_status** 服务器状态

SetHandler server-status
Order deny,allow
Deny from all
Allow from .your_domain.com

这个 SetHandler 语句告诉 Apache ，一旦接收到匹配的请求的话（在本例中就是/server-status）不是去寻找对应的文件，而是转去由相应的模块或者CGI 来处理。

mod_status 模块定义了一个处理机 (server-status) 和一个指示节(ExtendedStatus).

在以上的配置中，存取/server-status 资源时，将提供服务器当前活动的报告。

格式如下：
W\___\___\___………………………………………………
……………………………………………………….
……………………………………………………….
……………………………………………………….

W 代表一个正在应答的子进程，_ 表示空闲的子进程在等待进入的连接。每一个点代表一个还没有生成的潜在的子进程。每一个潜在允许使用的服务用这样的一段来表示。

他还同时告诉你，系统自从上次启动以来已经运行了多少时间。如果需要更多的信息，可以打开ExtendedStatus 开关，这个开关缺省是关的。打开这个开关之后，除了以上信息以外，还可以得到一张每一个子进程及其所作工作的列表。
对于每一个子进程而言，你可以得到它的PID ，以及它占用的CPU 时间和已经运行的时间。对于服务器而言，你可以得到服务器启动以后的合计点击数，CPU的利用率以及每分钟点击数，还有传输给客户端的总计字节数。

第三、URL进行访问,这里使用82端口进行访问

访 问状态页面可以每N秒自动刷新一次。

 获得一个面向机器可读的状态文件

[![apache-server-status](http://blog.haohtml.com/wp-content/uploads/2009/05/apache-server-status.jpg)][1]

server-status 的输出中每个字段所代表的意义如下：

**字段                         说明**

Server Version       Apache 服务器的版本。

Server Built         Apache 服务器编译安装的时间。

Current Time        目前的系统时间。

Restart Time         Apache 重新启动的时间。

Parent Server Generation        Apache 父程序 (parent process) 的世代编号，就是 httpd 接收到 SIGHUP 而重新启动的次数。

Server uptime         Apache 启动后到现在经过的时间。

Total accesses         到目前为此 Apache 接收的联机数量及传输的数据量。

CPU Usage           目前 CPU 的使用情形。

_SWSS….            所有 Apache process 目前的状态。每一个字符表示一个程序，最多可以显示 256 个程序的状态。

Scoreboard Key         上述状态的说明。以下为每一个字符符号所表示的意义：

* _：等待连结中。

* S：启动中。

* R：正在读取要求。

* W：正在送出回应。

* K：处于保持联机的状态。

* D：正在查找DNS。

* C：正在关闭连结。

* L：正在写入记录文件。

* G：进入正常结束程序中。

* I：处理闲置。

* .：尚无此程序。

Srv        本程序与其父程序的世代编号。

PID        本程序的process id。

Acc        分别表示本次联机、本程序所处理的存取次数。

M         该程序目前的状态。

CPU        该程序所耗用的CPU资源。

SS         距离上次处理要求的时间。

Req        最后一次处理要求所耗费的时间，以千分之一秒为单位。

Conn       本次联机所传送的数据量。

Child       由该子程序所传送的数据量。

Slot        由该 Slot 所传送的数据量。

Client       客户端的地址。

VHost       属于哪一个虚拟主机或本主机的IP。

Request     联机所提出的要求信息。

**mod_info**
mpd-info 是一个分类的扩展模块。也就是说他本身没有被集成到Apache 里面，你必须手工增加。

mod_info 对客户而言，可能不是很有用，但是对系统管理员而言，却是十分有用的。特别是有很多服务器需要维护的情况下。使用下面的节可以来实现。

SetHandler server-info
Order deny,allow
Deny from all
Allow from .your-domain.com

这个页面显示的启示就是你编译到Apache 里面的东西的列表以及其他针对服务器的各种特性。

如果你输入：[][2] [http://your.server/server-info/](http://your.server/server-info/) 就可以看到服务器内置的模块列表或者通过DSO 加载的模块列表。
这对于安装和配置特定的服务器来说是十分有用的。特别是用来对错误的配置文件查找问题时。

[![apache-server-info](http://blog.haohtml.com/wp-content/uploads/2009/05/apache-server-info.jpg)][3]

好了，这两个模块的基本介绍就到这里了。详细的信息你还是需要自己去琢磨。因为在方便客户的同时，也需要一定的保密措施，需要对这两个模块所显示的信息，限制到特定的人才能使用，所以，还需要使用Deny，Allow 等语句来限制访问权限。

 [1]: /wp-content/uploads/2009/05/apache-server-status.jpg
 [2]: http://your.server/server-info/
 [3]: http://blog.haohtml.com/wp-content/uploads/2009/05/apache-server-info.jpg