---
title: FreeBSD portupgrade升级你的FreeBSD软件
author: admin
type: post
date: 2010-06-24T14:39:56+00:00
url: /archives/3915
IM_contentdowned:
 - 1
categories:
 - 服务器

---

**portupgrade** 是一个软件，用于快捷便利地升级软件,安装办法:


#cd /usr/ports/sysutils/portupgrade

#make install clean


然后用cvsup更新ports树，最后运行:


**#portupgrade -r** pkg_name 升级单个软件和与其相关的,其中 pkg_name 是 pkg_info 中显示的名字

**portupgrade** -ar 就会自动更新全部了。

如果加上 P 参数，则先看是否有已经编译好的 pkg 下载，直接从 pkg 升级，省去自己编译。

下载站点可以通过修改 /usr/local/etc/pkgtools.conf 更改。


portupgrade -arR 升级所有已经安装的软件，并且检查依赖关系。