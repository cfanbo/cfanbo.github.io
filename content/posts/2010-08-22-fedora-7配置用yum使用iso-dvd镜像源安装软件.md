---
title: Fedora 7配置用yum使用iso DVD镜像源安装软件
author: admin
type: post
date: 2010-08-22T12:13:05+00:00
url: /archives/5259
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos

---
**1、先把光盘挂上**

#monut /dev/cdrom /mnt/cdrom 光盘将挂载到/mnt/cdrom   现在我们来检查光盘是否挂载成功（如果没有此目录，先创建）

#ls /mnt/cdrom    有内容则表示挂载成功。

**2、理解个道理**

CentOS有两个yum源，它们在/etc/yum.repos.d/下面有两个文件：CentOS-Base.repo和CentOS-Media.repo。但这两个源不是同时使用的，默认使用的是采用互联网升级的CentOS-Base.repo源（这文件里都是网址，你可以自己看看），除非我们手动修改让系统使用Media源，而Media源就是指计算机本地的源，就包含我们方才挂上的本地光盘。

**3、开始操作**

首先，把CentOS-Base.repo文件改名，让系统找不到该文件，从而不能使用互联网的更新方式：

#mv CentOS-Base.repo CentOS-Base.repo.bak

然后，vi CentOS-Media.repo

把以下三行的后两行删除
```
baseurl=file:///media/CentOS/
file:///media/cdrom/
file:///media/cdrecorder/
```



第一行修改为挂载光盘的路径，如下：

`baseurl=file:///mnt/cdrom/`

然后，找到这个属性，将值改成1，这样就打开了本地源文件的使能开关。
`enabled=0`

改成 `enabled=1`

保存即可。

然后可以通过yum check-update 或者yum install * 来测试源是否成功和生效。

==========================================================

这里有一个比较偷懒的办法。

进入 /etc/yum.repo.d/ 文件夹

vi Centos-Media.repo

将文件中的 file 位置修改为dvd的位置 /media/dvd，并打开本地源，如下：

```
[c5-media]
name=CentOS-$releasever – Media
baseurl=file:///media/dvd/
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5
```



保存推出后就可以使用本地的dvd作为yum源了，当然，你要记得把光盘放进去哦