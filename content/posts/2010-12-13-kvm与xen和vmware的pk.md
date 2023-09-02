---
title: KVM与Xen和VMware的PK
author: admin
type: post
date: 2010-12-13T08:36:06+00:00
url: /archives/6878
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - kvm
 - vmware
 - xen

---
KVM和Xen都是linux下的虚拟机软件，不过似乎都说Xen比KVM强大一些，我也试用了一段时间，今天终于在我的HP笔记本上安装了个Xen上的WinXP，感觉似乎比KVM下的是更稳定一些。

不过就配置来说，kvm比xen简单太多了，Xen还必须有个单独的内核，原有的Linux内核是作为模块加载的。Kvm不论是安装win还是linux系统都必须有CPU的支持，而Xen只有安装Win的时候才需要CPU的支持。

就稳定性来说，还是老辣的Xen好的多，KVM下用everest等软件取硬件信息就死机，xen就不会。但是对鼠标的支持就是KVM好一些了，到了虚拟机的屏幕里就直接锁定了，不像Xen有两个鼠标，位置不一样老是漂来漂去的，让我找的烦死了。

这篇文章翻译至KVM的maintainer Avi Kivity的一篇[文章][1]. 文中提到了KVM比ESX和Xen优越的一个地方：既能获得很好的performance,又能解决设备驱动的维护问题。还是有一定的道理。

I/O的性能对一个hypervisor而言至关重要。同时，I/O也是一个很大的维护负担，因为有大量需要被支持的硬件设备，大量的I/O协议，高可用性，以及对这些设备的管理。

VMware选择性能，但是把I/O协议栈放到了hypervisor里面。不幸的是，VMware kernel是专有的，那就意味着VMware不得不开发和维护整个协议栈。那将意味着开发速度会减慢，你的硬件可能要等一段时间才会得到VMware的支持。

Xen选择了可维护这条道路，它将所有的I/O操作放到了Linux guest里面，也就是所谓的domain-0里面。重用Linux来做I/O, Xen的维护者就不用重写整个I/O协议栈了。但不幸的是，这样就牺牲了性能：每一个中断都必需经过Xen的调度，才能切换到domain 0, 并且所有的东西都不得不经过一个附加层的映射。并不是说Xen已经完全解决了可维护性这个问题：Xen domain 0 kernel仍然是古老的Linux 2.6.18（尽管2.6.25也已经可用了。

【sudison注：】现在Xen已经在通过domain 0 pv_ops在解决这个问题了）那KVM是怎么处理的呢？像VMware一样，I/O是被放到hypervisor的上下文来执行的，所以性能上不会有损害。像Xen一样，KVM重用了整个Linux I/O协议栈，所以KVM的用户就自然就获得了最新的驱动和I/O协议栈的改进。

 [1]: http://avikivity.blogspot.com/2008/04/maintainability-vs-performance.html