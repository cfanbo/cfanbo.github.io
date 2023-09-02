---
title: 服务器的大用户量的承载方案 Nginx Squid Apache PHP MySQL
author: admin
type: post
date: 2009-05-07T14:05:34+00:00
excerpt: |
 服务器： Intel(R) Xeon(TM) CPU 3.00GHz * 2, 2GB mem, SCISC 硬盘
 操作系统：CentOs4.4，内核版本2.6.9-22.ELsmp，gcc版本3.4.4
 软件：
 Apache 2.2.3（能使用MPM模式）
 PHP 5.2.0（选用该版本是因为5.2.0的引擎相对更高效）
 eAccelerator 0.9.5（加速PHP引擎，同时也可以加密PHP源程序）
 memcache 1.2.0（用于高速缓存常用数据）
 libevent 1.2a（memcache工作机制所需）
 MySQL 5.0.27（选用二进制版本，省去编译工作）
 Nginx 0.5.4（用做负载均衡器）
 squid-2.6.STABLE6（做反向代理的同时提供专业缓存功能）
url: /archives/1349
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - apache
 - nginx
 - Squid

---

一、前言

二、编译安装

三、 安装MySQL、memcache

四、 安装Apache、PHP、eAccelerator、php-memcache

五、 安装Squid

六、后记

一、前言，准备工作

当前，LAMP开发模式是WEB开发的首选，如何搭建一个高效、可靠、稳定的WEB服务器一直是个热门主题，本文就是这个主题的一次尝试。

我们采用的架构图如下：


引用


——–               ———-                           ————-                   ———                 ————

| 客户端 | ===> |负载均衡器| ===> |反向代理/缓存| ===> |WEB服务器| ===> |数据库服务器|

——–               ———-                           ————-                   ———                 ————

Nginx                     Squid                     Apache,PHP                 MySQL

eAccelerator/memcache


准备工作：


引用


服务器： Intel(R) Xeon(TM) CPU 3.00GHz * 2, 2GB mem, SCISC 硬盘

操作系统：CentOs4.4，内核版本2.6.9-22.ELsmp，gcc版本3.4.4

软件：

Apache 2.2.3（能使用MPM模式）

PHP 5.2.0（选用该版本是因为5.2.0的引擎相对更高效）

eAccelerator 0.9.5（加速PHP引擎，同时也可以加密PHP源程序）

memcache 1.2.0（用于高速缓存常用数据）

libevent 1.2a（memcache工作机制所需）

MySQL 5.0.27（选用二进制版本，省去编译工作）

Nginx 0.5.4（用做负载均衡器）

squid-2.6.STABLE6（做反向代理的同时提供专业缓存功能）


二、编译安装

一、) 安装Nginx

1.) 安装

Nginx发音为[engine x]，是由俄罗斯人Igor Sysoev建立的项目,基于BSD许可。据说他当初是F5的成员之一，英文主页：http://nginx.net。俄罗斯的一些大网站已经使用它超过两年多了，一直表现不凡。

Nginx的编译参数如下：


[root@localhost]#./configure –prefix=/usr/local/server/nginx –with-openssl=/usr/include \

–with-pcre=/usr/include/pcre/ –with-http_stub_status_module –without-http_memcached_module \

–without-http_fastcgi_module –without-http_rewrite_module –without-http_map_module \

–without-http_geo_module –without-http_autoindex_module

在这里，需要说明一下，由于Nginx的配置文件中我想用到正则，所以需要 pcre 模块的支持。我已经安装了 pcre 及 pcre-devel 的rpm包，但是 Ngxin 并不能正确找到 .h/.so/.a/.la 文件，因此我稍微变通了一下：


[root@localhost]#mkdir /usr/include/pcre/.libs/

[root@localhost]#cp /usr/lib/libpcre.a /usr/include/pcre/.libs/libpcre.a

[root@localhost]#cp /usr/lib/libpcre.a /usr/include/pcre/.libs/libpcre.la

然后，修改 objs/Makefile 大概在908行的位置上，注释掉以下内容：


./configure –disable-shared

接下来，就可以正常执行 make 及 make install 了。


2.) 修改配置文件 /usr/local/server/nginx/conf/nginx.conf

以下是我的 nginx.conf 内容，仅供参考：


#运行用户

user   nobody nobody;

#启动进程

worker_processes   2;

#全局错误日志及PID文件

error_log   logs/error.log notice;

pid         logs/nginx.pid;

#工作模式及连接数上限

events {

use epoll;

worker_connections       1024;

}

#设定http服务器，利用它的反向代理功能提供负载均衡支持

http {

#设定mime类型

include       conf/mime.types;

default_type   application/octet-stream;

#设定日志格式

log_format main         ‘$remote_addr – $remote_user [$time_local] ‘

‘”$request” $status $bytes_sent ‘

‘”$http_referer” “$http_user_agent” ‘

‘”$gzip_ratio”‘;

log_format download ‘$remote_addr – $remote_user [$time_local] ‘

‘”$request” $status $bytes_sent ‘

‘”$http_referer” “$http_user_agent” ‘

‘”$http_range” “$sent_http_content_range”‘;

#设定请求缓冲

client_header_buffer_size     1k;

large_client_header_buffers   4 4k;

#开启gzip模块

gzip on;

gzip_min_length   1100;

gzip_buffers     4 8k;

gzip_types       text/plain;

output_buffers   1 32k;

postpone_output   1460;

#设定access log

access_log   logs/access.log   main;

client_header_timeout   3m;

client_body_timeout     3m;

send_timeout           3m;

sendfile                 on;

tcp_nopush               on;

tcp_nodelay             on;

keepalive_timeout   65;

#设定负载均衡的服务器列表

upstream mysvr {

#weigth参数表示权值，权值越高被分配到的几率越大

#本机上的Squid开启3128端口

server 192.168.8.1:3128 weight=5;

server 192.168.8.2:80   weight=1;

server 192.168.8.3:80   weight=6;

}

#设定虚拟主机

server {

listen           80;

server_name     192.168.8.1 www.yejr.com;

charset gb2312;

#设定本虚拟主机的访问日志

access_log   logs/www.yejr.com.access.log   main;

#如果访问 /img/*, /js/*, /css/* 资源，则直接取本地文件，不通过squid

#如果这些文件较多，不推荐这种方式，因为通过squid的缓存效果更好

location ~ ^/(img|js|css)/   {

root     /data3/Html;

expires 24h;

}

#对 “/” 启用负载均衡

location / {

proxy_pass       http://mysvr;

proxy_redirect           off;

proxy_set_header         Host $host;

proxy_set_header         X-Real-IP $remote_addr;

proxy_set_header         X-Forwarded-For $proxy_add_x_forwarded_for;

client_max_body_size     10m;

client_body_buffer_size 128k;

proxy_connect_timeout   90;

proxy_send_timeout       90;

proxy_read_timeout       90;

proxy_buffer_size       4k;

proxy_buffers           4 32k;

proxy_busy_buffers_size 64k;

proxy_temp_file_write_size 64k;

}

#设定查看Nginx状态的地址

location /NginxStatus {

stub_status             on;

access_log               on;

auth_basic               “NginxStatus”;

auth_basic_user_file   conf/htpasswd;

}

}

}

备注：conf/htpasswd 文件的内容用 apache 提供的 htpasswd 工具来产生即可，内容大致如下：

3.) 查看 Nginx 运行状态

输入地址 http://192.168.8.1/NginxStatus/，输入验证帐号密码，即可看到类似如下内容：

Active connections: 328

server accepts handled requests

9309     8982         28890

Reading: 1 Writing: 3 Waiting: 324


第一行表示目前活跃的连接数

第三行的第三个数字表示Nginx运行到当前时间接受到的总请求数，如果快达到了上限，就需要加大上限值了。

第四行是Nginx的队列状态


三、 安装MySQL、memcache

1.) 安装MySQL，步骤如下：

[root@localhost]#tar zxf mysql-standard-5.0.27-linux-i686.tar.gz -C /usr/local/server

[root@localhost]#mv /usr/local/server/mysql-standard-5.0.27-linux-i686 /usr/local/server/mysql

[root@localhost]#cd /usr/local/server/mysql

[root@localhost]#./scripts/mysql_install_db –basedir=/usr/local/server/mysql \

–datadir=/usr/local/server/mysql/data –user=nobody

[root@localhost]#cp /usr/local/server/mysql/support-files/my-large.cnf \

/usr/local/server/mysql/data/my.cnf


2.) 修改 MySQL 配置，增加部分优化参数，如下：

[root@localhost]#vi /usr/local/server/mysql/data/my.cnf


主要内容如下：

[mysqld]

basedir = /usr/local/server/mysql

datadir = /usr/local/server/mysql/data

user     = nobody

port     = 3306

socket   = /tmp/mysql.sock

wait_timeout     = 30

long_query_time=1

#log-queries-not-using-indexes = TRUE

log-slow-queries=/usr/local/server/mysql/slow.log

log-error = /usr/local/server/mysql/error.log

external-locking = FALSE

key_buffer_size = 512M

back_log         = 400

table_cache     = 512

sort_buffer_size = 2M

join_buffer_size = 4M

read_buffer_size = 2M

read_rnd_buffer_size     = 4M

myisam_sort_buffer_size = 64M

thread_cache_size       = 32

query_cache_limit       = 2M

query_cache_size         = 64M

thread_concurrency       = 4

thread_stack     = 128K

tmp_table_size   = 64M

binlog_cache_size       = 2M

max_binlog_size = 128M

max_binlog_cache_size   = 512M

max_relay_log_size       = 128M

bulk_insert_buffer_size = 8M

myisam_repair_threads   = 1

skip-bdb

#如果不需要使用innodb就关闭该选项

#skip-innodb

innodb_data_home_dir     = /usr/local/server/mysql/data/

innodb_data_file_path   = ibdata1:256M;ibdata2:256M:autoextend

innodb_log_group_home_dir       = /usr/local/server/mysql/data/

innodb_log_arch_dir     = /usr/local/server/mysql/data/

innodb_buffer_pool_size = 512M

innodb_additional_mem_pool_size = 8M

innodb_log_file_size     = 128M

innodb_log_buffer_size   = 8M

innodb_lock_wait_timeout         = 50

innodb_flush_log_at_trx_commit   = 2

innodb_file_io_threads   = 4

innodb_thread_concurrency       = 16

innodb_log_files_in_group       = 3


以上配置参数请根据具体的需要稍作修改。运行以下命令即可启动 MySQL 服务器：

/usr/local/server/mysql/bin/mysqld_safe \

–defaults-file=/usr/local/server/mysql/data/my.cnf &


由于 MySQL 不是安装在标准目录下，因此必须要修改 mysqld_safe 中的 my_print_defaults 文件所在位置，才能通过

mysqld_safe 来启动 MySQL 服务器。

3.) memcache + libevent 安装编译安装：

[root@localhost]#cd libevent-1.2a

[root@localhost]#./configure –prefix=/usr/ && make && make install

[root@localhost]#cd ../memcached-1.2.0

[root@localhost]#./configure –prefix=/usr/local/server/memcached –with-libevent=/usr/

[root@localhost]#make && make install


备注：如果 libevent 不是安装在 /usr 目录下，那么需要把 libevent-1.2a.so.1 拷贝/链接到 /usr/lib 中，否则

memcached 无法正常加载。运行以下命令来启动 memcached：

[root@localhost]#/usr/local/server/memcached/bin/memcached \

 -l 192.168.8.1 -d -p 10000 -u nobody -m 128


表示用 daemon 的方式启动 memcached，监听在 192.168.8.1 的 10000 端口上，运行用户为 nobody，为其分配

128MB 的内存。


四、 安装Apache、PHP、eAccelerator、php-memcache

四、) 安装Apache、PHP、eAccelerator、php-memcache由于Apache

2下的php静态方式编译十分麻烦，因此在这里采用动态模块(DSO)方式。1.) 安装Apache 2.2.3

[root@localhost]#./configure –prefix=/usr/local/server/apache –disable-userdir –disable-actions \

–disable-negotiation –disable-autoindex –disable-filter –disable-include –disable-status \

–disable-asis –disable-auth –disable-authn-default –disable-authn-file –disable-authz-groupfile \

–disable-authz-host –disable-authz-default –disable-authz-user –disable-userdir \

–enable-expires –enable-module=so


备注：在这里，取消了一些不必要的模块，如果你需要用到这些模块，那么请去掉部分参数。

2.) 安装PHP 5.2.0

[root@localhost]#./configure –prefix=/usr/local/server/php –with-mysql \

–with-apxs2=/usr/local/server/apache/bin/apxs –with-freetype-dir=/usr/ –with-png-dir=/usr/ \

–with-gd=/usr/ –with-jpeg-dir=/usr/ –with-zlib –enable-magic-quotes –with-iconv \

–without-sqlite –without-pdo-sqlite –with-pdo-mysql –disable-dom –disable-simplexml \

–enable-roxen-zts

[root@localhost]#make && make install


备注：如果不需要gd或者pdo等模块，请自行去掉。

3.) 安装eAccelerator-0.9.5

[root@localhost]#cd eAccelerator-0.9.5

[root@localhost]#export PHP_PREFIX=/usr/local/server/php

[root@localhost]#$PHP_PREFIX/bin/phpize

[root@localhost]#./configure –enable-eaccelerator=shared –with-php-config=$PHP_PREFIX/bin/php-config

[root@localhost]#make && make install


4.) 安装memcache模块

[root@localhost]#cd memcache-2.1.0

[root@localhost]#export PHP_PREFIX=/usr/local/server/php

[root@localhost]#$PHP_PREFIX/bin/phpize

[root@localhost]#./configure –enable-eaccelerator=shared –with-php-config=$PHP_PREFIX/bin/php-config

[root@localhost]#make && make install


5.) 修改 php.ini 配置然后修改 php.ini，修改/加入类似以下内容：

extension_dir = “/usr/local/server/php/lib/”

extension=”eaccelerator.so”

eaccelerator.shm_size=”32″       ;设定eaccelerator的共享内存为32MB

eaccelerator.cache_dir=”/usr/local/server/eaccelerator”

eaccelerator.enable=”1″

eaccelerator.optimizer=”1″

eaccelerator.check_mtime=”1″

eaccelerator.debug=”0″

eaccelerator.filter=”*.php”

eaccelerator.shm_max=”0″

eaccelerator.shm_ttl=”0″

eaccelerator.shm_prune_period=”3600″

eaccelerator.shm_only=”0″

eaccelerator.compress=”1″

eaccelerator.compress_level=”9″

eaccelerator.log_file = “/usr/local/server/apache/logs/eaccelerator_log”

eaccelerator.allowed_admin_path = “/usr/local/server/apache/htdocs/ea_admin”

extension=”memcache.so”


在这里，最好是在apache的配置中增加默认文件类型的cache机制，即利用apache的expires模块，新增类似如下几行：

ExpiresActive On

ExpiresByType text/html “access plus 10 minutes”

ExpiresByType text/css “access plus 1 day”

ExpiresByType image/jpg “access 1 month”

ExpiresByType image/gif “access 1 month”

ExpiresByType image/jpg “access 1 month”

ExpiresByType application/x-shockwave-flash “access plus 3 day”


这么设置是由于我的这些静态文件通常很少更新，因此我选择的是”access”规则，如果更新相对比较频繁，可以改用”modification”规则;或者也可以用”access”规则，但是在文件更新的时候，执行一下”touch”命令，把文件的时间刷新一下即可。


五、 安装Squid

五、) 安装Squid

[root@localhost]#./configure –prefix=/usr/local/server/squid –enable-async-io=100 –disable-delay-pools –disable-mem-gen-trace –disable-useragent-log –enable-kill-parent-hack –disable-arp-acl –enable-epoll –disable-ident-lookups –enable-snmp –enable-large-cache-files –with-large-files

[root@localhost]#make && make install


或使用如下安装方法：

[root@localhost]#yum install squid


如果是2.6的内核，才能支持epoll的IO模式，旧版本的内核则只能选择poll或其他模式了;另外，记得带上支持大文件的选项，否则在access

log等文件达到2G的时候就会报错。设定 squid 的配置大概如下内容：

#设定缓存目录为 /var/cache1 和 /var/lib/squid，每次处理缓存大小为128MB，当缓存空间使用达到95%时

#新的内容将取代旧的而不直接添加到目录中，直到空间又下降到90%才停止这一活动

#/var/cache1 最大1024MB，/var/lib/squid 最大 5000MB，都是 16*256 级子目录

cache_dir aufs /var/cache1 1024 16 256

cache_dir aufs /var/lib/squid 5000 16 256

cache_mem 128 MB

cache_swap_low 90

cache_swap_high 95

#设置存储策略等

maximum_object_size 4096 KB

minimum_object_size 0 KB

maximum_object_size_in_memory 80 KB

ipcache_size 1024

ipcache_low 90

ipcache_high 95

cache_replacement_policy lru

memory_replacement_policy lru

#设置超时策略

forward_timeout 20 seconds

connect_timeout 15 seconds

read_timeout 3 minutes

request_timeout 1 minutes

persistent_request_timeout 15 seconds

client_lifetime 15 minutes

shutdown_lifetime 5 seconds

negative_ttl 10 seconds

#限制一个ip最大只能有16个连接

acl OverConnLimit maxconn 16

http_access deny OverConnLimit

#限制baidu spider访问

#acl AntiBaidu req_header User-Agent Baiduspider

#http_access deny AntiBaidu

#常规设置

visible_hostname cache.yejr.com

cache_mgr webmaster@yejr.com

client_persistent_connections off

server_persistent_connections on

cache_effective_user nobody

cache_effective_group nobody

tcp_recv_bufsize 65535 bytes

half_closed_clients off

#设定不缓存的规则

hierarchy_stoplist cgi-bin

acl QUERY urlpath_regex cgi-bin

cache deny QUERY

#不要相信ETag 因为有gzip

acl apache rep_header Server ^Apache

broken_vary_encoding allow apache

#设置access log，并且令其格式和apache的格式一样，方便awstats分析

emulate_httpd_log   on

logformat apache %>a %ui %un [%tl] “%rm %ru HTTP/%rv” %Hs %


初始化和启动squid

[root@localhost]#/usr/local/server/squid/sbin/squid -z

[root@localhost]#/usr/local/server/squid/sbin/squid


第一条命令是先初始化squid缓存哈希子目录，只需执行一次即可。


六、后记

六、后记一、)想要启用squid所需的改变想要更好的利用squid的cache功能，不是把它启用了就可以的，我们需要做以下几个调整：

1、启用apache的 mod_expires 模块，修改 httpd.conf，加入以下内容：

#expiresdefault “modification plus 2 weeks”expiresactive

onexpiresbytype text/html “access plus 10 minutes”expiresbytype

image/gif “modification plus 1 month”expiresbytype image/jpeg “modification

plus 1 month”expiresbytype image/png “modification plus 1

month”expiresbytype text/css “access plus 1 day”expiresbytype

application/x-shockwave-flash “access plus 3 day”

以上配置的作用是规定各种类型文件的cache规则，对那些图片/flash等静态文件总是cache起来，可根据各自的需要做适当调整。

2、修改 php.ini 配置，如下：

session.cache_limiter = nocache

以上配置的作用是默认取消php中的cache功能，避免不正常的cache产生。

3、修改应用程序例如，有一个php程序页面static.php，它存放着某些查询数据库后的结果，并且数据更新并不频繁，于是，我们就可以考虑对其cache。只需在static.php中加入类似如下代码：

header(‘Cache-Control: max-age=86400

,must-revalidate’);header(‘Pragma:’);header(‘Last-Modified: ‘ .

gmdate(‘D, d M Y H:i:s’) . ‘ GMT’ );header(“Expires: ” .gmdate (‘D, d M Y

H:i:s’, time() + ‘86400’ ). ‘ GMT’);

以上代码的意思是，输出一个http头部信息，让squid知道本页面默认缓存时长为一天。

二、)squidclient简要介绍

*取得squid运行状态信息： squidclient -p 80 mgr:info

*取得squid内存使用情况： squidclient -p 80 mgr:mem

*取得squid已经缓存的列表： squidclient -p 80 mgr:objects. use it carefully,it may crash

*取得squid的磁盘使用情况： squidclient -p 80 mgr:diskd

*强制更新某个url：squidclient -p 80 -m PURGE http://www.yejr.com/static.php

*更多的请查看：squidclient-h 或者 squidclient -p 80 mgr: