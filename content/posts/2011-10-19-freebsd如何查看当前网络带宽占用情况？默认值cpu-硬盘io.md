---
title: FreeBSD如何查看当前网络带宽占用情况？默认值CPU 硬盘IO 虚拟内存命令
author: admin
type: post
date: 2011-10-19T02:33:06+00:00
url: /archives/11776
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - gstat
 - iostat
 - pnpinfo
 - systat
 - tcpdump
 - top

---
systat 能实时查看各种信息
systat -pigs 默认值CPU
systat -iostat 硬盘IO
systat -swap 交换分区
systat -mbufs 网络缓冲区
systat -vmstat 虚拟内存
systat -netstat 网络
systat -icmp ICMP协议
systat -ip IP协议
systat -tcp TCP协议
systat -ifstat 网卡


显示PCI总线设备信息
pciconf -lv
显示内核加载的模块
kldstat -v
显示指定模块
klsdstat -m ipfilter

即插即用设备
pnpinfo

显示设备占用的IRQ和内存地址
devinfo -u

cpu
sysctl -a|grep cpu
sysctl -a|grep sched 查看使用的调度器，我编译的是ULE

虚拟内存
vmstat

硬盘
gstat
systat -iostat
iostat

网卡
ifconfig
systat -ifstat

网络
netstat
sockstat
tcpdump
trafshow
systat -mbufs
systat -icmp
systat -ip
systat -tcp

只是看流量的话，用systat -netstat