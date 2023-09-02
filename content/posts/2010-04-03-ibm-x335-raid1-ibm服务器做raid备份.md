---
title: IBM X335 RAID1-IBM服务器做RAID备份
author: admin
type: post
date: 2010-04-03T07:31:00+00:00
url: /archives/3267
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - RAID

---
如何使用LSI配置RAID1
X服务器中有一些集成LSI SCSI控制器的机型,在开机自检时按CTRL C可以配置两个硬盘的镜像。但是当升级BIOS之后，CTRL C中的一些设置发生了变化，配置方法也较以前的版本有些差异。
新版本的配置步骤：

1．启动服务器，在自检过程中按CTRL C键，进入到菜单（双通道LSI控制器）

2．选择硬盘所在SCSI通道回车

3．选择,回车。

4．发现两个硬盘，选择一个为主盘，在按减号。

5．系统提示按F3保存磁盘的数据，按Delete删除磁盘上的数据。如果这个磁盘上有操作系统，一定要选择F3.完成之后下面的[No]变成[Yes].

6. 在第二个磁盘上[No]的位置按减号：

7．系统警告这个磁盘上的信息会丢失，按DELETE删除这个磁盘上的所有数据，或者按任意键取消。按DELETE，第二个磁盘的[No]也会变成[Yes],重启系统，开始同步磁盘。
旧版本

1） 重启主机 按 CTRL-C 进入配置菜单 ，光标放在第一个通道上，按继续

2） 选择 DEVICE PROPERTIES 可发现硬盘，按回到前一菜单

3） 选择 MIRRORING PROPERTIES 按继续

4） 选择第一块硬盘 ，第三列mirror pair项，按-/+号 将其设为PRIMARY。

5） 选择第二块硬盘 ，第三列mirror pair项，按-/+号 将其设为SECONDE

6） 按ESC ，选择“SAVE CHANGE THEN EXIT THIS MENU”，按继续

7） 按ESC ，选择“EXIT THE CONFIGURATION UTITILY”，按继续

使用RAID的情况，最少你要装ServeRaid这个软件，一般你装最新的就OK了，随机应该有配，没有就去IBM网站上下载。
除了给你看到当前RAID的信息，是否有坏盘，更可以提前给你PFA警告，预测盘是否还在稳定运行。

如果想全面管理你的IBM硬件，装Director吧。