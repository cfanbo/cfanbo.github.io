---
title: FreeBSD 8.0 Firefox 安装 Flash 插件
author: admin
type: post
date: 2010-10-06T10:25:45+00:00
url: /archives/5930
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - firefox

---
\# cd /usr/ports/www/nspluginwrapper && make install clean
如果没有加载Linux核心模块，会出错，请加载Linux后重新安装

> \# kldload linux
> \# echo ‘linux_enable=”YES”‘ >> /etc/rc.conf

\# cd /usr/ports/www/linux-f10-flashplugin10 && make install clean
\# mkdir /usr/local/lib/browser_plugins
\# ln -s /usr/local/lib/npapi/linux-f10-flashplugin/libflashplayer.so /usr/local/lib/browser_plugins/

按照 FreeBSD 版本， 在安装了正确的 Flash port 之后， **插件必须由每个用户运行 nspluginwrapper 安装**：
% nspluginwrapper -v -a -i

\# mount -t linprocfs linproc /usr/compat/linux/proc

\# ee /etc/fstab

> 把以下这行加入 /etc/fstab：
> linproc /usr/compat/linux/proc linprocfs rw 0 0

\# cd /usr/local/lib/firefox3/plugins && ln -s /usr/local/lib/browser_plugins/npwrapper.libflashplayer.so npwrapper.libflashplayer.so

参考： [http://cnsnap.cn.freebsd.org/doc/zh_CN.GB2312/books/handbook/desktop-browsers.html](http://cnsnap.cn.freebsd.org/doc/zh_CN.GB2312/books/handbook/desktop-browsers.html)