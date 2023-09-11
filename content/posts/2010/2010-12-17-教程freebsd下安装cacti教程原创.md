---
title: '[教程]FreeBSD下安装cacti教程(原创)'
author: admin
type: post
date: 2010-12-17T09:16:53+00:00
url: /archives/6988
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - CACTI

---
以下配置环境为:FreeBSD8.1 Nginx0.8.54 PHP5.2.15 Mysql5.1.54-log

**一．首先配置php网站环境**

一般采用的lamp(linux,apache,php,mysql),我们这里用的是FreeBSD的系统,web用的是Nginx,平台搭建教程请参考:

**二.安装sockets扩展**

注意要选择sockets的扩展版本与您所使用的php版本一致,这里我使用的为php5.2.15版本,所以选择了php52-sockets.

> #cd /usr/ports/net/php52-sockets
> #make install clean

上面的命令会产生一个sockets.so的扩展,系统会自动将一行

> extension=sockets.so

信息添加到/usr/local/etc/php/extensions.ini 文件末尾.

如果您确认已经安装过pdo_mysql这个扩展的话,这步可以跳过.

> #cd /usr/ports/databases/php5-pdo_mysql
> #make install clean

**三、安装rrdtool12**

> #cd /usr/ports/databases/rrdtool12
> #make install clean

> #cd /usr/ports/databases/php5-rrdtool
> #make install clean

执行上面php5-rrdtool后,会产生一个rrdtool.so扩展,自动添加一行

>

> extension=rrdtool.so
>

到 /usr/local/etc/php/extensions.ini 文件末尾.

**四、安装net-snmp**

> ****#cd /usr/ports/net-mgmt/net-snmp/
> #make install clean

配置

> #cd /usr/local/share/snmp
> #cp snmpd.conf.example snmpd.conf

编辑snmpd.conf,在文本最后添加 rocommunity public 一行
或者手动通过命令加入也可以

> \# echo ‘rocommunity public’ >> /usr/local/share/snmp/snmpd.conf

启动snmpd服务：

> #echo ‘snmpd_enable=”YES”‘ >> /etc/rc.conf
>
> \# /usr/local/etc/rc.d/snmpd start

通过端号查看服务是否已经安装

> \# netstat -na | grep “LISTEN”
>
> \# sockstat
>
> //199 161 port

可以用动输入以下命令用来查看snmpd服务是否正常

> \# snmpwalk -v 1 -c public 127.0.0.1 system

**五、编辑开机启动项**

> vi /etc/rc.conf
>
> 在内容尾添加以下几行
>
> snmpd_flags=”-a”
> snmpd_pidfile=”/var/run/snmpd.pid”
> snmptrapd_enable=”YES”
> snmptrapd_flags=”-a -p /var/run/snmptrapd.pid”

上面snmpd和snmptrapd两点好像一种是标准写法，一种是缩写方法的。

**六、安装cacti**

> \# cd /data/cacti.mytest.com
> \# fetch http://www.cacti.net/downloads/cacti-0.8.7c.tar.gz
> \# tar -zxvf cacti-0.8.7c.tar.gz
> \# cd cacti-0.8.7c

**配置cacti**

 vi ./include/config.php

$database_hostname = “localhost”;
$database_username = “cacti”; \*/mysql中cacti的用户名/\*
$database_password = “cacti”; \*/mysql中cacti用户的密码/\*
$database_port = “3389”; \*/mysql监控端口/\*

vi ./include/global.php

$database_hostname = “localhost”;
$database_username = “cacti”; \*/mysql中cacti的用户名/\*
$database_password = “cacti”; \*/mysql中cacti用户的密码/\*
$database_port = “3389”;

在mysql中配置cacti数据库及cacti用户信息

> #mysql -u root -p
> mysql>create database cacti default character set utf8;
> mysql> use cacti;
> mysql> source /data/cacti.mytest.com/cacti-0.8.7c/cacti.sql;
> mysql> GRANT ALL ON cacti.* TO cacti@localhost IDENTIFIED BY ‘cacti’;
> mysql> flush privileges;

现在我们在nginx里创建一个虚拟主机来访问cacti,我们在nginx.conf 配置文件里添加以下配置块:

> server {
>
> listen 80;
>
> server_name cacti.mytest.com;
>
> root /data/cacti.mytest.com/cacti-0.8.7c;
>
> location / {
>
> index index.html index.htm index.php;
> }
>
> error_page 500 502 503 504 /50x.html;
>
> location = /50x.html {
> root /usr/local/www/nginx-dist;
> }
>
> \# proxy the PHP scripts to Apache listening on 127.0.0.1:80
> #
> #location ~ .php$ {
> \# proxy_pass http://127.0.0.1;
> #}
>
> \# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
> #
>
> location ~ .php$ {
> fastcgi_pass 127.0.0.1:9000;
> fastcgi_index index.php;
> fastcgi\_param SCRIPT\_FILENAME /data/cacti.mytest.com/cacti-0.8.7c$fastcgi\_script\_name;
> include fastcgi_params;
> }
>
> \# deny access to .htaccess files, if Apache’s document root
> \# concurs with nginx’s one
> #
> #location ~ /.ht {
> \# deny all;
> #}
> }

重启nginx

> #/usr/local/etc/rc.d/nginx reload

现在cacti已经完成.在浏览器里输入cacti访问地址就可以看到cacti的安装界面了,这里我们用的域名为 http://cacti.mytest.com．

**七、创建cacti计划任务**

为了让系统自动采集一些数据,我们还需要执行下面的一些步骤．

> vi /etc/crontab
>
> \*/5 \* \* \* * cacti /usr/local/bin/php /data/cacti.mytest.com/cacti-0.8.7c/poller.php > /dev/null 2>&1

注意，在FreeBSD系统中,cacti采集数据的时候可能会提示

 cat: /proc/meminfo: No such file or directory

类似的错误的，暂时不知道如何解决的

由于刚安装完,没有任何数据信息的,可以通过手动在终端里执行下面的命令即可.

> #/usr/local/bin/php /data/cacti.mytest.com/cacti-0.8.7c/poller.php

如果用的webserver为apache,并且启用了open\_basedir限制目录功能,open\_basedir内容要设置如下:

```
php_admin_value open_basedir "/data/haohtml.com/cacti/:/var/tmp/:/usr/local/bin/php/:/usr/local/bin/snmpwalk/:/usr/local/bin/snmpbulkwalk/:/usr/local/bin/snmpgetnext/:/usr/local/bin/snmpget/:/usr/local/bin/rrdtool/:/usr/bin/perl/:/usr/local/share/rrdtool/fonts/"
```

在设置中字体路径为:

> /usr/local/share/rrdtool/fonts/DejaVuSansMono-Roman.ttf

相关文章： [FreeBSD下用cacti抓取内存信息的方法](http://blog.haohtml.com/index.php/archives/4824)

**高级:安装cacti**

注意：0.8.6f以下的版本有SQL注入漏洞

#cd /usr/ports/net/cacti
#make install FORCE\_PKG\_REGISTER=yes clean ;

因为mysql-client已经装过了，所以需要加上FORCE\_PKG\_REGISTER=yes

#cd /usr/ports/net/cactid
#make install clean;
#ee /usr/local/etc/cactid.conf

DB_Host        localhost
DB_Database    cactidb
DB_User        cacti
DB_Pass        123456
DB_Port        3306

**参考文档：**

我用的是apache,启用了安全模式,禁用了一些函数:

> disable\_functions = passthru,chroot,scandir,chgrp,chown,proc,ini\_alter,ini\_alter,ini\_restore,dl,readlink,symlink,popepassthru,stream\_socket\_server,pfsockopen,openlog,syslog,system,shell_exec

注意:  proc_open, proc_get_status 这两个函数由于要用在监控apache,所以不能禁用的.

**相关教程：**

FreeBSD下安装cacti的memcached监控插件：

在用cacti监控memcached的时候，需要一个python-memcached插件，下载地址：