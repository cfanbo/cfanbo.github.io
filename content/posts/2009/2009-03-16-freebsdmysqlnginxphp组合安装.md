---
title: freebsd+mysql+nginx+php组合安装
author: admin
type: post
date: 2009-03-16T06:52:14+00:00
excerpt: |
 安装mysql
 #cd/usr/ports/databases/mysql51-server
 #make WITH_CHARSET=gbk WITH_XCHARSET=all WITH_PROC_SCOPE_PTH=yes BUILD_OPTIMIZED=yes BUILD_STATIC=yes SKIP_DNS_CHECK=yes WITHOUT_INNODB=yes install clean
 #cp /usr/local/share/mysql/my-small.cnf /etc/my.cnf
 #rehash

 初始化表
 #/usr/local/bin/mysql_install_db --user=mysql#一定要运行此步,否将下面设定权限将会出现错误,因为这句命令会将生在/usr/local/mysql下面将生var及以下目录,是下面的前提条件.
url: /archives/1097
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - mysql
 - nginx
 - php

---
**安装mysql**
#cd/usr/ports/databases/mysql51-server
#make WITH\_CHARSET=gbk WITH\_XCHARSET=all WITH\_PROC\_SCOPE\_PTH=yes BUILD\_OPTIMIZED=yes BUILD\_STATIC=yes SKIP\_DNS\_CHECK=yes WITHOUT\_INNODB=yes install clean
#cp /usr/local/share/mysql/my-small.cnf /etc/my.cnf
#rehash

**初始化表**
#/usr/local/bin/mysql\_install\_db –user=mysql#一定要运行此步,否将下面设定权限将会出现错误,因为这句命令会将生在/usr/local/mysql下面将生var及以下目录,是下面的前提条件.

**安装php**
\# cd /usr/ports/lang/php5
\# make config
[X] CLI Build CLI version
[X] CGI Build CGI version
[ ] APACHE Build **Apache** module //不要安装这个
[ ] DEBUG Enable debug
[X]] SUHOSIN Enable Suhosin protection system
[X] MULTIBYTE Enable **zend** multibyte support
[ ] IPV6 Enable ipv6 support
[ ] REDIRECT Enable force-cgi-redirect support (CGI only)
[ ] DISCARD Enable discard-path support (CGI only)
[X] FASTCGI Enable fastcgi support (CGI only)
[X] PATHINFO Enable path-info-check support (CGI only)
\# make install clean
#cp /usr/local/etc/php.ini-dist /usr/local/etc/php.ini

**安装php5-extensions**
\# cd /usr/ports/lang/php5-extensions/
\# make config
Options for php5-extensions 1.0
————————————————-
[X] FTP **FTP** support
[X] GD
[X] GETTEXT
[X] MBSTRING
[X] MYSQL
[ ] POSIX //去掉.
[ ] SQLITE //去掉.
[X] ZLIB
\# make install clean

**安装Zend Optimizer**
\# cd /usr/ports/devel/ZendOptimizer/
#make install clean
//直接就可以**安装**.不用去fetch好几M的包..知道diskfiles好处了吧.
//你会看到以下提示:
//You have installed the ZendOptimizer package.
//Edit /usr/local/etc/php.ini and add:
//[Zend]
//zend\_optimizer.optimization\_level=15
//zend\_extension\_manager.optimizer=”/usr/local/lib/php/20060613/Optimizer”
//zend\_extension\_manager.optimizer\_ts=”/usr/local/lib/php/20060613/Optimizer\_TS”
//zend_extension=”/usr/local/lib/php/20060613/ZendExtensionManager.so”
//zend\_extension\_ts=”/usr/local/lib/php/20060613/ZendExtensionManager_TS.so”
//\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\*****
//ok根据提示我们继续.
\# ee /usr/local/etc/php.ini
//如果你打开是空白.那一定是忘了
\# cp /usr/local/etc/php.ini-dist /usr/local/etc/php.ini//
//然后再
\# ee /usr/local/etc/php.ini
//在最下边加上.
[Zend]
zend\_optimizer.optimization\_level=15
zend\_extension\_manager.optimizer=”/usr/local/lib/php/20060613/Optimizer”
zend\_extension\_manager.optimizer\_ts=”/usr/local/lib/php/20060613/Optimizer\_TS”
zend_extension=”/usr/local/lib/php/20060613/ZendExtensionManager.so”
zend\_extension\_ts=”/usr/local/lib/php/20060613/ZendExtensionManager_TS.so”

**安装nginx**
#cd /usr/ports/www/nginx/
#make install clean

**安装lighttpd**
#cd /usr/ports/www/lighttpd/
#make install clean

**nginx+mysql开机后自动运行**
#ee /etc/rc.conf
mysql_enable=”YES”
nginx_enable=”YES”

**配置nginx**
#user nobody
删除前面的注释#，改成 user www

location / {
root /usr/local/www/nginx;
index index.html index.htm;
}
在index.html前面添加一个index.php
location / {
root /usr/local/www/nginx;
index index.phpindex.html index.htm;
}

#location ~ \.php$ {
\# fastcgi_pass 127.0.0.1:9000;
\# fastcgi_index index.php;
\# fastcgi\_param SCRIPT\_FILENAME /scripts$fastcgi_script.name;
\# include fastcgi_params;
#}
将前面的#去掉，修改为
location ~ \.php$ {
fastcgi_pass 127.0.0.1:9000;
fastcgi_index index.php;
fastcgi\_param SCRIPT\_FILENAME /usr/local/www/nginx$fastcgi_script.name;
include fastcgi_params;
}

**配置spawn-fcgi**
#ee /etc/rc.local
/usr/local/bin/spawn-fcgi -a 127.0.0.1 -p 9000 -u www -g www -C 4 -f /usr/local/bin/php-cgi
这样spawn-fcgi就能开机自启动了

**phpMyAdmin**
下载phpMyAdmin
lynx ftp.freebsdchina.org/pub/FreeBSD/ports/i386
在database里边找到phpMyAdmin，D 下载，然后save到硬盘
#tar -zvf phpMyAdmin*
然后把解压出来的phpMyAdmin文件mv到/usr/local/www/nginx/

#mv phpMyAdmin /usr/local/www/nginx/phpMyAdmin
#cd /usr/local/www/nginx/phpMyAdmin
#cp config.sample.inc.php config.inc.php
#ee config.inc.php
//找到
$cfg[‘blowfish_secret’] = ”; /\* YOU MUST FILL IN THIS FOR COOKIE AUTH! \*/
//改成
//如果这儿不添的话.他会提示你”**配置**文件现在需要绝密的短语密码(blowfish_secret)。”
$cfg[‘blowfish_secret’] = ‘iambillgates‘; /\* YOU MUST FILL IN THIS FOR COOKIE AUTH! \*/

//继续找
$cfg\[‘Servers’\]\[$i\][‘controluser’] = ‘pmausr’;
$cfg\[‘Servers’\]\[$i\][‘controlpass’] = ‘pmapass’;
//找到这句改成
$cfg\[‘Servers’\]\[$i\][‘controluser’] = ‘root‘;
$cfg\[‘Servers’\]\[$i\][‘controlpass’] = ”;

//打开http://ip/phpMyAdmin
//您直接输入root回车就可以.
//点权限
//root localhost 否 ALL PRIVILEGES 是 编辑权限
//更改密码 执行
//刷新phpMyAdmin页面
//使用新密码登录

**重启机器**
#reboot

这些很粗略，不是很详细，因为安装的时候并没有记录，很多都是靠记忆写的，如果有差错……那就是我的错，这段时间考试，有时间再重新修改。