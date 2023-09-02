---
title: Linux下的虚拟主机管理软件kloxo 控制面板的安装
author: admin
type: post
date: 2010-05-01T04:59:16+00:00
url: /archives/3514
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - 虚拟主机
 - kloxo

---
官方网站：

安装教程：

64位：

============================================================================

安装Kloxo/Lxadmin控制面板首先要确保安装的CentOS 32bit的Linux发行版（64位问题比较多），再使用putty登录Linux，如果不会可以查看： [如何使用Putty远程(SSH)管理Linux VPS](http://www.vpser.net/uncategorized/putty-ssh-linux-vps.html "到《如何使用Putty远程(SSH)管理Linux VPS》的永久链接")

执行如下命令：
wget http://download.lxcenter.org/download/kloxo/production/kloxo-installer.sh
sh ./kloxo-installer.sh –type=master

先按提示，然任意建开始安装，后面会有提示，一般输入y，回车就行。

国内主机可能安装要慢点了，因为是在线安装（更新源在国外），使用美国主机的朋友们很快就能安装完了。
安装完后你除了安好Kloxo/Lxadmin，同时也基本安好了Apache、Lighttpd、MySQL、Xcache、Bind、Djbdns等一系列服务器软件。

yum install php-bcmath /\*高精度数学运算组件，默认没安装，MD5运算时用到\*/
yum install  php-mhash
yum install php-mbstring
yum check-update (检查更新)
yum update (更新所有更新)
yum clean all （清理安装包）

基本完成，可以把终端关闭了。我们来登录Kloxo/Lxadmin，第一次登陆默认的用户名和密码都是admin，登录地址：

**https://IP:7777/**      /\*安全连接，不过默认证书不受IE信任\*/
**http://IP:7778/**       /\*还是用这个普通链接吧\*/

Zend可以在Lxadmin后台的PHPConfig里启用，Apache可以从SwichProgram里选择，建议先选择lighttpd和bind然后再选回apache和djbdns，否则你会看到内存占用量很高。

新手建议用Apache，99%能正常支持.htaccess的rewrite规则。

安装中文语言包看一参考本文：

安装完后需要修改/etc/httpd/conf/httpd.conf  查找：AddDefaultCharset UTF-8 改为：AddDefaultCharset OFF ，这样就会引起网页的乱码问题。

**>>转载请注明出处：[VPS侦探][1] 本文链接地址： [http://www.vpser.net/vps-cp/centos-linux-vps-kloxol-xadmin.html](http://www.vpser.net/vps-cp/centos-linux-vps-kloxol-xadmin.html "CentOS Linux VPS Kloxo/Lxadmin虚拟主机控制面板安装教程")**

 [1]: http://www.vpser.net/