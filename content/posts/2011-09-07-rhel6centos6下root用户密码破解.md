---
title: RHEL6/CentOS6下root用户密码破解
author: admin
type: post
date: 2011-09-07T05:44:45+00:00
url: /archives/11323
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos

---
RHEL升级到版本6以后，发现root用户密码破解和centos.5的不一样了；在单用户模式下输入passwd命令不再有效。

[![](http://blog.haohtml.com/wp-content/uploads/2011/09/centos-6-pass.png)][1]

这是由于在安装RHEL6(centos6)的过程中或者以前使用过程上，SELinux的默认级别为非0的缘故；

因此在进入单用户模式以后需要输入_setenforce 0_命令来将SELinux级别临时变为0以后，才可以使用passwd命令！

[![](http://blog.haohtml.com/wp-content/uploads/2011/09/centos-reset-root-passwd.png)][2]

当然如果你在安装过程中，更改了SELinux的级别，那么就不会遇到上述问题了！

据称，这是RHEL6的一个bug……

 [1]: http://blog.haohtml.com/wp-content/uploads/2011/09/centos-6-pass.png
 [2]: http://blog.haohtml.com/wp-content/uploads/2011/09/centos-reset-root-passwd.png