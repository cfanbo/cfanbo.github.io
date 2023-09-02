---
title: 错误1920 服务McAfee Framework服务(McAfee Framework)启动失败.确认有足够的权限启动
author: admin
type: post
date: 2010-05-15T05:05:10+00:00
url: /archives/3613
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - mcafee

---
确认有足够的权限启动系统服务的解决方法重装至正在启动服务，出现安装程序信息：“错误1920。服务McAfee Framework服务（McAfee Framework）启动失败。确认有足够的权限启动系统服务。”按“忽略（I）”才能继续安装，但最后还是不能升级。

经过不断摸索，上官方网站查询资料，找到解决办法如下：

打开服务控制台，禁用 McAfee Framework 服务
重新启动，启动后再进程中(打开任务管理器) 结束 UpdaterUI.exe

运行regedit ,删除以下：
HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Services\McAfee Framework
HKEY\_LOCAL\_MACHINE\SOFTWARE\Network Associates\TVD
HKEY\_LOCAL\_MACHINE\SOFTWARE\Network Associates\ePolicy orchestrator
以上三个必须删除

删除以下目录
C:\Program Files\Network Associates\Common Framework
C:\Documents and Settings\All Users\Application Data\Network Associates
如果没有all Users目录，或者all users 目录下没有Network Associates 目录的，利用查找功能，将Network Associates目录删除！

卸载，重启以后再重装后问题解决。

如果不幸碰见同样问题，可以按上面的方法试一下。免得重装系统耗时耗力，呵呵。

**其他办法：**

可能有用户在Windows XP SP2测试版下使用McAfee时会遇到更新失败的情况。为了解决这个问题，你需要做如下设置：
1、“运行”>“dcomcnfg.exe”
2、双击“组件服务>计算机>我的电脑”
3、展开“DCOM配置”，打开“FrameworkService”项的属性。
4、选到“FrameworkService属性”中的“安全”选项卡
5、在“启动与激活权限”下选“自定义”，点击“编辑”按钮。
6、在弹出的“启动权限”对话框中，“添加”你的windows登录帐号和SYSTEM两个用户帐号，并分别给与“本地激活”的权限，确定退出。
7、OK，问题解决了，现在McAfee可以正常更新了~

**另外一个方法（感谢龙卷风极品论坛thefirst会员提供 ^_^ ）：**
1、Run：dcomcnfg.exe（在开始－运行里键入dcomconfig，执行）
2、Component Services – Computers – My COmputer – DCOM config
3、找到FrameworkService，在Properties里选择Security页签，将Launch and Activation Permissions 设置为“User Default”。

**MCAFee（麦咖啡）进程解释+设置指南**

首先介绍一下安装后产生的进程！
如果不安装8.1的防火墙就一共应该有7个进程
UpdaterUI.exe、shstat.exe、Tbmon.exe 、Vstskmgr.exe 、Mcshield.exe 、Frameworkservice.exe 、naPrdMgr.exe
UpdaterUI.exe：自动升级进（咖啡一个星期升级一次。）
shstat.exe：也就是你系统栏里那个盾牌一样的图标，启动项处于注册表内.(装完重新启动系统后，图标才会出现在系统任务栏中。不
过，即使没有图标，VirusScan Enterprise 仍在运行，且您的计算机仍受到保护。）
您可以通过检查以下注册表键进行确认：
HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run ShStatEXE=”C:\Program Files\Network
Associates\VirusScan\SHSTAT.EXE”/STANDALONE
Tbmon.exe：错误报告程序
Vstskmgr.exe（Network Associates Task Manager）：这个东西属于系统服务。
Mcshield.exe：咖啡的核心，系统服务！
Frameworkservice.exe（McAfee Framework 服务，McAfee 产品的共享组件框架）：咖啡的后台框架进程，属于服务。平时关闭的话，经
过我测试是不影响普通使用的，但是会影响升级，且这个进程和Vstskmgr.exe不一样，他不能自动的去启动服务，如果你调整成手动，那么你非得自己动手去启动这个服务，才能运行升级程序。
naPrdMgr.exe：这个进程以前的版本就有，它是个Frameworkservice.exe在一起的，如果Frameworkservice.exe被停止，则它绝对不会在任务管理器里出现。
下面就打开你的咖啡，（右击系统托盘咖啡的图标，选virusscan控制台)

这里就是咖啡的简单的网络防火墙功能，可以设定让麦咖啡来阻止相应的端口，比如阻止了25端口以后，就可以禁止某些木马把你的密码等信
息当邮件发送出去，但是如果你用软件发邮件也会被阻止，可以在排除进程中输入你现在使用的邮件软件的进程名字，比如foxmail.exe

比如有个木马是blazer5，连接的是5000端口，在这里设置一下可以屏蔽掉5000端口，如下图

这里可以设置你的共享资源，主要是下面，他已经有很多的自带规则了，你可以禁止在windows的目录中新建文件，防止木马的破坏，要装软件
的时候临时运行，这样虽然比较麻烦，但是安全性确实很不错。默认的设置状态下，打开一个压缩文件是无法直接双击运行文件的，像上面那
个的设置，因为有些压缩包中可能有恶意代码，在临时文件夹中运行某些恶意程序.

这里是防止某些病毒利用缓冲区溢出漏洞来传播

建议把关机时检测软盘关了，不然很烦的，下面的扫描时间是个很难改动的设置，时间太短的话可能会检查不完，太长的话有大文件会很慢
，不过好在这里是最长检测时间，后面的选项，默认就好.

这是一个很不错的设定，可以对高低风险的进程进行不同的设置，比如有些病毒很喜欢使用系统进程的名字，当然就是高风险的了，这样设置
的话可以对低风险进程放宽设置以降低系统资源占用扫描文件这里，可以设置成只在读取文件时检测，这样可以节省一些资源，当然也是放弃了安全性的。如果你在局域网上，建议选中检测网络
驱动器。扫描文件类型，按访问扫描的话建议选择默认类型，一般某些不常用的类型，包括某些.bak文件，都是没有太大危害的，至少现在他
不会运行，不是特别担心的话选择默认类型好了。下面是排除列表，就是不扫描的文件夹，比如麦咖啡把破解的serv-u当病毒，这里可以把
serv-u的文件夹排除就可以了。建议把c盘的隔离文件夹添加进去，不然要去里面删除病毒的时候他可能会弹出来烦人.

这里要说的就是是否扫描压缩文件，我的建议是不扫描，因为扫描他们会花去大量的时间，即时里面有病毒，也需要先解压出来，就是直接运
行也要解压到临时文件夹，这个时候麦咖啡也会自动去检测的。

这里可以在系统空闲的时候自动扫描内存或者某些敏感文件夹。新建一个按需扫描任务，目标是内存或者敏感文件夹（比如c:\windows\），然后点计划

这里可以选择检测时的cpu占用率，太低的花话可能会让检测速度变慢。如果你设定了空闲时扫描内存的话，这里还是改低一些吧。