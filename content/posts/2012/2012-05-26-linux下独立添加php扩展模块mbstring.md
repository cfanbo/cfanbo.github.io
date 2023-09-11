---
title: Linux下独立添加PHP扩展模块mbstring 和 curl
author: admin
type: post
date: 2012-05-26T14:13:47+00:00
url: /archives/13059
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - mbstring

---
环境php5.2.13, 不支持mbstring扩展
假如php的源码包在/usr/local/src/php-5.2.13
php安装目录是/usr/local/php

> \# cd /usr/local/src/php-5.2.13/ext/mbstring/
> \# rpm -qa|egrep “autoconf|gcc”      这个是检测这些组件是否安装，没有安装请执行下面这句，否则会报错
> \# yum -y install autoconf gcc gcc-c++
> \# phpize
> \# ./configure –with-php-config=/usr/local/bin/php-config
> \# make
> \# make install

执行完毕后在php.ini里增加

> extension=mbstring.so

重启web服务器, 看一下phpinfo, 应该支持mbstring了!

===================================
1.安装curl

> wget http://curl.haxx.se/download/curl-7.19.6.tar.gz
> tar -zxvf curl-7.19.6.tar.gz
> cd curl-7.19.6
> ./configure –prefix=/usr/local/curl
> make
> make install

2.编译生成扩展
进入php源程序目录中的ext目录中，这里存放着各个扩展模块的源代码，选择你需要的模块，比如curl模块：


cd curl
执行phpize生成编译文件，phpize在PHP安装目录的bin目录下

> /usr/local/php5/bin/phpize

运行时，可能会报错：

> Cannot find autoconf. Please check your autoconf installation and the $PHP_AUTOCONF
> environment variable is set correctly and then rerun this script.，需要安装autoconf：
> yum install autoconf（RedHat或者CentOS）、apt-get install autoconf（Ubuntu Linux）



3.修改配置
在php.ini里，设置扩展目录：

> extension_dir = “/usr/local/php5/extensions/”

并添加扩展模块引用：

> extension = curl.so

4.检查并重启Apache

> /usr/local/php5/bin/php -v

执行这个命令时，php会去检查配置文件是否正确，如果有配置错误，这里会报错，可以根据错误信息去排查.

**相关教程:**

另外一种安装mb_string.so扩展的方法见: [http://blog.haohtml.com/archives/13053](http://blog.haohtml.com/archives/13053) [CentOS 5.1安装php mcrypt和mbstring的扩展](http://blog.haohtml.com/archives/13053)