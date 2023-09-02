---
title: kickstart 语法详解
author: admin
type: post
date: 2011-09-09T05:50:01+00:00
url: /archives/11374
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - kickstart
 - redhat

---
**kickstart 语法**

接下来探讨ks.cfg 的相关参数，这些参数笔者将依上述ks,cfg 出现的先后顺序来讨论，有些参数并不是一定要设置。完整的kickstart 参数意义可参考下列网址。

[http://www.redhat.com/docs/manuals/enterprise/RHEL-3-Manual/sysadmin-guide/s1-kickstart2-options.html](http://www.redhat.com/docs/manuals/enterprise/RHEL-3-Manual/sysadmin-guide/s1-kickstart2-options.html)

**ks.cfg 文件由三个部份皆组成：**

 * command 区段—此部份包含了必要安装选项
 * packages 区段—列出欲安装套件
 * ％pre and %post 区段
 * command 区段
 ■lang(必要)：安装时所使用的语言
 例如：安装过程中选用中文语言，lang zh_TW.Big5
 ■langsupport (必要)：指定系统使用的语言。假如你安装一至多国语系，你必需使用默认选项去指定默认语言。语法为：
 例如：langsupport –default en\_US.UTF-8 zh\_TW.Big5 en_US.UTF-8
 ■键盘(必要)：设置系统键盘的种类。语法为：keyboard us
 ■鼠标(必要)：设置鼠标。语法为：
 mouse- -device=ttvS0(鼠标识别装置位置)- – emulthree(仿真三个按键)generics/2(定义鼠标种类)
 ■timezone(必要) 设置系统时区。
 timezone Asia/Taipei (指定你的时区位置)
 ■设置系统键盘的种类。语法为：keyboard us
 ■鼠标(必要)：设置鼠标。语法为：
 mouse- -device=ttvS0(鼠标识别装置位置)- – emulthree(仿真三个按键)generics/2(定义鼠标种类)
 ■xconfig(非必要)：在安装过程中手动设置X，假如你不想安装X，你不应该使用此选项。命令的格式为：
 ■xconfig- – card(显示卡类别)- – videoram(指定显示卡记忆容量)- – hsync(指定屏幕水平扫描频率)- -vsync(指定屏幕垂直扫描频率)- – resolution(指定屏幕分辨率) – – depth(指定X 窗口系统彩度)- -startxonboot (假如你想在系统开机时激活X 时使用)- – defaultdesktop gnome(或kde)(指定默认桌面)。
 ■install (非必要)：告知系统安装一个新的安装。这是默认模式，因此一个新的安装不需再选用这个命令。接着您必需指定安装方式，可以是cdrom、harddrive、nfs 或url。
 ■cdrom
 ■harddrive—partition=your partition –dir=/your directory path
 – partition = 来源分区
 – dir = Red Hat 子目录
 (请确定你所键入来源分区和子目录信息的正确性)。
 ■nfs – server—your server –dir=/your directory path
 – server = 指定安装来源服务器
 – dir = Red Hat 子目录
 (请确定你所键入来源分区和子目录信息的正确性)。
 ■url – url http://your server/dir
 使用HTTP 进行安装
 ■url – url ftp://your username:password@your server/dir
 使用FTP 进行安装
 ■rootpw (必要) 设置一组系统root 密码。
 rootpw – – iscrypted (表示密码已被加密) password
 ■firewall(非必要) 提供安全性等级来保护系统。
 ■authconfig (必要) 设置系统认证选项。命令格式：
 – -enablemd5 (使用md5 编码使用者密码)
 – -enableshadow (使用shadow 密码)
 ■bootloader (必要) 指定开机管理程序的位置和传递任何kernel 选项。默认开机管理程序为GRUB，但是你也能选择LILO 开机管理程序来取代GRUB。命令格式为：
 – – location=mbr (指定开机管理程序的位置)
 – -append=(指定要传递的核心参数)。
 – -useLilo (使用LILO 为开机管理程序)。
 ■clearpart (非必要)告知系统移除系统上的分区。你可以使用clearpart 移除Linux 分区以及移除所有的分区，或者你也能指定你想要移除分区的磁碟机。命令格式为：
 – linux (移除所有Linux 分区)
 – – all (移除系统上所有的分区)
 – drives = (指定要移除分区的磁盘驱动器)
 ■Part (必要) 安装时是必要的，升级时请忽略。使用这个命令你能为系统建立分区。
 * package 区段安装一个新的系统，你必需选择你想安装的套件。选择欲安装的套件是使用%packages 命令。套件可分为单一套件或者是套件组。你能在第一片Red Hat安装光盘下的/base/comps.xml 寻找群组套件清单。通常，只需列出套件组不需要列出单一套件。注意！默认之下core 和base 群组是被选取的，所以也不需要在 %packages 这个区段下去指定它们。如同利用ksconfig 所产生出来的ks.cfg %packages 区段中套件组是一行指定一个，以＠节号开头，后面加上一格空白接下来是完整群组名称就如同comps.xml 文件所指定。如果个别单一套件并列出该单一套件名，不加上额外的字符。套件组是一行指定一个，以＠节号开头，后面加上一格空白接下来是完整群组名称就如同comps.xml 文件所指定。如果是个别单一套件则列出该单一套件名，前面不需加上额外的字符。%package 有三个选项可以设置：
 ◆- -resolvedeps
 决解自动相依性问题及安装套件。建意选项，在安装中由于没使用自动决解相依性，若有相依性问题可能会造成中止安装并且做提示响应。
 ◆- -ignoredeps
 你选择安装套某套件但乎略它的相依性，可能造成此套件无法运作，尤其是此套件需要其它相依的套件。
 ◆—ignoremissing
 标示忽视安装遗失套件及群组并且也不做提示响应。

 * ％pre and %post 区段%pre 区段内可填入在开始安装操作系统需要先执行的工作。%post 命令传递到系统上执行必须在Kickstart 安装完成后。能有效的执行命令去安装其它的软件或者设置系统信息。