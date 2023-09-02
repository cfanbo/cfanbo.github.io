---
title: '使用Nginx的proxy_cache缓存功能取代Squid[转载]'
author: admin
type: post
date: 2010-04-01T14:16:45+00:00
url: /archives/3165
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx

---
[文章作者：张宴 本文版本：v1.2 最后修改：2009.01.12 转载请注明原文链接： [http://blog.s135.com/nginx_cache/](http://blog.s135.com/nginx_cache/)]

Nginx从0.7.48版本开始，支持了类似Squid的缓存功能。这个缓存是把URL及相关组合当作Key，用md5编码哈希后保存在硬盘上，所以 它可以支持任意URL链接，同时也支持404/301/302这样的非200状态码。虽然目前官方的Nginx Web缓存服务只能为指定URL或状态码设置过期时间，不支持类似Squid的PURGE指令，手动清除指定缓存页面，但是，通过一个第三方的Nginx 模块，可以清除指定URL的缓存。

Nginx的Web缓存服务主要由proxy\_cache相关指令集和fastcgi\_cache 相关指令集构成，前者用于反向代理时，对后端内容源服务器进行缓存，后者主要用于对FastCGI的动态程序进行缓存。两者的功能基本上一样。

最新的Nginx 0.8.32版本，proxy\_cache和fastcgi\_cache已经比较完善，加上第三方的ngx\_cache\_purge模块（用于清除指定 URL的缓存），已经可以完全取代Squid。我们已经在生产环境使用了 Nginx 的 proxy_cache 缓存功能超过两个月，十分稳定，速度不逊于 Squid。

在功能上，Nginx已经具备Squid所拥有的Web缓存加速功能、清除 指定URL缓存的功能。而在性能上，Nginx对多核CPU的利用，胜过Squid不少。另外，在反向代理、负载均衡、健康检查、后端服务器故障转移、 Rewrite重写、易用性上，Nginx也比Squid强大得多。这使得一台Nginx可以同时作为“负载均衡服务器”与“Web缓存服务器”来使用。

* * *

1、Nginx 负载均衡与缓存服务器在 Linux 下的编译安装：



ulimit -SHn 65535

wget [ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.00.tar.gz](ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.00.tar.gz)

tar zxvf pcre-8.00.tar.gz

cd pcre-8.00/

./configure

make && make install

cd ../

wget [http://labs.frickle.com/files/ngx_cache_purge-1.0.tar.gz](http://labs.frickle.com/files/ngx_cache_purge-1.0.tar.gz)

tar zxvf ngx_cache_purge-1.0.tar.gz


wget [http://nginx.org/download/nginx-0.8.32.tar.gz](http://nginx.org/download/nginx-0.8.32.tar.gz)

tar zxvf nginx-0.8.32.tar.gz

cd nginx-0.8.32/

./configure –user=www –group=www –add-module=../ngx_cache_purge-1.0 –prefix=/usr/local/webserver/nginx –with-http_stub_status_module –with-http_ssl_module

make && make install

cd ../


* * *

2、/usr/local/webserver/nginx/conf/nginx.conf 配置文件内容如下：



user  www www;

worker_processes 8;


error_log  /usr/local/webserver/nginx/logs/nginx_error.log  crit;


pid        /usr/local/webserver/nginx/nginx.pid;


#Specifies the value for maximum file descriptors that can be opened by this process.

worker_rlimit_nofile 65535;


events

{

use epoll;

worker_connections 65535;

}


http

{

include       mime.types;

default_type  application/octet-stream;


charset  utf-8;


server_names_hash_bucket_size 128;

client_header_buffer_size 32k;

large_client_header_buffers 4 32k;

client_max_body_size 300m;


sendfile on;

tcp_nopush     on;


keepalive_timeout 60;


tcp_nodelay on;


client_body_buffer_size  512k;

proxy_connect_timeout    5;

proxy_read_timeout       60;

proxy_send_timeout       5;

proxy_buffer_size        16k;

proxy_buffers            4 64k;

proxy_busy_buffers_size 128k;

proxy_temp_file_write_size 128k;


gzip on;

gzip_min_length  1k;

gzip_buffers     4 16k;

gzip_http_version 1.1;

gzip_comp_level 2;

gzip_types       text/plain application/x-javascript text/css application/xml;

gzip_vary on;


#注：proxy_temp_path和proxy_cache_path指定的路径必须在同一分区

proxy_temp_path   /data0/proxy_temp_dir;

#设置Web缓存区名称为cache_one，内存缓存空间大小为200MB，1天没有被访 问的内容自动清除，硬盘缓存空间大小为30GB。

proxy_cache_path  /data0/proxy_cache_dir  levels=1:2   keys_zone=cache_one:200m inactive=1d max_size=30g;


upstream backend_server {

server   192.168.8.43:80 weight=1 max_fails=2 fail_timeout=30s;

server   192.168.8.44:80 weight=1 max_fails=2 fail_timeout=30s;

server   192.168.8.45:80 weight=1 max_fails=2 fail_timeout=30s;

}


server

{

listen       80;

server_name  www.yourdomain.com 192.168.8.42;

index index.html index.htm;

root  /data0/htdocs/www;


location /

{

#如果后端的服务器返回502、504、执行超时等错误，自动将请求转发到upstream负载均衡池中的另一台服务器，实现故障转移。

proxy_next_upstream http_502 http_504 error timeout invalid_header;

proxy_cache cache_one;

#对不同的HTTP状态码设置不同的缓存时间

proxy_cache_valid  200 304 12h;

#以域名、URI、参数组合成Web缓存的Key值，Nginx根据Key值哈希，存储缓存内容到二级缓存目录内

proxy_cache_key $host$uri$is_args$args;

proxy_set_header Host  $host;

proxy_set_header X-Forwarded-For  $remote_addr;

proxy_pass http://backend_server;

expires      1d;

}


# 用于清除缓存，假设一个URL为http://192.168.8.42/test.txt，通过访问http://192.168.8.42 /purge/test.txt就可以清除该URL的缓存。

location ~ /purge(/.*)

{

#设置只允许指定的IP或IP段才可以清除URL缓存。

allow            127.0.0.1;

allow            192.168.0.0/16;

deny            all;

proxy_cache_purge    cache_one   $host$1$is_args$args;

}


# 扩展名以.php、.jsp、.cgi结尾的动态应用程序不缓存。

location ~ .*\.(php|jsp|cgi)?$

{

proxy_set_header Host  $host;

proxy_set_header X-Forwarded-For  $remote_addr;

proxy_pass http://backend_server;

}


access_log  off;

}

}


* * *

3、启动 Nginx：



/usr/local/webserver/nginx/sbin/nginx

* * *

4、清除指定的URL缓存示例：



[![nginx_purge](http://blog.haohtml.com/wp-content/uploads/2010/04/nginx_purge.png)][1]

 [1]: http://blog.haohtml.com/wp-content/uploads/2010/04/nginx_purge.png