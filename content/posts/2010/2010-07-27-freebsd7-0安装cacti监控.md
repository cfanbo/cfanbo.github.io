---
title: FreeBSD7.0安装cacti监控
author: admin
type: post
date: 2010-07-27T04:22:39+00:00
url: /archives/4821
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - CACTI

---
FreeBSD 7.0-RELEASE-i386

\# cd /usr/ports/net-mgmt/net-snmp && make install clean
\# cd /usr/ports/net-mgmt/cacti && make install clean

> ucd-snmp不选

\# make pretty-print-run-depends-list

> This port requires package(s) “mysql-client-5.0.67_1” to run.

\# cd /usr/ports/databases/mysql50-server && make install clean
\# echo ‘mysql_enable=”YES”‘ >> /etc/rc.conf
\# /usr/local/etc/rc.d/mysql-server start
\# mysqladmin –user=root create cacti
\# echo “GRANT ALL ON cacti.* TO cactiuser@localhost IDENTIFIED BY ‘cactiuser’; FLUSH PRIVILEGES;” | mysql
\# mysql cacti < /usr/local/share/cacti/cacti.sql


\# echo ‘rocommunity public’ >> /usr/local/share/snmp/snmpd.conf
\# /usr/local/etc/rc.d/snmpd start
\# netstat -na | grep “LISTEN”
\# sockstat

> //199 161 port

# snmpwalk -v 1 -c public 127.0.0.1 system

\# ee /etc/rc.conf

> snmpd_enable=”YES”
> snmpd_flags=”-a”
> snmpd_pidfile=”/var/run/snmpd.pid”
> snmptrapd_enable=”YES”
> snmptrapd_flags=”-a -p /var/run/snmptrapd.pid”

\# ee /usr/local/share/cacti/include/config.php
\# ee /usr/local/etc/apache22/Includes/cacti.conf

> Alias /cacti “/usr/local/share/cacti/”
>
>
> Options None
> AllowOverride None
> Order allow,deny
> Allow from all
>

\# apachectl configtest
\# apachectl restart

[http://localhost/cacti/](http://localhost/cacti/)

完成
登录名：admin
密码：admin
配置完成后。

\# /usr/local/bin/php /usr/local/share/cacti/poller.php
\# crontab -u cacti -e

> \*/5 \* \* \* * /usr/local/bin/php /usr/local/share/cacti/poller.php > /dev/null 2>&1

\# cat /usr/local/share/cacti/log/cacti.log

参考：
[CACTI Version 0.8.7a for FreeBSD 6.2 release 配置全攻略][1]
[FreeBSD下用cacti抓取内存信息的方法\
\
 FreeBSD下安装cacti的memcached监控插件：](http://blog.haohtml.com/archives/4824)

 [1]: http://www.sdlkm.cn/article.asp?id=156