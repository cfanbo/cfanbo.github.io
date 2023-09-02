---
title: 有关Apache Server Status和Cacti对Apache的监控
author: admin
type: post
date: 2010-05-18T01:38:33+00:00
url: /archives/3635
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - CACTI

---
我们平时使用apache常常了解他的性能只能使用ps aux|grep httpd|wc -l查看有多少个进程,但处理了多少http的请求我们不清楚,进程是不是在工作,还是在等都不是很明白,要了解apache的性能,我们需要使用 Apache Server Status的模块来详细了解apache工作的怎么样.下面我还介绍使用cacti来监控它.

**一.对Apache Server Status的启用**
对Apache的状态管理的模块是LoadModule status\_module modules/mod\_status.so,所以这个需要有
然后打开下面的配置

> ExtendedStatus On

配置Apache Server Status的权限

> ```
> <location /server-status>
>          SetHandler server-status
>          Order Deny,Allow
>          Deny from all
>          Allow from 60.60.60.60
> </location>
> ```

打开查看的话就使用http://60.60.60.60/server-status来访问，注意VH的apache要设置在一个VH中,不然你 分不清是那个地址来查看看这个信息.但ExtendedStatus不能放在VH中.

上面的链接还可以加个?refresh=N来设置多久自动刷新一次


如下,是我的机器的显示

Apache Server Status for 60.60.60.60


Server Version: Apache/2.2.3 (CentOS)

Server Built: Jan 15 2008 20:33:41


Current Time: Wednesday, 03-Sep-2008 17:17:37 CST

Restart Time: Wednesday, 03-Sep-2008 17:09:44 CST

Parent Server Generation: 9

Server uptime: 7 minutes 53 seconds

Total accesses: 19517 – Total Traffic: 1.4 GB

CPU Usage: u27.78 s2.67 cu0 cs0 – 6.44% CPU load

41.3 requests/sec – 3.0 MB/second – 73.5 kB/request

131 requests currently being processed, 33 idle workers


KKKK_WK.KKKK_KKC_____C_CWKKK_CK_.K_WKK.KK__K_KK.KKK_W_KKCWKKW.K.

KC__KW_KW._KKKKKWCKKK_K.KKCCWKKKW_KW.K.KWC._W.CKKKKK.KKKK_KKC_.K

_K…_K.WC._..KKC._.._..KK__.C..WK.CK.K.WWKCK..KK_.W.K…K..WKCC

..WKKK..K.KK…W.K..W.K.KK..


server-status 的输出中每个字段所代表的意义如下：

字段         说明

Server Version         Apache 服务器的版本。

Server Built         Apache 服务器编译安装的时间。

Current Time         目前的系统时间。

Restart Time         Apache 重新启动的时间。

Parent Server Generation         Apache 父程序 (parent process) 的世代编号，就是 httpd 接收到 SIGHUP 而重新启动的次数。

Server uptime         Apache 启动后到现在经过的时间。

Total accesses         到目前为此 Apache 接收的联机数量及传输的数据量。

CPU Usage         目前 CPU 的使用情形。

_SWSS….         所有 Apache process 目前的状态。每一个字符表示一个程序，最多可以显示 256 个程序的状态。

Scoreboard Key         上述状态的说明。以下为每一个字符符号所表示的意义：


* _：等待连结中。

* S：启动中。

* R： 正在读取要求。

* W：正在送出回应。

* K：处于保持联机的状态。

* D：正在查找 DNS。

* C：正在关闭连结。

* L：正在写入记录文件。

* G：进入正常结束程序中。

* I：处理闲置。

* .：尚无此程序。


Srv         本程序与其父程序的世代编号。

PID         本程序的 process id。

Acc         分别表示本次联机、本程序所处理的存取次数。

M         该程序目前的状态。

CPU         该程序所耗用的 CPU 资源。

SS         距离上次处理要求的时间。

Req         最后一次处理要求所耗费的时间，以千分之一秒为单位。

Conn         本次联机所传送的数据量。

Child         由该子程序所传送的数据量。

Slot         由该 Slot 所传送的数据量。

Client         客户端的地址。

VHost         属于哪一个虚拟主机或本主机的 IP。

Request         联机所提出的要求信息。


相当不错吧.下面正题,怎么在Cacti中监控他的参数.


**二.安装监控插件**

下载模板和脚本 [http://forums.cacti.net/about25227.html&highlight=apachestats](http://forums.cacti.net/about25227.html&highlight=apachestats)

在上面的地址下载一个叫 **ApacheStats08.zip** 的,中间有二个文件,一个处理脚本php的,另一个是xml的文件.

1.其中的 **ss_apache_stats.php** 是脚本文件,它是一个php的文件,放到你的 **cacti/scripts/** 下面.


2.接下来在cacti界面导入 cacti_host_template_webserver_-_apache.xml 这个文件


3.你就可以在cacti中加入这些设置.就不细写了,如下


被监测的apache服务器需要向上面一样,打开mod_status功能，记的设置好权限访问,不然任何人都可以见到可不好哦

所以我的上面的allow是写的cacti的(60.60.60.60你有这么好的ip吗,呵呵)服务器的地址,你也记的改一下你的


本文链接: [http://www.php-oa.com/2008/09/03/apacheserverstatuscacti.html](http://www.php-oa.com/2008/09/03/apacheserverstatuscacti.html "有关Apache Server Status和Cacti对Apache的监控")

可以手动在dos命令符下执行以下命令来检查:


> php d:/www/ss_apache_stats.php 192.168.1.208

如果有输入信息的话,则正常,否则请检查


相关文章:


[http://blog.haohtml.com/archives/3641](http://blog.haohtml.com/archives/3641)