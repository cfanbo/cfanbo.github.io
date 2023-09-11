---
title: 为centos添加EPEL软件仓库
author: admin
type: post
date: 2010-09-18T14:29:16+00:00
url: /archives/5755
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos

---
想用Red Hat Enterprise Linux，但苦于囊中羞涩（欢迎 [购买 RHEL AS Subscription](http://filteroff.com/index.php?q=uggcf%3A%2F%2Fjjj.erqung.pbz%2Fjnccf%2Ffgber%2F "购买 RHEL AS Subscription")：US$1,499 / year），好在还有centos可选，但总感觉它的包少了点。

好在现在可以使用Fedora Project 推出的 EPEL（Extra Packages for Enterprise Linux），EPEL是RHEL 的 Fedora 软件仓库，把它添上，你就可以获得 RHEL AS 的高质量、高性能、高可靠性，又需要方便易用（关键是免费）的软件包更新功能。

EPEL（ [http://fedoraproject.org/wiki/EPEL](http://fedoraproject.org/wiki/EPEL "http://fedoraproject.org/wiki/EPEL")） 是由 Fedora 社区打造，为 RHEL 及衍生发行版如 CentOS、Scientific Linux 等提供高质量软件包的项目。装上了 EPEL，就像在 Fedora 上一样，可以通过 yum install package-name，随意安装软件。
安装 EPEL 非常简单：
* RHEL 4（centos 4）：

> su -c ‘rpm -Uvh http://download.fedora.redhat.com/pub/epel/4/i386/epel-release-3-9.noarch.rpm’

* RHEL 5（centos 5）：

> su -c ‘rpm -Uvh http://download.fedora.redhat.com/pub/epel/5/i386/epel-release-5-4.noarch.rpm’

安装完毕之后，即可使用 yum 来安装软件(注意下面的版本号,现在没有epel-release-5.3.noarch.rpm了,5.4为最新的版本的)，比如 Nagios：

> #yum -y install nagios

对于nginx:(可以参考：)

> #yum -y install nginx

若要查看 EPEL Repo 中是否存在某个软件包：

> #yum search package-name