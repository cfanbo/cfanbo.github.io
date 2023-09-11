---
title: cat /proc/loadavg 命令详解
author: admin
type: post
date: 2010-12-10T01:24:58+00:00
url: /archives/6860
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - 负载
 - Linux

---
/proc文件系统是一个虚拟的文件系统，不占用磁盘空间，它反映了当前操作系统在内存中的运行情况，查看/proc下的文件可以聊寄到系统的运行状态。

cat /proc/loadavg是查看系统平均负载的命令，输出结果：
0.18 0.26 0.25 2/251 20320

前三个数字是1、5、15分钟内的平均进程数（有人认为是系统负荷的百分比，其实不然，有些时候可以看到200甚至更多）。

第四个值的分子是正在运行的进程数，分母是进程总数，最后一个是最近运行的进程ID号。

这里的平均负载也就是可运行的进程的平均数。

from   proc(5)   manual   page:

/proc/loadavg
The   first   three   fields   in   this   file     are     load     average     figures giving     the   number   of   jobs   in   the   run   queue   (state   R)   or   waiting
for   disk   I/O   (state   D)   averaged   over   1,   5,   and   15   minutes.     They are     the   same   as   the   load   average   numbers   given   by   uptime(1)   and other   programs.     The   fourth   field   consists   of   two   numbers     sepa‐ rated     by   a   slash   (/).     The   first   of   these   is   the   number   of   cur‐ rently     executing       kernel       scheduling       entities       (processes, threads);   this   will   be   less   than   or   equal   to   the   number   of   CPUs.

The   value   after   the   slash   is   the     number     of     kernel     scheduling entities   that   currently   exist   on   the   system.     The   fifth   field   is the   PID   of   the   process   that   was   most     recently     created     on     the system.

php里可以通过这个文件监控服务器现在的状态。

>  if($fp = @fopen(‘/proc/loadavg’, ‘r’)) {
> list($loadaverage) = explode(‘ ‘, fread($fp, 6));
> fclose($fp);
> if($loadaverage > 一个数) {
> header(“HTTP/1.0 503 Service Unavailable”);
> echo ‘server die 囧’;
> exit();
> }
> }
> ?>

有关更多linux管理员常用命令见: [http://blog.haohtml.com/index.php/archives/6830](http://blog.haohtml.com/index.php/archives/6830)