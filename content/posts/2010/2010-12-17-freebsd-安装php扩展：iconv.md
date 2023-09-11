---
title: freeBSD 安装php扩展：iconv
author: admin
type: post
date: 2010-12-17T10:51:30+00:00
url: /archives/7001
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - php扩展

---
对于Linux下安装php扩展的教程，请参考这里：

FreeBSD上默认安装php的时候不会带iconv扩展，因此不会有iconv这个函数。
利用port方式安装（如果系统上没有port树，参考 [freeBSD 利用portsnap更新port](http://blog.haohtml.com/index.php/archives/830)，利用portsnap获取一份最新的port树），过程如下：
**获取php5源文件包**

> ****#cd /usr/ports/lang/php5
> #make fetch

默认情况下，源码包会下载到/usr/ports/distfiles/目录下

**安装iconv**

> ****#cd ../../distfiles/
> #tar -xjvf php-5.2.11.tar.bz2
> #cd php-5.2.11/ext/iconv
> #phpize
> #./configure
> #make
> #make install
> Installing shared extensions:     /usr/local/lib/php/20060613/
> Installing header files:          /usr/local/include/php/

**将扩展模块写入配置文件**

> #cd /usr/local/etc/php
> #echo extension=iconv.so >> extensions.ini

**查看php模块**

> ****#php -m
> [PHP Modules]
> date
> gd
> **iconv**
> json
> libxml
> mbstring
> mysql
> pcre
> PDO
> pdo_mysql
> Reflection
> session
> SimpleXML
> SPL
> standard
> xml
>
> [Zend Modules]

**最后再修改一下php.ini，将以下注释取消**

> ****[iconv]
> ;iconv.input_encoding = ISO-8859-1
> ;iconv.internal_encoding = ISO-8859-1
> ;iconv.output_encoding = ISO-8859-1

**最后重启php**

> ****#cd /usr/local/etc/rc.d
> \# ./spawn-fcgi restart
> Starting spawn_fcgi.
> spawn-fcgi: child spawned successfully: PID: 39530

**phpinfo()**
[![](http://blog.haohtml.com/wp-content/uploads/2010/12/php_extension_for_iconv.jpg)][1]

OK!

来源：

 [1]: http://blog.haohtml.com/wp-content/uploads/2010/12/php_extension_for_iconv.jpg