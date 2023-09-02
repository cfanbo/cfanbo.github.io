---
title: 利用taskset有效控制cpu资源
author: admin
type: post
date: 2011-07-26T08:23:00+00:00
url: /archives/10666
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - taskset

---
常常感觉系统资源不够用，一台机子上跑了不下3个比较重要的服务，但是每天我们还要在上面进行个备份压缩等处理，网络长时间传输，这在就很影响本就不够用的系统资源；

这个时候我们就可以把一些不太重要的比如copy/备份/同步等工作限定在一颗cpu上，或者是多核的cpu的一颗核心上进行处理，虽然这不一定是最有效的方法，但可以最大程度上利用了有效资源，降低那些不太重要的进程占用cpu资源；

查看系统下cpu信息：

```
#cat /proc/cpuinfo
```

taskset就可以帮我们完成这项工作，而且操作非常简单；

该工具系统默认安装，rpm包名util-linux

```
#taskset --help
taskset (util-linux 2.13-pre7)
usage: taskset [options] [mask | cpu-list] [pid | cmd [args...]]
set or get the affinity of a process

-p, --pid                  operate on existing given pid
-c, --cpu-list            display and specify cpus in list format
-h, --help                 display this help
-v, --version             output version information
```

举例：
1、开启一个只用0标记的cpu核心的新进程(job.sh是你的工作脚本)

```
#taskset -c 0 sh job.sh
```

2、查找现有的进程号，调整该进程cpu核心使用情况（23328举例用的进程号）

```
#taskset -pc 0 23328
```

pid 23328’s current affinity list: 0-3 #0-3表示使用所有4核进行处理
pid 23328’s new affinity list: 0 #调整后改为仅适用0标记单核处理

3、可在top中进行负载check

最后你可以在你的工作脚本中加入该指令来合理利用现有的cpu资源；

来源: