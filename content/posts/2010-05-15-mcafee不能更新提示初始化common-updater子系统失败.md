---
title: mcafee不能更新,提示初始化Common updater子系统失败
author: admin
type: post
date: 2010-05-15T05:00:03+00:00
url: /archives/3610
IM_data:
 - 'a:1:{s:73:"http://qingwa.hackroad.com/newblog/template/elegantly/images/download.gif";s:68:"http://blog.haohtml.com/wp-content/uploads/2011/03/c315_download.gif";}'
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - mcafee

---
[![mcafee-autoupdate](http://blog.haohtml.com/wp-content/uploads/2010/05/mcafee-autoupdate.jpg)][1]升级mcafee时出现初始化Common updater子系统失败，重装修复问题依然存在，不能解决，看图：

一共有两种解决办法，其一是复制 FrameworkManifest.xml 这个文件来覆盖，其二是删除 FrameworkManifest.xml并重新安装Common Management Agent (CMA)

8.5.0.i版本的FrameworkManifest.xml文件下载：

![mcafee不能更新,提示初始化Common updater子系统失败 - 饿狼 - 我们俩](http://qingwa.hackroad.com/newblog/template/elegantly/images/download.gif)下载文件 (已下载 153 次)


[点击这里下载文件: FrameworkManifest.xml](http://qingwa.hackroad.com/newblog/attachment.php?fid=9)

官方的bug说明：

Corporate KnowledgeBase

ERROR: McAfee Common Framework returned error fffff95b @ 2 (issue: FrameworkManifest.xml corrupt)

Corporate KnowledgeBase ID:     KB54520

Published:     August 08, 2008

Environment

McAfee Common Management Agent 3.60

McAfee Common Management Agent 3.5x

McAfee VirusScan Enterprise 8.5i

McAfee VirusScan Enterprise 8.0i

Problem 1

The following errors occur after initiating an AutoUpdate:

McAfee Common Framework returned error fffff95b @ 2. (FrameworkManifest.xml corrupt)

Failed to initialize common updater subsystem

Make sure the McAfee Framework Services is running

Problem 2

Subsequent errors when trying to start the McAfee Framework Service:

Could not start the McAfee Framework Service on Local Computer

The system cannot find the file specified

Problem 3

Any of the following update methods will result in the AutoUpdate error:

Right-clicking the McShield icon in the system tray and selecting Update Now

Right-clicking the AutoUpdate in the VirusScan Console and clicking Start

Creating a new scheduled task

Editing the properties of the existing AutoUpdate task

Cause

FrameworkManifest.xml has become corrupted.

Solution 1

McAfee Agent 4.0

Changes in the design of the McAfee Agent 4.0 will prevent the corruption of the FrameworkManifest.xml file.

To download the McAfee Agent from the www.mcafee.com website, see KB54808 .

Solution 2

Common Management Agent

Solution 1 – Obtain FrameworkManifest.xml from another computer:

Locate another VirusScan Enterprise (VSE) computer where the updates are working without error.

Copy the FrameworkManifest.xml from the following path:

For VSE 8.5i   (running on Windows Vista)

x:\Program Data\McAfee\Common Framework

For VSE 8.5i (running on Windows XP and earlier)

x:\Documents and Settings\All Users\Application Data\McAfee\Common Framework

For VSE 8.0i (running on Windows XP and earlier)

x:\Documents and Settings\All Users\Application Data\Network Associates\Common Framework

Paste the file to portable media or network share that can be accessed by both computers.

Click Start, Run, type services.msc, and click OK.

Right-click McAfee Framework Service and select Stop.

Copy FrameworkManifest.xml to the Common Framework directory.

Right-click McAfee Framework Service and select Start.

Update the product.

Solution   2 – Delete FrameworkManifest.xml and reinstall the Common Management Agent (CMA)

It is necessary to delete FrameworkManifest.xml because it may not be removed or replaced when an uninstall/re-install is undertaken.

Step 1 – Allow VSE files and settings to be modified (VirusScan Enterprise 8.5i and higher only)

Click Start, Programs, McAfee, VirusScan Console.

Right-click Access Protection, then select Properties.

Select Common Standard Protection.

Select Prevent modification of McAfee files and settings and disable this option.

Click OK.

Step 2 – Delete FrameworkManifest.xml and reinstall CMA:

Delete FrameworkManifest.xml from the following path:

For VSE 8.5i (running on Windows Vista)

x:\Program Data\McAfee\Common Framework

For VSE 8.5i (running on Windows XP and earlier)

x:\Documents and Settings\All Users\Application Data\McAfee\Common Framework

For VSE 8.0i (running on Windows XP and earlier)

x:\Documents and Settings\All Users\Application Data\Network Associates\Common Framework

Restart your computer.

Re-install the Common Management Agent (CMA) / ePO agent.

NOTE: CMA is available for download from the McAfee Downloads site. See KB54808.

Previous Document ID

5432392

Rate this Page

Please take a moment to complete this form to help us better serve you.

官方的解决方法原介绍页面是

[https://kc.mcafee.com/corporate/index?page=content&id=KB54520](https://kc.mcafee.com/corporate/index?page=content&id=KB54520)

以下是第二份资料：

『朝花夕拾』论坛（互联网址：Zhxs.Net）|www.zhxs.net 详细出处参考： [http://www.zhxs.net/bbs/ShowPost.asp?ThreadID=1505](http://www.zhxs.net/bbs/ShowPost.asp?ThreadID=1505)

McAfee 不能升级了，升级时提示 [“fffff95b@2](http://xu020408.blog.163.com/“fffff95b@2)”返回错误，卸载后重装，升级时提示变成“初始化常规更新程序子系统失败，确保McAfee Franework Service正在运行。McAfee Common Framework返回错误 [80040154@1](http://xu020408.blog.163.com/80040154@1)”，进入服务管理器手动启动McAfee Common Framework服务，不能启动该服务。交你一招! 删除以下三个子键：

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\McAfee Framework

HKEY_LOCAL_MACHINE\SOFTWARE\Network Associates\TVD

HKEY_LOCAL_MACHINE\SOFTWARE\Network Associates\ePolicy Orchestrator

再删除以下目录：

C:\Program Files\Network Associates\Common Framework

C:\Documents and Settings\All Users\Application Data\Network Associates

如果没有all Users目录，或者all users目录下没有Network Associates目录的，利用查找功能，将Network Associates目录删除。最后再将McAfee卸载，重启以后再重新安装即可解决问题。

其他办法：

可能有用户在Windows XP SP2测试版下使用McAfee时会遇到更新失败的情况。为了解决这个问题，你需要做如下设置：

1、“运行”>“dcomcnfg.exe”

2、双击“组件服务>计算机>我的电脑”

3、展开“DCOM配置”，打开“FrameworkService”项的属性。

4、选到“FrameworkService属性”中的“安全”选项卡

5、在“启动与激活权限”下选“自定义”，点击“编辑”按钮。

6、在弹出的“启动权限”对话框中，“添加”你的windows登录帐号和SYSTEM两个用户帐号，并分别给与“本地激活”的权限，确定退出。

7、OK，问题解决了，现在McAfee可以正常更新了~

另外一个方法:

1、Run：dcomcnfg.exe（在开始－运行里键入dcomconfig，执行）

2、Component Services – Computers – My COmputer – DCOM config

3、找到FrameworkService，在Properties里选择Security页签，将Launch and Activation Permissions 设置为“User Default”。

MCAFee（麦咖啡）进程解释+设置指南

首先介绍一下安装后产生的进程！

如果不安装8.1的防火墙就一共应该有7个进程

UpdaterUI.exe、shstat.exe、Tbmon.exe 、Vstskmgr.exe 、Mcshield.exe 、Frameworkservice.exe 、naPrdMgr.exe

UpdaterUI.exe：自动升级进（咖啡一个星期升级一次。）

shstat.exe：也就是你系统栏里那个盾牌一样的图标，启动项处于注册表内.(装完重新启动系统后，图标才会出现在系统任务栏中。不

过，即使没有图标，VirusScan Enterprise 仍在运行，且您的计算机仍受到保护。）

您可以通过检查以下注册表键进行确认：

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run ShStatEXE=”C:\Program Files\Network

Associates\VirusScan\SHSTAT.EXE”/STANDALONE

Tbmon.exe：错误报告程序

Vstskmgr.exe（Network Associates Task Manager）：这个东西属于系统服务。

Mcshield.exe：咖啡的核心，系统服务！

Frameworkservice.exe（McAfee Framework 服务，McAfee

转贴请注明『朝花夕拾』论坛（互联网址：Zhxs.Net）|www.zhxs.net。请保留本段文字。『朝花夕拾』论坛版权所有，Copyright＠Zhxs.Net，All rights reserved！

第三份::

Macfee 官方网站疑难问题解析

我一直在用Macfee 8.5版杀毒软件，近日好多同事Macfee 不能升级了，提示 返回错误，卸载后重装，升级时提示变成“初始化常规更新程序 **子系统** 失败，确保**McAfee** Franework Service正在运行。 **McAfee** Common Framework 返回错误[email=80040154@1]80040154@1[/email]”。进服务控制台手动启动 **McAfee** Common Framework 服务，不能启动该服务。

重装至正在启动服务，出现安装程序信息：“错误1920。服务 **McAfee** Framework服务（ **McAfee** Framework）启动失败。确认有足够的权限启动系统服务。”按“忽略（I）”才能继续安装，但最后还是不能升级。

经过不断摸索，上官方网站查询资料，找到解决办法如下：可以解决问题

打开服务控制台，禁用 **McAfee** Framework 服务

重新启动，启动后再进程中(打开任务管理器) 结束 UpdaterUI.exe

运行regedit ,删除以下：

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\ **McAfee** Framework

HKEY_LOCAL_MACHINE\SOFTWARE\Network Associates\TVD

HKEY_LOCAL_MACHINE\SOFTWARE\Network Associates\ePolicy Orchestrator

以上三个必须删除

删除以下目录

C:\Program Files\Network Associates\Common Framework

C:\Documents and Settings\All Users\Application Data\Network Associates

如果没有all Users目录，或者all users 目录下没有Network Associates 目录的，利用查找功能，将Network Associates目录删除！

卸载，重启以后再重装后问题解决。

如果不幸碰见同样问题，可以按上面的方法试一下。免得重装系统耗时耗力，呵呵。

[其他办法』：

可能有用户在Windows XP SP2测试版下使用 **McAfee** 时会遇到更新失败的情况。为了解决这个问题，你需要做如下设置：

1、“运行”>“dcomcnfg.exe”

2、双击“组件服务>计算机>我的电脑”

3、展开“DCOM配置”，打开“FrameworkService”项的属性。

4、选到“FrameworkService属性”中的“安全”选项卡

5、在“启动与激活权限”下选“自定义”，点击“编辑”按钮。

6、在弹出的“启动权限”对话框中，“添加”你的windows登录帐号和SYSTEM两个用户帐号，并分别给与“本地激活”的权限，确定退出。

7、OK，问题解决了，现在 **McAfee** 可以正常更新了~

另外一个方法（感谢龙卷风极品论坛thefirst会员提供 ^_^ ）：

1、Run：dcomcnfg.exe（在开始－运行里键入dcomconfig，执行）

2、Component Services – Computers – My COmputer – DCOM config

3、找到FrameworkService，在Properties里选择Security页签，将Launch and Activation Permissions 设置为“User Default”。

还可以试一下这个东西：

1) Click Start, Run, type CMD and press ENTER.

2) From the command prompt, type:

Regsvr32.exe %Windir%\System32\Ole32.dll and press ENTER

 [1]: http://blog.haohtml.com/wp-content/uploads/2010/05/mcafee-autoupdate.jpg