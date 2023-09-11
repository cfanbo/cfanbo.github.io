---
title: '[教程]CentOS下安装lnmp教程(最新版2012-02-05)'
author: admin
type: post
date: 2012-02-05T06:00:31+00:00
url: /archives/12473
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - lnmp

---
2013-01-25 更新的安装shell脚本,这里使用的是nginx1.2.6。 [点击查看shell安装脚本](http://blog.haohtml.com/wp-content/uploads/2012/02/sh.txt),**测试环境：**

Centos 6.3 X86_64
PHP 5.3.10
Nginx-1.2.6
memcached-1.4.15.tar.gz

* * *

以下教程参考上次写的lnmp安装教程整理的,部分细节由于软件版本的变更也同时进行了增加和修改.

以下基于x64位操作系统(64位操作系统,64位cpu).查看方法参考: [http://blog.haohtml.com/archives/11093](http://blog.haohtml.com/archives/11093)

**安装环境及软件:**

Centos6.1 X86_64
mysql-5.5.22-linux2.6-x86_64.tar.gz
 [php-5.3.10.tar.gz](http://cn2.php.net/distributions/php-5.3.10.tar.gz) [nginx-1.2.0.tar.gz](http://nginx.org/download/nginx-1.0.11.tar.gz)

以上软件全部为截止当前日期 2012-2-5 为止最新稳定版的软件.

**前期准备工作**

一.安装常用命令

>

> yum -y install wget make zip unzip [patch](http://blog.haohtml.com/archives/5980)
>

//有些命令可能以前安装过.这里就不需要重新安装了,不确定的话,再安装一次也没有关系的,系统会自动跳过安装过的命令的.

=========================================================

二.修改yum源

为了加快安装速度,这里我们使用163的镜像yum源的,否则后面的安装会很费时间的．

>

> cd /etc/yum.repos.d
>

>
>

> mv CentOS-Base.repo CentOS-Base.repo.bak
>

>
>

> wget -O CentOS-Base.repo http://mirrors.163.com/.help/CentOS6-Base-163.repo
>

>
>

> vi CentOS-Base.repo

yum makecache
>

将所有mirrorlist行注释掉,在行首添加#符号即可.有关更多镜像站点,请参考: [http://blog.haohtml.com/index.php/archives/5669](http://blog.haohtml.com/index.php/archives/5669)

如果要使用本地的DVD光盘作为yum源的话,请参考: [http://blog.haohtml.com/archives/5742](http://blog.haohtml.com/archives/5742)

=========================================================

三.安装软件常用命令及常用相关库文件

>

> yum -y install gcc gcc-c++ autoconf libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel libidn libidn-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers libevent libevent-devel
>

===================================

四.下载相关软件

这里操作系统为64位，如果您使用的为32位的话，请下载软件的时候注意一下。

>

> cd ~
>

>
>

> mkdir soft

cd soft
>

wget [http://mysql.spd.co.il/Downloads/MySQL-5.5/mysql-5.5.24-linux2.6-x86_64.tar.gz](http://mysql.spd.co.il/Downloads/MySQL-5.5/mysql-5.5.24-linux2.6-x86_64.tar.gz)

wget [http://pecl.php.net/get/PDO_MYSQL-1.0.2.tgz](http://pecl.php.net/get/PDO_MYSQL-1.0.2.tgz)

wget http://museum.php.net/php5/php-5.3.10.tar.gz

wget [http://nginx.org/download/nginx-1.2.0.tar.gz](http://nginx.org/download/nginx-1.2.0.tar.gz)

wget [http://www.imagemagick.org/download/ImageMagick-6.7.9-0.tar.gz](http://www.imagemagick.org/download/ImageMagick-6.7.9-0.tar.gz)

wget [http://pecl.php.net/get/imagick-2.3.0.tgz](http://pecl.php.net/get/imagick-2.3.0.tgz)

wget [http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.13.1.tar.gz](http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.13.1.tar.gz)

wget [http://pecl.php.net/get/memcache-2.2.5.tgz](http://pecl.php.net/get/memcache-2.2.5.tgz)

wget [http://cdnetworks-kr-1.dl.sourceforge.net/project/eaccelerator/eaccelerator/eAccelerator%200.9.6.1/eaccelerator-0.9.6.1.zip](http://cdnetworks-kr-1.dl.sourceforge.net/project/eaccelerator/eaccelerator/eAccelerator%200.9.6.1/eaccelerator-0.9.6.1.zip)

wget [http://cdnetworks-kr-1.dl.sourceforge.net/project/mcrypt/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz](http://cdnetworks-kr-1.dl.sourceforge.net/project/mcrypt/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz  )

wget [http://cdnetworks-kr-1.dl.sourceforge.net/project/mhash/mhash/0.9.9.9/mhash-0.9.9.9.tar.gz](http://cdnetworks-kr-1.dl.sourceforge.net/project/mhash/mhash/0.9.9.9/mhash-0.9.9.9.tar.gz)

wget [http://cdnetworks-kr-1.dl.sourceforge.net/project/pcre/pcre/8.21/pcre-8.21.tar.gz](http://cdnetworks-kr-1.dl.sourceforge.net/project/pcre/pcre/8.21/pcre-8.21.tar.gz )

===================================

**升级系统内核为3.2.4**

考虑到最近linux内核频繁的出现漏洞,这里将linux内核升级到最新版本(Kernl 3.2.4).内核升级教程参考: [http://blog.haohtml.com/archives/12448](http://blog.haohtml.com/archives/12448) 目前系统为Centos6.1(x86_64bit)版本.默认内核为


> [root@bogon ~]# uname -r
>
> 2.6.32-131.0.15.el6.x86_64

升级后内核为


> [root@bogon ~]# uname -r
>
> 3.2.4

=================================


**一.安装Mysql数据库**

需要安装libaio库

> #yum -y install libaio

否则在初始化mysql系统表的时候会提示以下错误

> Installing MySQL system tables…
> ./bin/mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory

以下为详细教程,也可以参考压缩包里的 INSTALL-BINARY 文件

> groupadd mysql
> useradd -r -g mysql mysql
>
> cd ~/soft/
> tar zxvf mysql-5.5.24-linux2.6-x86_64.tar.gz
> mv mysql-5.5.24-linux2.6-x86_64 /usr/local/mysql
> cd /usr/local/mysql
> chown -R mysql .
> chgrp -R mysql .
> scripts/mysql\_install\_db –user=mysql
> chown -R root .
> chown -R mysql data
>
> \# Next command is optional
> cp support-files/my-medium.cnf /etc/my.cnf
>
> 修改 /etc/my.cnf 文件,将socket文件位置修改如下
> socket=/tmp/mysql.sock
>
> \# create mysql pid work dir
>
> mkdir /var/run/mysqld/
> chown -R mysql:mysql /var/run/mysqld/
>
> bin/mysqld_safe –user=mysql &
>
> \# Next command is optional
> cp support-files/mysql.server /etc/init.d/mysql.server
>
> #修改mysql密码
> bin/mysqladmin -u root password ‘new-password’
> bin/mysql -u root -p

> =======
> 注册系统服务
> cp support-files/mysql.server /etc/init.d/mysqld
>
> #把msql的脚本文件拷到系统的启动目录下
>
> cd /etc/init.d/
> chkconfig –add mysqld #将mysql加到启动服务列表里
> chkconfig mysqld on #让系统启动时自动打开mysql服务,如果指定级别,用–level参数

这里为了以后维护的时候,不用输入完整的路径,做了软链接

> ln -s /usr/local/mysql/bin/mysqldump /usr/sbin/mysqldump
>
> ln -s /usr/local/mysql/bin/mysqld_safe /usr/sbin/mysqld_safe
>
> ln -s /usr/local/mysql/bin/mysqlslap /usr/sbin/mysqlslap
>
> ln -s /usr/local/mysql/bin/mysql /usr/sbin/mysql
>
> ln -s /usr/local/mysql/bin/mysqladmin /usr/sbin/mysqladmin

我们这里将sock放在了/tmp目录里．有时候系统会使用默认的/var/lib/mysql/mysql.sock文件，为了兼容这个情况可以建立一个链接:


> ln -s /tmp/mysql.sock /var/lib/mysql/mysql.sock

=======================================


**二.安装PHP**

首先安装iconv,否则安装php的时候会提示这个错误

>

>  cd ~/soft/
>

>
>

>
>

>
>

> tar zxvf libiconv-1.13.1.tar.gz
>

>
>

> cd libiconv-1.13.1/
>

>
>

> ./configure –prefix=/usr/local
>

>
>

> make
>

>
>

> make install
>

>
>

> cd ../
>

>
>

> #########################################
>

>
>

> tar zxvf libmcrypt-2.5.8.tar.gz

cd libmcrypt-2.5.8/

./configure

make

make install

/sbin/ldconfig

cd libltdl/

./configure –enable-ltdl-install

make

make install

cd ../../
>

>
>

> #########################################
>

>
>

> tar zxvf mhash-0.9.9.9.tar.gz

cd mhash-0.9.9.9/

./configure

make

make install
>

>
>

> cd ../
>

安装php

> tar zxvf php-5.3.10.tar.gz
> cd php-5.3.10
> ./configure –prefix=/usr/local/php –with-config-file-path=/usr/local/php/etc –with-mysql=/usr/local/mysql –with-mysqli=/usr/local/mysql/bin/mysql_config –with-iconv-dir=/usr/local –with-freetype-dir –with-jpeg-dir –with-png-dir –with-zlib –with-libxml-dir=/usr –enable-xml –disable-rpath –enable-safe-mode –enable-bcmath –enable-shmop –enable-sysvsem –enable-inline-optimization –with-curl –with-curlwrappers –enable-mbregex –enable-fpm –enable-mbstring –with-mcrypt –with-gd –enable-gd-native-ttf –with-openssl –with-mhash –enable-pcntl –enable-sockets –with-ldap –with-ldap-sasl –with-xmlrpc –enable-zip –enable-soap –without-pear
> make ZEND\_EXTRA\_LIBS=’-liconv’
> make install
> cp php.ini-production /usr/local/php/etc/php.ini
> cd ../

在configure过程中如果出现以下错误:

> configure: error: Cannot find ldap libraries in /usr/lib.解决办法:

> cp -frp /usr/lib64/libldap* /usr/lib

如果在make的过程中出现以下错误:

> /root/dev/php-5.3.10/sapi/cli/php: error while loading shared libraries:  libmysqlclient.so.18: cannot open shared object file: No such file or  directory
> make: *** [ext/phar/phar.php] Error 127解决办法🙁)

> ln -s /usr/local/mysql/lib/libmysqlclient.so.18  /usr/lib64/

如果用的是32位系统的话,则用

> ln -s /usr/local/mysql/lib/libmysqlclient.so.18  /usr/lib/

如果按上面的操作,再次执行 make ZEND\_EXTRA\_LIBS=’-liconv’ 后提示以下错误:

> “chmod: cannot access \`ext/phar/phar.phar’: No such file or directory”

只需要重装执行上面的configure命令即可.只需要在./configure的后面加上–without-pear 即可解决办法:

如果在make install的时候还提示上面类似的错误,只需要重新从./configure开始再执行一下就可以了.

**三.安装php扩展**

> tar zxvf memcache-2.2.5.tgz
> cd memcache-2.2.5/
> /usr/local/php/bin/phpize
> ./configure –with-php-config=/usr/local/php/bin/php-config
> make
> make install
> cd ../
>
> ########################################
>
> unzip eaccelerator-0.9.6.1.zip
> cd eaccelerator-0.9.6.1/
> /usr/local/php/bin/phpize
> ./configure –enable-eaccelerator=shared –with-php-config=/usr/local/php/bin/php-config
> make
> make install
> cd ../
>
> ########################################
>
> tar zxvf PDO_MYSQL-1.0.2.tgz
> cd PDO_MYSQL-1.0.2/
> /usr/local/php/bin/phpize
> ./configure –with-php-config=/usr/local/php/bin/php-config –with-pdo-mysql=/usr/local/mysql
> make
> make install
> cd ../
>
> ########################################
>
> tar zxvf ImageMagick-6.7.9-0.tar.gz
> cd ImageMagick-6.7.9-0/
> ./configure
> make
> make install
> cd ../
>
> ########################################
>
> tar zxvf imagick-2.3.0.tgz
> cd imagick-2.3.0/
> /usr/local/php/bin/phpize
> ./configure –with-php-config=/usr/local/php/bin/php-config
> make
> make install
> cd ../

===========================

修改php.ini文件,配置扩展
手工修改：查找/usr/local/php/etc/php.ini中的extension_dir = “./”

> #vi /usr/local/php/etc/php.ini

修改为

> extension_dir = “/usr/local/php/lib/php/extensions/no-debug-non-zts-20090626”

并在此行后增加以下几行，然后保存：

> extension = “memcache.so”
> extension = “pdo_mysql.so”
> extension = “imagick.so”

1.再查找output_buffering = Off,修改为 output_buffering = On.
2.为了安装起见，隐藏http头信息里的php信息,查找 expose_php = on 修改为 expose_php = off
3.再查找; cgi.fix_pathinfo=0，把前面的;注释符号删除，改为 cgi.fix_pathinfo=0，预防方法:防止Nginx文件类型错误解析漏洞。
4.找到;date.timezone= 修改为date.timezone = PRC,修正php中于真实时间相关8小时的问题.

配置eAccelerator加速PHP：

> #mkdir -p /usr/local/eaccelerator_cache
> #vi /usr/local/php/etc/php.ini

按shift+g键跳到配置文件的最末尾，加上以下配置信息

> [eaccelerator]
> zend_extension=”/usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/eaccelerator.so”
> eaccelerator.shm_size=”64″
> eaccelerator.cache_dir=”/usr/local/eaccelerator_cache”
> eaccelerator.enable=”1″
> eaccelerator.optimizer=”1″
> eaccelerator.check_mtime=”1″
> eaccelerator.debug=”0″
> eaccelerator.filter=””
> eaccelerator.shm_max=”0″
> eaccelerator.shm_ttl=”3600″
> eaccelerator.shm_prune_period=”3600″
> eaccelerator.shm_only=”0″
> eaccelerator.compress=”1″
> eaccelerator.compress_level=”9″

如果要安装ZendOptimizer的话，参考： [http://blog.haohtml.com/archives/9180](http://blog.haohtml.com/archives/9180).注意扩展路径最后的日期目录在php5.3中发生了变化.zendOptimizer 3.3.9可能不支持php5.3.　 zend 官方推出了zendOptimzer的代替品 **Zend Opcache**，并进行了开源.安装教程请参考： [http://blog.haohtml.com/archives/14646](/archives/14646), 注意同时只能用一个加速软件，同时使用多个加速软件可能会发生冲突．目前opcache和eaccelerator会发生冲突．

配置php-fpm

//创建www用户,php-fpm和nginx统一使用这个

> groupadd www
> useradd -g www www
>
> cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
>
> vi /usr/local/php/etc/php-fpm.conf
>
> 修改以下两行,并将服务用户名和用户所在组(nobody),修改为www
> user = www
> group = www

启用php-fpm

> /usr/local/php/sbin/php-fpm

停止的话,用

> killall php-fpm

**四 安装NGINX**

> #安装正则表达式库,支持rewite
> tar zxvf pcre-8.21.tar.gz
> cd pcre-8.21/
> ./configure
> make && make install
> cd ../
>
> tar zxvf  nginx-1.2.0.tar.gz
> cd nginx-1.2.0/
> ./configure –user=www –group=www –prefix=/usr/local/nginx –with-http\_stub\_status\_module –with-http\_ssl_module
> make && make install
> cd ../
>
> vi /usr/local/nginx/conf/nginx.conf

修改 /usr/local/nginx/conf/nginx.conf,删除user nobody;行前面的注释,并修改为 user www www;
将以下几行前面的注释删除,将修改fastcgi_param后面的路径

> location ~ \.php$ {
> root html;
> fastcgi_pass 127.0.0.1:9000;
> fastcgi_index index.php;
> fastcgi\_param SCRIPT\_FILENAME /usr/local/nginx/html$fastcgi\_script\_name;
> include fastcgi_params;
> }

测试nginx.conf配置文件

> /usr/local/nginx/sbin/nginx -t

启用nginx

> /usr/local/nginx/sbin/nginx

重新加载配置文件

> /usr/local/nginx/sbin/nginx -s reload

**五.全局配置**

>

> vi /etc/rc.local
>

按shift+g快捷键,在末尾增加以下内容：

>

> ulimit -SHn 65535
>

>
>

> /usr/local/php/sbin/php-fpm
>

>
>

> /usr/local/nginx/sbin/nginx
>

测试是否支持php

>

> vi /usr/local/nginx/html/phpinfo.php
>

输入内容

**浏览http://ip地址/phpinfo.php,可以看到php的相关信息,可以查看扩展是否支持.**

以上使用的是nginx的默认配置,为了充分发挥nginx的性能,实际生产过程中,我们还需要对nginx进行一些配置优化,请参考: [Nginx优化配置(转)](http://blog.haohtml.com/index.php/archives/6213)

参考文章: [http://blog.haohtml.com/archives/6051](http://blog.haohtml.com/archives/6051)

**使用技巧:**

在不停止Nginx服务的情况下平滑变更Nginx配置


1、修改/usr/local/nginx/conf/nginx.conf配置文件后，请执行以下命令检查配置文件是否正确：


> /usr/local/nginx/sbin/nginx -t

如果屏幕显示以下两行信息，说明配置文件正确：


> the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
>
>
> the configuration file /usr/local/nginx/conf/nginx.conf was tested successfully

2、平滑重启：


①、对于Nginx 0.8.x版本，现在平滑重启Nginx配置非常简单，执行以下命令即可：


> /usr/local/nginx/sbin/nginx -s reload

②、隐藏nginx版本号：


> 隐藏方法：vim nginx.conf 在http里加入 server_tokens  Off

屏幕显示的即为Nginx主进程号，例如：


6302


这时，执行以下命令即可使修改过的Nginx配置文件生效：


> kill -HUP 6302

或者无需这么麻烦，找到Nginx的Pid文件：


> kill -HUP `cat /usr/local/nginx/nginx.pid`

检查 [ImageMagick](ftp://ftp.imagemagick.org/pub/ImageMagick/ImageMagick.tar.gz) 安装是否成功,也可以用以下命令查看

> convert -version

php里的时间有相关八小时的问题，解决办法为修改/usr/local/php/etc/php.ini里的date.timezone：


> date.timezone = PRC

到此为止．基本上web环境已经配置完成了．为了方便用户可以通过ftp软件上传程序文件，下面我们接着安装vsftpdl软件 [CentOS下安装 vsftpd 虚拟用户设置教程](http://blog.haohtml.com/archives/9084)

======================================


**相关文章：**

- [Nginx优化配置(转)](http://blog.haohtml.com/index.php/archives/6213)
- [nginx配置支持php的pathinfo模式配置方法](http://blog.haohtml.com/archives/5941)
- [网站压力测试工具webbench简介、安装、使用](http://blog.haohtml.com/index.php/archives/6144)
- [[教程]CentOS下安装 vsftpd 虚拟用户设置教程](http://blog.haohtml.com/archives/9084)
- [Linux下 XCache 编译安装方法](http://blog.haohtml.com/index.php/archives/6128)
- [linux下用phpize给PHP动态添加扩展](http://blog.haohtml.com/index.php/archives/6118)
- [ImageMagick及PHP的imagick扩展的安装及配置](http://blog.haohtml.com/index.php/archives/5793)
- [在CentOS 5.5上安装MongoDB](http://blog.haohtml.com/index.php/archives/5813)
- [安装Memcached](http://blog.haohtml.com/archives/11664)
- [centos下安装php-json](http://blog.haohtml.com/index.php/archives/5688)
- [nginx配置支持php的pathinfo模式配置方法](http://blog.haohtml.com/index.php/archives/5941)
- [nginx 目录自动加斜线 “/” 真正最佳](http://blog.haohtml.com/index.php/archives/5937)
- [CentOS配置SSH证书登录验证](http://blog.haohtml.com/index.php/archives/6024)
- [nginx 虚拟目录的配置](http://blog.haohtml.com/index.php/archives/5628)
- [nginx rewrite规则和参考](http://blog.haohtml.com/index.php/archives/5631)
- [Nginx虚拟主机配置](http://blog.haohtml.com/archives/6203)
- [nginx虚拟主机防webshell跨目录](http://blog.haohtml.com/archives/6899)
- [管理员必看:20个Nginx Web服务器最佳安全实践](http://blog.haohtml.com/archives/9256)
- [Ｎginx无缝版本升级](http://blog.haohtml.com/archives/11025)
- [**128M小内存VPS服务器上的配置优化[原创]**](http://blog.s135.com/post/375/)