---
title: 如何在FreeBSD下查看CPU信息【FreeBSD CPU Information】
author: admin
type: post
date: 2009-11-11T04:41:07+00:00
excerpt: |
 推荐使用：＃systat -vm 命令

 How do I get more information about CPU under FreeBSD such as CPU Speed and model?

 A. You can use the dmesg utility displays the contents of the system message buffer when FreeBSD comes up. For accuracy I recommend querying /var/run/dmesg.boot file. Usually a snapshot of the buffer contents taken soon after file systems are mounted at startup time and dumped to /var/run/dmesg.boot file.
 Check CPU Speed in FreeBSD

 Type the command at a shell prompt:
 # sysctl -a | egrep -i 'hw.machine|hw.model|hw.ncpu'
url: /archives/2574
IM_contentdowned:
 - 1
categories:
 - 服务器

---

推荐使用： _＃ **systat -vm** 命令_

How do I get more information about CPU under FreeBSD such as CPU Speed and model?

A. You can use the dmesg utility displays the contents of the system message buffer when FreeBSD comes up. For accuracy I recommend querying /var/run/dmesg.boot file. Usually a snapshot of the buffer contents taken soon after file systems are mounted at startup time and dumped to /var/run/dmesg.boot file.

## Check CPU Speed in FreeBSD

Type the command at a shell prompt:
`# sysctl -a | egrep -i 'hw.machine|hw.model|hw.ncpu'`
Sample output:

```
hw.machine: amd64
hw.model: Intel(R) Xeon(R) CPU           X3220  @ 2.40GHz
hw.ncpu: 4
hw.machine_arch: amd64
```

So I’ve Intel Xeon quad core processor running at 2.40GHz speed.

You need to use following commands in association with grep command.

### FreeBSD CPUINFO using dmesg command

Type the following command
`# dmesg | grep -i cpu`
Or directly query /var/run/dmesg.boot file
`# grep -i cpu /var/run/dmesg.boot`
Output:

```
CPU: Dual Core AMD Opteron(tm) Processor 170 (1999.08-MHz 686-class CPU)
FreeBSD/SMP: Multiprocessor System Detected: 2 CPUs
 cpu0 (BSP): APIC ID:  0
 cpu1 (AP): APIC ID:  1
cpu0:  on acpi0
acpi_throttle0:  on cpu0
cpu1:  on acpi0
acpi_throttle1:  on cpu1
SMP: AP CPU #1 Launched!
```

You can also dump more information using sysctl command
`# sysctl -a | grep -i cpu | less`