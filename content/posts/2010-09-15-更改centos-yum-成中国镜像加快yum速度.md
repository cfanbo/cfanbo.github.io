---
title: 更改centos yum 成中国镜像加快yum速度
author: admin
type: post
date: 2010-09-15T05:21:21+00:00
url: /archives/5669
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos

---
163的开源镜像地址 [http://mirrors.163.com/.help/CentOS-Base-163.repo](http://mirrors.163.com/.help/CentOS-Base-163.repo) 不同版本见 [http://mirrors.163.com/.help/centos.html](http://mirrors.163.com/.help/centos.html)

sohu的开源镜像地址 [http://mirrors.sohu.com/help/CentOS-Base-sohu.repo](http://mirrors.sohu.com/help/CentOS-Base-sohu.repo) 不同版本见 [http://mirrors.sohu.com/help/centos.html](http://mirrors.sohu.com/help/centos.html) (只支持4, 5版本)

中国科技大学 [http://lug.ustc.edu.cn/wiki/_export/code/mirrors/help/centos?codeblock=2](http://lug.ustc.edu.cn/wiki/_export/code/mirrors/help/centos?codeblock=2) 不同版本见 [http://lug.ustc.edu.cn/wiki/mirrors/help/centos](http://lug.ustc.edu.cn/wiki/mirrors/help/centos)

> 如果使用上面YUM源的话,最好把里面的 **mirrorlist** 注释掉,否则系统会启动 **fastesmirror** 插件自动检查的,并不一定会使用这个yum源的.

我用的是中国科技大学的速度不错。

**法一：直接下载源文件**

> CentOS USTC mirror 这个镜像不错，大家更新可用这个
>
> #yum -y install wget
> #cd /etc/yum.repos.d
> #mv CentOS-Base.repo  CentOS-Base.repo.save
> #wget http://centos.ustc.edu.cn/CentOS-Base.repo
> #yum makecache

注意：如果为第一次安装的新系统的话，则需要先安装wget这个下载软件，不然没有办法下载ＣentOS-Base.repo这个文件的。

**法二：手动创建源文件**

更改yum镜像站点为中国站点地址,推荐 [http://centos.ustc.edu.cn/centos/](http://centos.ustc.edu.cn/centos/)。(中国科技大学)

> #cd /etc/yum.repos.d/
>
> #mv CentOS-Base.repo CentOS-Base.repo.bak
>
> #vi CentOS-Base.repo

修改 **/etc/yum.repos.d/CentOS-Base.repo** 如下：

#———————————–分割线以内的———————————————–

\# CentOS-Base.repo
#
\# This file uses a new mirrorlist system developed by Lance Davis for CentOS.
\# The mirror system uses the connecting IP address of the client and the
\# update status of each mirror to pick mirrors that are updated to and
\# geographically close to the client.  You should use this for CentOS updates
\# unless you are manually picking other mirrors.
#
\# If the mirrorlist= does not work for you, as a fall back you can try the
\# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever – Base
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
baseurl=http://centos.ustc.edu.cn/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-5

#released updates
[updates]
name=CentOS-$releasever – Updates
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
baseurl=http://centos.ustc.edu.cn/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-5

#packages used/produced in the build but not released
[addons]
name=CentOS-$releasever – Addons
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=addons
baseurl=http://centos.ustc.edu.cn/centos/$releasever/addons/$basearch/
gpgcheck=1
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-5

#additional packages that may be useful
[extras]
name=CentOS-$releasever – Extras
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
baseurl=http://centos.ustc.edu.cn/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-5

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever – Plus
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
baseurl=http://centos.ustc.edu.cn/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-5

#———————————–分割线以内的———————————————–

简单的说就是把mirrorlist注释掉，将baseurl改成 [centos.ustc.edu.cn](http://centos.ustc.edu.cn/centos/) 的镜像地址。

用下面命令测试:

> \# yum upgrade
> #yum makecache