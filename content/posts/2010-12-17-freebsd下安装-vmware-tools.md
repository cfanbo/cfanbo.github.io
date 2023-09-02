---
title: FreeBSD下安装 VMware Tools
author: admin
type: post
date: 2010-12-17T11:34:53+00:00
url: /archives/7014
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - vmware

---

Install VMware Tools in a FreeBSD Guest

Before you begin, make sure the virtual machine is powered on and the guest operating system is running.

To install VMware Tools in a FreeBSD guest

1


On the host, select VM > Install VMware Tools.


If an earlier version of VMware Tools is installed, the menu item is Update VMware Tools. If the current version is installed, the menu item is Reinstall VMware Tools.

2


Make sure the guest operating system is running in text mode.

You cannot install VMware Tools while X is running.

3


On the guest, log in as root.

4


If necessary, mount the VMware Tools virtual CD-ROM image by entering a command similar to the following:


mount /cdrom

Some FreeBSD distributions automatically mount CD-ROMs. If your distribution uses automounting, skip this step.

5


Change to a working directory (for example, /tmp):


cd /tmp

6


Untar the VMware Tools tar file:

tar zxpf /cdrom/vmware-freebsd-tools.tar.gz

7


If necessary, unmount the VMware Tools virtual CD-ROM image:

umount /cdrom

If your distribution uses automounting, you do not need to unmount the image.

8


Run the VMware Tools installer:

cd vmware-tools-distrib

./vmware-install.pl

9


Log out of the root account:

exit

10


(Optional) Start your graphical environment.

11


In an X terminal, to start the VMware User process, enter the following command:

vmware-user

In minimal installations of the FreeBSD 4.5 guest operating system, sometimes VMware Tools does not start.

来源：vmware官方文档