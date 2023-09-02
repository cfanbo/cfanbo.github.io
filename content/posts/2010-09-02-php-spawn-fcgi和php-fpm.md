---
title: php spawn-fcgi和php-fpm
author: admin
type: post
date: 2010-09-02T09:13:21+00:00
url: /archives/5530
IM_contentdowned:
 - 1
categories:
 - 服务器

---
spawn-fcgi是一个通用的FastCGI管理服务器

她是lighttpd中的一部份，但目前已经单独成为一个项目，最新的lighttpd没有这一块()，但可以在以前版本中找到她

在lighttpd-1.4.15（ http://www.lighttpd.net/download/lighttpd-1.4.15.tar.gz ）中就有她

Note注：最新的spawn-fcgi可以到lighttpd.net网站搜索“spawn-fcgi”找到她的最新版本发布地址

目前她的下载地址是http://redmine.lighttpd.net/news/2 最新版本是http://www.lighttpd.net/download/spawn-fcgi-1.6.0.tar.gz

> tar -zxvf lighttpd-1.4.15.tar.gz
> cd lighttpd-1.4.15
> ./configure #编译
> make  #因为我不需要安装lighttp而是只需要他其中的某个文件，所以只make就可以了，不需要make install
> cp src/spawn-fcgi /usr/local/bin/spawn-fcgi  #取出spawn-fcgi的程序

下面我们就可以使用 spawn-fcgi 来控制php-cgi的FastCGI进程了

> /usr/local/bin/spawn-fcgi -a 127.0.0.1 -p 9000 -C 5 -u www-data -g www-data -f /usr/bin/php-cgi

参数含义如下:

> -f  指定调用FastCGI的进程的执行程序位置，根据系统上所装的PHP的情况具体设置
> -a  绑定到地址addr
> -p  绑定到端口port
> -s  绑定到unix socket的路径path
> -C  指定产生的FastCGI的进程数，默认为5（仅用于PHP）
> -P  指定产生的进程的PID文件路径
> -u和-g FastCGI使用什么身份（-u 用户 -g 用户组）运行，Ubuntu下可以使用www-data，其他的根据情况配置，如nobody、apache等

**php-fpm**

她同样也是一个PHP FastCGI管理服务器，**是只用于PHP的,**可以在 http://php-fpm.org/download 下载得到.

她其实是PHP源代码的一个补丁，必须将她patch到你的PHP源代码中，在编译安装PHP后才可以使用。

安装方法类似：

> tar zxvf php-5.2.10.tar.gz
> gzip -cd php-5.2.10-fpm-0.5.11.diff.gz | patch -d php-5.2.10 -p1
> cd php-5.2.10/
> ./configure –prefix=/usr/local/webserver/php \
> –with-config-file-path=/usr/local/webserver/php/etc \
> –with-mysql=/usr/local/webserver/mysql \
> –with-mysqli=/usr/local/webserver/mysql/bin/mysql_config \
> –with-iconv-dir=/usr/local \
> –with-freetype-dir \
> –with-jpeg-dir \
> –with-png-dir \
> –with-zlib \
> –with-libxml-dir=/usr \
> –enable-xml \
> –disable-rpath \
> –enable-discard-path \
> –enable-safe-mode \
> –enable-bcmath \
> –enable-shmop \
> –enable-sysvsem \
> –enable-inline-optimization \
> –with-curl \
> –with-curlwrappers \
> –enable-mbregex \
> –enable-fastcgi \
> –enable-fpm \
> –enable-force-cgi-redirect \
> –enable-mbstring \
> –with-mcrypt \
> –with-gd \
> –enable-gd-native-ttf \
> –with-openssl \
> –with-mhash \
> –enable-pcntl \
> –enable-sockets \
> –with-ldap \
> –with-ldap-sasl \
> –with-xmlrpc \
> –enable-zip \
> –enable-soap \
> –without-pear
> make ZEND\_EXTRA\_LIBS=’-liconv’
> make install
> cp php.ini-dist /usr/local/webserver/php/etc/php.ini
> cd ../

[http://topic.csdn.net/u/20100216/22/5809e272-6f67-4248-bde9-99deeae5215b.html](http://topic.csdn.net/u/20100216/22/5809e272-6f67-4248-bde9-99deeae5215b.html)