---
title: 'FreeBSD下安装mysqli扩展[原创]'
author: admin
type: post
date: 2011-06-26T06:59:18+00:00
url: /archives/10013
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - mysqli

---
参考原来的文章:,后来发现程序使用的mysqli扩展没有安装,这里介绍安装方法.

此方法在FreeBSD8.2下,php5.2.17和php5.3.6均正常!

由于原来用的ports安装方法,默认的安装包下载到了/usr/ports/distfiles这个目录里了.这里直接使用,如果没有的话,请从网上下载一个安装包,但要注意一定要和已经安装过的php版本一样才可以.

**一.找到mysqli所在位置**

> cd /usr/ports/distfiles/
> tar zxvf php-5.3.6.tar.gz
> cd php-5.3.6/ext/mysqli

**二.安装mysqli**

> /usr/local/bin/phpize
> ————————–
> Configuring for:
> PHP Api Version: 20090626
> Zend Module Api No: 20090626
> Zend Extension Api No: 220090626
> configure.in:3: warning: prefer named diversions
> configure.in:3: warning: prefer named diversions
> ——————————–
> ./configure –with-php-config=/usr/local/bin/php-config –with-mysqli=/usr/local/bin/mysql_config
> make
> make install
> Installing shared extensions:     /usr/local/lib/php/20060613/

如果用的是php5.3版本的可能需要手动复制so文件到”/usr/local/lib/php/20090626″ 目录.

> cp modules/mysqli.so /usr/local/lib/php/20090626

修改php.ini,或者编辑”/usr/local/etc/php/extensions.ini”文件

> vi /usr/local/php/etc/php.ini

在extension区别添加一行

> extension=mysqli.so

即可(系统默认的是dll文件的,这里用so文件实现).

对于php5.2.17版本,需要直接编辑extensions.ini文件,如果编辑的是php.ini的话,会提示错误信息:

> PHP Warning:  PHP Startup: Unable to load dynamic library ‘/usr/local/lib/php/20060613/mysqli.so’ – /usr/local/lib/php/20060613/mysqli.so: Undefined symbol “spl\_ce\_RuntimeException” in Unknown on line 0

查看php模块

> php -m

发现mysqli模块已经加载成功!