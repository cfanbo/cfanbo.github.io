---
title: xen、kvm、vmware、hyper-v等虚拟化技术的比较
author: admin
type: post
date: 2011-12-15T06:52:28+00:00
url: /archives/12307
IM_contentdowned:
 - 1
categories:
 - 其它
tags:
 - 虚拟化
 - hyper-v
 - kvm
 - vmware
 - xen

---
最近在实战Xen中，这篇文章是最近在网上看到的，发出来分享一下。

xen和kvm，是开源免费的虚拟化软件。
vmware是付费的虚拟化软件。
hyper-v比较特别，是微软windows 2008 R2附带的虚拟化组件，如果你买了足够的授权，hyper-v（包括hyper-v 2008 core）都可以免费使用。

如果是vmware或hyper-v虚拟windows系统，不管是虚拟化软件本身，还是其中的子系统，都要支付许可费用。
如果是vmware或hyper-v虚拟linux，虚拟化软件本身要支付许可费用，子系统可以用linux来节省成本。
如果是xen或kvm虚拟windows，其中的子系统要支付许可费用。
如果是xen或kvm虚拟linux，那么虚拟化软件本身和其中的子系统无需产生任何费用。

从性能上来讲，虚拟windows，如果都能得到厂商的支持，那么，性能优化可以不用担心。这几款软件全都能达到主系统至少80%以上的性能（磁盘，CPU，网络，内存），这时建议使用hyper-v来虚拟windows，微软自身的产品，虚拟windows是绝对有优势的。如果是虚拟linux，建议首先使用xen，支持linux的半虚拟化，可以直接使用主系统的cpu和磁盘及网络资源，达到较少的虚拟化调度操作，可以达到非常高的性能，但xen操作复杂，维护成本较高。其次我们推荐kvm来虚拟linux，linux本身支持kvm的virtio技术，可以达到少量的虚拟化调度操作，得到较高的系统性能。不推荐使用hyper-v来虚拟linux，太多的不兼容性导致linux基本无法在hyper-v上跑。

如果以上产品我们不打算买厂商支持，其中vmware和hyper-v，是不建议使用的，主要是授权问题。
这时就剩下kvm和xen了，如果虚拟windows，建议使用kvm，我们可以从redhat那里免费拿到针对windows优化过的磁盘和网络的驱动程序，可以达到较高的性能（几乎与hyper-v性能持平）。而xen的windows优化驱动不是那么容易就能拿到的（由于redhat以后不支持xen了，看看novell是否放水了，呵呵，就开放程度上来讲，redhat要好于novell）。

综上所述，
在有授权的情况下，虚拟windows，建议使用hyper-v
在有授权的情况下，虚拟linux，建议使用xen，如考虑到需要降低管理维护和学习成本，建议使用kvm。
在没有授权的情况下，虚拟windows，建议使用KVM
在没有授权的情况下，虚拟linux，建议使用xen，如考虑到需要降低管理维护和学习成本，建议使用kvm。

转载: