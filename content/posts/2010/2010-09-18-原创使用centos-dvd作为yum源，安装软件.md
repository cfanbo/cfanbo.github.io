---
title: '[原创]使用Centos-DVD作为YUM源安装系统'
author: admin
type: post
date: 2010-09-18T13:42:17+00:00
url: /archives/5742
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - mount

---

使用Centos DVD作为软件yum源来安装软件

**1.挂载CDROM**

>

> mkdir /mnt/cdrom
>

>
>

> mount -t auto /dev/cdrom /mnt/cdrom
>

有关mount的用法请参考: [http://blog.haohtml.com/archives/9583](http://blog.haohtml.com/archives/9583),可以将上面的两行写到一个文件set_yum_dvd.sh里.

**2.修改配置文件**

默认情况下是以网络的方式来安装的，如果网络无法连接话，再从本地YUM源安装，对于网络方式配置文件为etc/yum.repos.d/CentOS-Base.repo,而本地yum源配置文件为/etc/yum.repos.d/CentOS-Media.repo，所以为了让使用本地yum源，只有把CentOS-Base.repo改为名字即可。

>

> cd  /etc/yum.repos.d/

mv CentOS-Base.repo CentOS-Base.repo.bak
>

>
>

> vi CentOS-Media.repo
>

把以下三行的后两行删除

>

> baseurl=file:///media/CentOS/
>

>
>

> file:///media/cdrom/
>

>
>

> file:///media/cdrecorder/
>

第一行修改为挂载光盘的路径，如下：

>

> baseurl=file:///mnt/cdrom/
>

然后，找到这个属性，将值改成1，这样就打开了本地源文件的使能开关。

enabled=0

>

> 改成 enabled=1
>

保存即可，以后使用yum命令安装软件的话，会直接从DVD里取软件来安装的。

gpgkey=file:///etc/pki/rpm-jpg/RPM-GPG-KEY-CentOS-5 注释：这里是指定光盘中的那个名字叫RPM-GPG-KEY的认证文件

> #yum clean all
> #yum makecache

如果要安装LNMP的话，没有办法再用这个了，DVD里不会有软件的。

安装教程：[CentOS下安装lnmp(Nginx+PHP+MySQL)][1]

================= 无意义的分隔线 ========================
配置yum仓库(RedHat)

```
vi /etc/yum.repos.d/install.repo
(install.repo是自定义的,但是一定要以repo结尾系统才能识别到)
[rhel-ClusterStorage]    (仓库名称)
name=rhel-ClusterStorage  (描述)
baseurl=file:///mnt/ClusterStorage  (安装源,也可以使用ftp或http形式)
enabled=1   (是否启用此仓库,1是启用，0是不启用)
gpgcheck=1  (1是代表检测gpgkey，0是不检测)
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
(系统key位置，红帽系统都是放在此处)
```



 [1]: http://blog.haohtml.com/index.php/archives/5732