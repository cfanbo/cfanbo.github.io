---
title: CentOS 5 VPS的nginx+php+mysql解决方案之一
author: admin
type: post
date: 2010-10-12T01:06:22+00:00
url: /archives/5970
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - nginx

---
在CentOS5 VPS下的nginx+php+mysql的解决方案有多个，本文介绍其中的解决方案之一。

本文基于64位的CentOS 5 VPS，如果是32位的VPS，请在相应部分做修改。

本解决方案使用[瑞豪开源自己编译的最新稳定版本的Nginx][1]，[fastcgi进程管理使用spawn-fcgi][2]，还有CentOS 5自带的5.0.45版本的MySQL和5.1.6版本的php。

## 优缺点

本方案的优点是使用CentOS5自带的php和mysql，扩展性好，php的各种扩展yum库里面都有，都可以直接使用；另外，由于使用系统自带的php和mysql，安全性要好一些，如果有什么漏洞都可以直接升级为centos官方的最新版本。由于使用spawn-fcgi，所以无须重新编译php。

本方案的缺点有：

 1. php和mysql都是centos自带的版本，不是最新版本，万一用到php最新版本的某些特性则就不行了。
 2. spawn-fcgi的性能不如php-fpm，如果想用php-fpm，请参考[http://rashost.com/blog/centos5-vps-nginx-solution2][3]

## 安装Nginx

到 ` [http://rashost.com/download](http://rashost.com/download "http://rashost.com/download") 下载``nginx-0.7.61-1.x86_64.rpm`

安装命令：

> `rpm -ivh nginx-0.7.61-1.x86_64.rpm

chkconfig --list nginx

chkconfig nginx on

/etc/init.d/nginx start

rpm -ql nginx`

上面的rpm -ql nginx命令是看看nginx的文件都安装在哪些目录下面了，可以看到nginx的缺省网页目录应该是/usr/share/nginx/html/

通过浏览器访问，应该能看到nginx的缺省网页了，说明nginx正常工作了！

## 安装MySQL

> `yum install -y mysql-server

chkconfig --list mysqld

chkconfig mysqld on

/etc/init.d/mysqld start`

运行mysql -u root命令，应该可以正常连接到MySQL

## 安装PHP

> `yum install -y php-cgi php-mysql`

## 安装spawn-fcgi

到` [http://rashost.com/download](http://rashost.com/download) 下载 spawn-fcgi-1.6.2-1.32.x86_64.rpm`

> `rpm -ivh spawn-fcgi-1.6.2-1.32.x86_64.rpm

`

然后在/etc/rc.local里面加入spawn-fcgi的启动命令：

> `spawn-fcgi -C 10 -a 127.0.0.1 -p 9000 -u nginx -d /tmp -f php-cgi`

其中的-C 10参数是指启动的php fastcgi的进程数目，这个数值可以根据网站的访问量和内存大小修改。

然后先手工启动一下php:

> `spawn-fcgi -C 10 -a 127.0.0.1 -p 9000 -u nginx -d /tmp -f php-cgi`

## 整合

首先在/usr/share/nginx/html目录下创建文件test.php，其内容很简单，只要下面一行：

``

假设所在VPS的地址是centos5.rashost.com，这时通过浏览器访问http://centos5.rashost.com/test.php是得不到正确的显示结果的。

修改nginx的配置文件/etc/nginx/nginx.conf，在文件内搜索fastcgi_pass，修改该部分内容为：

> `location ~ \.php$ {

root           html;

fastcgi_pass   127.0.0.1:9000;

fastcgi_index  index.php;

fastcgi_param  SCRIPT_FILENAME  $document_root/$fastcgi_script_name;

include        fastcgi_params;

}`

然后重启nginx:

> `/etc/init.d/nginx/restart`

然后在浏览器中访问test.php页面，就应该能正确显示了，reboot VPS测试一下，各个模块应该都能自带启动。大功告成，该来些瓶啤酒庆祝一下了！

来源:

 [1]: http://rashost.com/blog/centos5-build-nginx-rpm
 [2]: http://rashost.com/blog/spawn-fcgi-release-from-lighttpd
 [3]: http://rashost.com/blog/centos5-vps-nginx-solution2 "http://rashost.com/blog/centos5-vps-nginx-solution2"