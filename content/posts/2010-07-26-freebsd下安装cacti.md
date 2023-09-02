---
title: '[教程]freebsd下安装cacti教程'
author: admin
type: post
date: 2010-07-26T04:17:08+00:00
url: /archives/4798
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - CACTI

---
**一、安装mysql51-server**
cd /usr/ports/database/mysql51-server

make with-debug=no with-client-ldflags=-all-static with-mysqld-ldflags=-all-static witch-assembler=yes with-pthread=yes enable-thread-safe-client=yes install clean
make install clean

cp /usr/local/share/mysql/my-small.cnf /usr/local/etc/my.cnf

cd /
mkdir mysql
chmod 777 /mysql
ee /usr/local/etc/my.cnf


[client]
port = 3389
sock = /mysql/mysql.sock
default-character-set = utf8
[mysqld]
port = 3389
sock = /mysql/mysql.sock
set-variable=max_connections=1000
set-variable=max\_user\_connections=500
set-variable=wait_timeout=200
report_port = 3389
default-character-set = utf8
skip-networking

ln -s /mysql/mysql.sock /tmp/mysql.sock

**二、安装php5**
cd /usr/ports/lang/php5
make config
[X] CLI Build CLI version
[X] CGI Build CGI version
[ ] APACHE Build Apache module
[ ] DEBUG Enable debug
[X] SUHOSIN Enable Suhosin protection system
[X] MULTIBYTE Enable zend multibyte support
[ ] IPV6 Enable ipv6 support
[X] REDIRECT Enable force-cgi-redirect support (CGI only)
[X] DISCARD Enable discard-path support (CGI only)
[X] FASTCGI Enable fastcgi support (CGI only)
[X] PATHINFO Enable path-info-check support (CGI only)

make instll clean

cp /usr/local/etc/php.ini–recommended /usr/local/etc/php.ini

**三、安装php5-extensions**
cd /usr/ports/lang/php5-extensions
make config
[X] BZ2 bzip2 library support
[X] CTYPE ctype functions
[X] DOM DOM support
[X] FILEINFO fileinfo support
[X] FILTER input filter support
[X] FTP FTP support
[X] GD GD library support
[X] GETTEXT gettext library support
[X] HASH HASH Message Digest Framework
[X] ICONV iconv support
[X] JSON JavaScript Object Serialization support
[X] MBSTRING multibyte string support
[X] MCRYPT Encryption support
[X] MYSQL MySQL database support
[X] NCURSES ncurses support (CLI only)
[X] OPENSSL OpenSSL support
[X] PCNTL pcntl support (CLI only)
[X] PDF PDFlib support (implies GD)
[X] PDO PHP Data Objects Interface (PDO)
[X] PDO_SQLITE PDO sqlite driver
[X] READLINE readline support (CLI only)
[X] SESSION session support
[X] SIMPLEXML simplexml support
 [X] SOCKETS sockets support
[X] SPL Standard PHP Library
[X] SQLITE sqlite support
[X] TOKENIZER tokenizer support
[X] WDDX WDDX support (implies XML)
[X] XML XML support
[X] XMLREADER XMLReader support
[X] XMLWRITER XMLWriter support
[X] ZIP ZIP support
[X] ZLIB ZLIB support

make install clean

**四、安装ZendOptimizer**
cd /usr/ports/devel/ZendOptimizer
make install clean

编辑php.ini
ee /usr/local/etc/php.ini
在最后加上
[Zend]
zend\_optimizer.optimization\_level=15
zend\_extension\_manager.optimizer=”/usr/local/lib/php/20060613/Optimizer”
zend\_extension\_manager.optimizer\_ts=”/usr/local/lib/php/20060613/Optimizer\_TS”
zend_extension=”/usr/local/lib/php/20060613/ZendExtensionManager.so”
zend\_extension\_ts=”/usr/local/lib/php/20060613/ZendExtensionManager_TS.so”

> 注意
> 上面的路径中”20060613″这个是变量,不是固定的.
>
> 解决一个问题
> Failed loading /usr/local/lib/php/20060613/ZendExtensionManager.so: Shared object “libm.so.4” not found, required by “ZendExtensionManager.so”报错
> 解决办法：ln -s /lib/libm.so.5 /usr/lib/libm.so.4即可。

cd /usr/ports/net/php5-sockets
make install clean
cd /usr/ports/databases/php5-pdo_mysql
make install clean

**五、安装rrdtool12**
cd /usr/ports/database/rrdtool12
make install clean

cd /usr/ports/databases/php5-rrdtool
make install clean

执行上面php5-rrdtool后,会产生一个rrdtool.so扩展,自动添加一行

>

> extension=rrdtool.so
>

到 /usr/local/etc/php/extensions.ini 文件末尾.

**六、安装net-snmp**
cd /usr/ports/net-mgmt/net-snmp/
make install clean

cd /usr/local/share/snmp
cp snmpd.conf.example snmpd.conf

编辑snmpd.conf,在文本最后添加 rocommunity public 一行
\# echo ‘rocommunity public’ >> /usr/local/share/snmp/snmpd.conf

**七、安装nginx**
cd /usr/ports/www/nginx
make install clean

获得spawn-fcgi
\# pkg_add -r -v lighttpd
\# cp /usr/local/bin/spawn-fcgi /tmp/
\# pkg\_delete -v lighttpd-1.4.18\_1
\# cp /tmp/spawn-fcgi /usr/local/bin/spawn-fcgi
\# /usr/local/bin/spawn-fcgi -a 127.0.0.1 -p 9000 -u www -g www -f /usr/local/bin/php-cgi

建立fastcgi.sh启动文件
ee /usr/local/etc/rc.d/fastcgi.sh

#!/bin/sh
\# Shell Script to start / stop PHP FastCGI using lighttpd – spawn-fcgi binary file.
\# ————————————————————————-
\# Copyright (c) 2006 nixCraft project
\# This script is licensed under GNU GPL version 2.0 or above
\# ————————————————————————-
\# This script is part of nixCraft shell script collection (NSSC)
\# Visit http://bash.cyberciti.biz/ for more information.
\# ————————————————————————-
PROVIDES=php-cgi
LIGHTTPD_FCGI=/usr/local/bin/spawn-fcgi
SERVER_IP=127.0.0.1
SERVER_PORT=9000
SERVER_USER=www
SERVER_GROUP=www
PHP_CGI=/usr/local/bin/php-cgi
PGREP=/bin/pgrep
KILLALL=/usr/bin/killall
\### No editing below ####
cmd=$1

pcgi_start(){
echo “Starting $PROVIDES…”
$LIGHTTPD\_FCGI -a $SERVER\_IP -p $SERVER\_PORT -u $SERVER\_USER -g $SERVER_GROUP -f

$PHP_CGI
}

pcgi_stop(){
echo “Killing $PROVIDES…”
$KILLALL $PROVIDES
}

pcgi_restart(){
pcgi_stop
pcgi_start
}

pcgi_status(){
PID=\`$PGREP $PROVIDES tail -1\` > /dev/null
[ -z $PID ] && echo “$PROVIDES is not running.” echo “$PROVIDES is running as

pid $PID.”

}

pcgi_help(){
echo “Usage: $0 {startstoprestartstatus}”
}

case ${cmd} in
\[Ss\]\[Tt\]\[Aa\]\[Rr\][Tt]) pcgi_start;;
\[Ss\]\[Tt\]\[Oo\]\[Pp\]) pcgi_stop;;
\[Rr\]\[Ee\]\[Ss\]\[Tt\]\[Aa\]\[Rr\][Tt]) pcgi_restart;;
\[Ss\]\[Tt\]\[Aa\]\[Tt\]\[Uu\]\[Ss\]) pcgi_status 0;;
*) pcgi_help ;;
esac

配置nginx支持php
server {
listen 80;
server_name www.test.com;

access_log /var/log/nginx-access.log main;

location / {
root /usr/local/www;
index index.php index.html index.htm;
}

error_page 500 502 503 504 /50x.html;
location = /50x.html {
root /usr/local/www/nginx-dist;
}

\# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
location ~ \.php$ {
fastcgi_pass 127.0.0.1:9000;
fastcgi_index index.php;
fastcgi\_param SCRIPT\_FILENAME /usr/local/www$fastcgi\_script\_name;
include fastcgi_params;
}

location ~ /\.ht {
deny all;
}
}

测试php
ee /usr/local/www/test.php
phpinfo();
?>

**八、安装cacti**
\# cd /usr/local/www
\# fetch http://www.cacti.net/downloads/cacti-0.8.7c.tar.gz

\# tar cacti-0.8.7c.tar.gz
\# mv cacti-0.8.7c cacti
\# chown -R www wheel
\# chmod 777 /usr/local/cacti rrd log

配置cacti
ee /usr/local/share/cacti/include/config.php
$database_hostname = “localhost”;
$database_username = “cacti”; \*/mysql中cacti的用户名/\*
$database_password = “cacti”; \*/mysql中cacti用户的密码/\*
$database_port = “3389”; \*/mysql监控端口/\*

ee /usr/local/share/cacti/include/global.php
$database_hostname = “localhost”;
$database_username = “cacti”; \*/mysql中cacti的用户名/\*
$database_password = “cacti”; \*/mysql中cacti用户的密码/\*
$database_port = “3389”;

在mysql中配置cacti用户信息
#mysql -u root -p
mysql>create database cacti default character set utf8;
mysql> use cacti;
mysql> source /usr/local/www/cacti/cacti.sql
mysql> GRANT ALL ON cacti.* TO cacti@localhost IDENTIFIED BY ‘cacti’;
mysql> set password for cacti@’localhost’= password(‘cacti’);
mysql> flush privileges;

**九、创建cacti计划任务**

> ****ee /etc/crontab
> \*/5 \* \* \* * cacti /usr/local/bin/php /usr/local/www/cacti/poller.php > /dev/null 2>&1

**十、编辑开机启动项**

> ****vi /etc/rc.conf
> sshd_enable=”YES”
> mysql_enable=”YES”
> nginx_enable=”YES”
>
> sendmail_enable=”NONE”
>
> snmpd_enable=”YES”
> snmpd_flags=”-a”
> snmpd_pidfile=”/var/run/snmpd.pid”
> snmptrapd_enable=”YES”
> snmptrapd_flags=”-a -p /var/run/snmptrapd.pid”

上面snmpd和snmptrapd两点好像一种是标准写法，一种是缩写方法的。

对于关闭sendmail的方法，也可以这样写，只是麻烦一点：

> sendmail_enable=”NONE”
> sendmail\_submit\_enable=”NO”
> sendmail\_outbound\_enable=”NO”
> sendmail\_msp\_queue_enable=”NO”

相关文章: [FreeBSD7.0安装cacti监控](http://blog.haohtml.com/index.php/archives/4821)