---
title: 用freebsd-update命令升级freebsd系统
author: admin
type: post
date: 2010-01-20T02:37:10+00:00
excerpt: |
 freebsd-update -- fetch and install binary updates to FreeBSD

 描述:
 freebsd-update 是FreeBSD系统用来撷取, 安装及取得binary update 的工具。要注意的是,这些update仅止于FreeBSD Release Engineering Team维护的版本.
 诸如: FreeBSD 6.1-RELEASE 或 FreeBSD 6.2-RC1 而非 FreeBSD 6.2-STABLE
 or FreeBSD 7.0-CURRENT 之类的版本.
 补充: FreeBSD 6.2-RELEASE版本后才有此指令
url: /archives/2840
IM_contentdowned:
 - 1
categories:
 - 服务器

---
freebsd-update — fetch and install binary updates to FreeBSD

描述:
freebsd-update 是FreeBSD系统用来撷取, 安装及取得binary update 的工具。要注意的是,这些update仅止于FreeBSD Release Engineering Team维护的版本.
诸如: FreeBSD 6.1-RELEASE 或 FreeBSD 6.2-RC1 而非 FreeBSD 6.2-STABLE
or FreeBSD 7.0-CURRENT 之类的版本.
补充: FreeBSD 6.2-RELEASE版本后才有此指令

语法:
freebsd-update \[-b basedir\] \[-d workdir\] \[-f conffile\] \[-k KEY\]
\[-r newrelease\] \[-s server\] [-t address] command

**参数:**

> -b basedir　　　　指定系统挂载的最基本路径 (预设: / )
>
> -d workdir　　　　档案暂存数据夹 (预设: /var/db/freebsd-update/ )
>
> -f conffile　　　　设定文件位置 (预设: /etc/freebsd-update.conf)
> -k KEY　　　　　　信任的RSA金钥位置 (预设: 从设定档读取)
> -r newrelease　　定义新的RELEASE版本升级标的 (针对 upgrade)
> -s server　　　　定义撷取更新档案的server (预设:从设定档读取)
>
> -t address　　　　邮件输出的对象  (预设: root )

**命令:**

> fetch      以现有安装的环境及设定参数, 撷取可能的binary更新.
> cron        随机休息(sleep)1~3600秒,然后下载更新档.
> 若更新档案下载完成,系统会发送email通知root(可透过 -t 参数或设定档 , 将信件递送给其它人员) .
> 如同此命令的名称(cron), 被用来设计透过cron程序执行.
> 随机休息秒数则是用来避免同时间有大量机器向server要求更新.
> upgrade   截取必要的升级到新版本RELEASE的档案,请小心使用.
> 并确认您已经阅读过新版本的 announcement and release notes.
> install      安装最近撷取的更新(update)/升级(upgrade)档案.
> rollback    反安装最近安装过的更新(update).

**小技巧:**
把 freebsd-update 放入cron执行 , 如此就能够自动更新.

>

```
@daily     root   /usr/sbin/freebsd-update cron
```

相关档案/数据夹:

> /etc/freebsd-update.conf  预设的freebsd-update程序设定文件
> /var/db/freebsd-update/   预设的freebsd-update暂存及下载档案数据夹

实际操作:这里将版本从7.2升级到8.1 RELEASE版本

> freebsd-update -r 8.1-RELEASE upgrade

中间按提示输入就行了.

Does this look reasonable (y/n)?    全 y

大部分都不需要修改,只是文件的版本时间改变
会有一些需要合并的文件,程序会自动用 vi 打开,解决一下就行了(只要把以前版本里面的那些resolv配置修改一下就可以了，保持原来的域名和解析的ip一样，把my.domain换成自己原来的域名)

安装:

> freebsd-update  install

会有如下提示

> ////////
> Installing updates…
> Kernel updates have been installed.  Please reboot and run
> “/usr/sbin/freebsd-update install” again to finish installing updates
> ////////

然后

> shutdown -r now

重新启动后

> freebsd-update install

uname -a 查看,已经OK

重新开机后,执行 uname -a 名称就会变成 8.1-RELEASE , 若不在意名称,也不需要重新开机.

整理参考：

请参考官方手册：(FreeBSD Update段)