---
title: unable to find a supported device to write the vmware esx server ESXi 3.5 image to 的解决办法
author: admin
type: post
date: 2010-05-07T04:10:22+00:00
url: /archives/3564
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - vmware

---
使用sata 320G硬盘安装vmware esx3.5,在按f11接受许可协议后总是出现 “unable to find a supported device to write the VMware ESXi 3.5 image to .”的错误提示,如图

[![vmware-esxi-3.5](http://blog.haohtml.com/wp-content/uploads/2010/05/vmware-esxi-3.5.jpg)][1]

解决办法：

1、在此界面按ALT+ F1键进入控制台，用户名**root**,密码为空

2、**vi /usr/lib/vmware/installer/Core/TargetFilter.py**
找到 return interface.GetInterfaceType() == ScsiInterface.SCSI\_IFACE\_TYPE_**IDE**这一行，改为return interface.GetInterfaceType() == ScsiInterface.SCSI_IFACE_TYPE_**ISCSI**保存退出3、输入install命令重新安装。4、当回到原来的错误界面后,再按**ALT+ F1键**,按照提示进行安装就不会再出错误提示了。[![](https://blogstatic.haohtml.com//uploads/2023/09/Install_start_screen.jpg)](http://blog.haohtml.com/wp-content/uploads/2010/05/Install_start_screen.jpg)5) Press F11 on the next screen and you should then see your IDE drive as show below. Press Enter to continue and after a few minutes the install should complete and you will be prompted to reboot.[![](https://blogstatic.haohtml.com//uploads/2023/09/Install_IDE_drive.jpg)](http://blog.haohtml.com/wp-content/uploads/2010/05/Install_IDE_drive.jpg)6) After the reboot, you will be able to connect with the VI client and see that ESXi has installed to and created a VMFS partition on your IDE drive.[![](https://blogstatic.haohtml.com//uploads/2023/09/Install_Complete.jpg)](http://blog.haohtml.com/wp-content/uploads/2010/05/Install_Complete.jpg)这种情况只有硬盘为IDE的时候才会出现此问题。 