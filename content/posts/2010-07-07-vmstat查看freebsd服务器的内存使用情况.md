---
title: vmstat查看FreeBSD服务器的内存使用情况
author: admin
type: post
date: 2010-07-07T08:02:35+00:00
url: /archives/4465
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - vmstat

---
在FreeBSD里运行vmstat命令执行结果如下：

> \# vmstat
> procs memory page disk faults cpu
> r b w avm fre flt re pi po fr sr ad0 in sy cs us sy id
> 0 2 1 270512 20316 30 0 0 0 26 5 1223 1589 98 593 1 1 99

当然，仅执行一次vmstat命令是无法反映真正的系统情况的。最好使用vmstat t [n]命令，例如 vmstat 5 5,表示在T（5）秒时间内进行N（5）次采样，或者干脆vmstat 1让系统每秒钟执行一次。

下面是对各个参数的详细解释

procs:
r–>在运行的进程数
b–>在等待io的进程数(等待i/o,paging等等)
w–>可以进入运行队列但被替换的进程
memoy（以k为单位，包括虚拟内存和真实内存，正在运行或最近20秒在运行的进程所用的虚拟内存将被视为active）
avm–>活动的虚拟内存
free–>空闲的内存

pages（统计错误页和活动页，每5秒平均一下，以秒为单位给出数值）
flt–>错误页总数
re–>回收的页面
pi–>进入页面数
po–>出页面数
fr–>空余的页面数
sr–>每秒通过时钟算法扫描的页面

disks  显示每秒的磁盘操作（磁盘名字的前两个字母加数字，默认只显示两个磁盘，如果有多的，可以加-n来增加数字或在命令行下把磁盘名都填上。）

fault 显示每秒的中断数
in–>设备中断
sy–>系统中断
cy–>cpu交换

cpu 表示cpu的使用状态
cs–>用户进程使用的时间
sy–>系统进程使用的时间
id–>cpu空闲的时间

另外根据网上各种大大的使用经验，如果 r经常大于 4 ，且id经常少于40，表示cpu的负荷很重。如果pi，po 长期不等于0，表示内存不足。如果disk 经常不等于0， 且在 b中的队列 大于3， 表示 io性能不好。

**Freebsd下面的其他监视工具：**

fstat
gstat
iostat
netstat
nfsstat
pstat
sockstat
systat
vmstat
w
ps
top

**fstat(identify active files)**
fstat -u chifeng
显示用户chifeng的所有打开的文件，-f也可用
%fstat ./sh_tools.txt
USER CMD PID FD MOUNT INUM MODE SZ|DV R/W NAME
chifeng vi 788 3 /home 189028 -rw-r–r– 115 r ./sh_tools.txt

gstat(print statistics about GEOM disks) GEOM(modular disk I/O request transformation framework)
查看所有GEOM磁盘I/O的繁忙程度

**iostat**
%iostat 1
tty ad0 cpu
tin tout KB/t tps MB/s us ni sy in id
125 57 19.26 3 0.05 6 0 1 0 93
查看设备I/O

**netstat**
netstat -m 查看网络资源使用情况
netstat -rn 查看路由表

**nfsstat**
查看nfs网络文件系统的信息

**pstat**
通常使用pstat -s来查看交换设备的当前状态，相当于swapinfo
%**swapinfo**
Device 1K-blocks Used Avail Capacity
/dev/ad0s3b 524288 0 524288 0%
%pstat -T
368/3976 files
0M/512M swap space
%pstat -s
Device 1K-blocks Used Avail Capacity
/dev/ad0s3b 524288 0 524288 0%

**sockstat**
查看系统当前打开的socket列表

**systat**
功能非常的强大，可以使用的选项很多，如下：
:swap
:ip
:pigs
:mbufs
:iostat
:vmstat
:netstat
:icmp
:ip
:tcp
也可以在启动的时候用 -上面的选项来显示。

**vmstat**
%vmstat
procs memory page disk faults cpu
r b w avm fre flt re pi po fr sr ad0 in sy cs us sy id
1 0 0 296168 14948 136 1 1 0 110 7 0 409 0 679 7 1 92

> procs:r表示正在运行状态的进程，b由于等待输入出等情况而处于阻塞状态的进程，w为已经交换到交换空间而短暂休眠的进程。
> memory:avm为最近访问过的虚拟内存数量，fre为空余的虚拟内存数量。单位：千字节
> page:flt发生缺页中断的次数，re为页面多次引用次数，pi为页面交换进内存的数量，po为页面交换出内存的数量，fr为每秒页面释放的数 量，sr为每秒扫描页面的数量。
> disk:我只有一个磁盘，所以就只显示ad0，最多可以显示3个，不过可以用-n来指定。
> faults:in为硬件设备发生的中断次数，sy为系统调用次数，cs为处理器上下文切换速率。
> cpu:us为用户程序占用处理器时间的百分比，sy为系统内核占用处理器时间的百分比，id为处理器空余时间的百分比。

**w(who)**
通常使用他来获得当前系统中正在登陆的用户的信息

**ps**
通常使用ps -ax来查看系统中的全部进程

对于Linux下的vmstat请参考：，和FreeBSD下的vmstat稍微有些不一样的．