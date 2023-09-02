---
title: '[教程]FreeBSD下使用ports安装Nginx + PHP5.2.6 + Php-fpm'
author: admin
type: post
date: 2010-09-06T06:38:38+00:00
url: /archives/5586
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx
 - php-fpm

---
钟情FreeBSD的其中一个原因就是它的方便快捷的ports软件包管理，本文在安装Nginx、PHP、Php-fpm的时候也采用ports方式安装。ports是一个非常优秀的软件包管理器，如果不希望编译安装的话，使用ports安装，几个命令就能全部搞定，这对初学者来说是很有帮助的。

事实上，Nginx 和 PHP已经在FreeBSD的ports系统里了，只是Php-fpm没有，不过，简单几个命令就能把Php-fpm添加到FreeBSD的ports中去。下面我们来看看具体的操作步骤：

**1. 安装nginx**

\# cd /usr/ports/www/nginx

\# make install

安装过程中要选择安装模块，这里我选择如下几个模块做示范

 * HTTP_MODULE
 * HTTP\_REWRITE\_MODULE
 * HTTP\_SSL\_MODULE
 * HTTP\_STATUS\_MODULE

FreeBSD下的ports安装实在是太简单、方便了，没什么可多说的，下面直接安装php。

**2. 安装php**

\# cd /usr/ports/lang/php5
\# make install

安装过程中，选择如下模块：

 * CLI
 * CGI
 * SUHOSIN
 * FASTCGI]
 * FPM (目前FreeBSD里面已经集成了php-fpm这一项了,所以下面的第三步就不用安装了)
 * PATHINFO

**3. 安装Php-fpm**

FreeBSD是没有php-fpm的ports的，那就下个呗

\# wget http://alamster.googlepages.com/php5-fpm.5.2.6.tar.gz

然后解压至ports中

\# tar xvzf php5-fpm.5.2.6.tar.gz –-directory=/usr/ports/lang

\# rm php5-fpm.5.2.6.tar.gz

现在就可以进入ports安装php-fpm了
\# cd /usr/ports/lang/php5-fpm/ && make install

安装过程中选择安装如下模块：

 * CLI
 * SUHOSIN
 * PATHINFO

事实上，以上的步骤已经把这本个软件包都安装好了，简单吧，现在开始对nginx作简单的配置吧

**4. 配置**

配置前，我们更改一下rc.conf文件，让nginx和php-fpm开机启动，编辑 /etc/rc.conf 文件

\# ee /etc/rc.conf

添加如下配置

nginx_enable=”YES”


php_fpm_enable=”YES”


保存并退出编辑器。

编辑nginx.conf文件以配置nginx

\# ee /usr/local/etc/nginx/nginx.conf

找开如下行，并做如下修改

01. location / {

02. root /usr/local/www/nginx;

03. index index.php index.html index.htm;

04. }

05. location ~ \.php$ {

06. fastcgi_pass 127.0.0.1:9000;

07. fastcgi_index index.php;

08. fastcgi_param SCRIPT_FILENAME /usr/local/www/nginx$fastcgi_script_name;

09. include fastcgi_params;

10. }


编辑php-fpm.conf

#ee /usr/local/etc/php-fpm.conf

找到如下行

nobody –>
nobody –>

把nobody改为www，并去除注释

1. www
2. www

开启nginx及php-fpm

\# /usr/local/etc/rc.d/php-fpm start

\# /usr/local/etc/rc.d/nginx start

**5. 测试**

安装相对简单，文中添加的文字也不是很多，大家一看命令便知，安装配置好后，现在我们可以做一下简单的测试，看是否工作。新建info.php文件，并在浏览器中访问之

\# cd /usr/local/www/nginx

\# ee info.php

另一篇介绍FastCGI方式运行php的: [http://blog.haohtml.com/index.php/archives/1133](http://blog.haohtml.com/index.php/archives/1133),不推荐使用,安装也麻烦一些.

如果需要安装Memcached的话，请参考这篇文章：,只要执行一下#/usr/local/etc/rc.d/php-fpm reload 命令即可，不用重启nginx的。