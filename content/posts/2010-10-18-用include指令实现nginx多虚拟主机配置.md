---
title: '[推荐]用include指令实现nginx多虚拟主机配置'
author: admin
type: post
date: 2010-10-18T07:04:39+00:00
url: /archives/6203
IM_contentdowned:
 - 1
categories:
 - 服务器

---
**1.nginx.conf内容如下:**

程序代码


worker_processes 1;

error_log  /host/nginx/logs/error.log  crit;

pid        /host/nginx/logs/nginx.pid;


events {

#使用的网络I/)模型,Linux系统推荐采用epoll模型,FreeBSD系统推荐采用kqueue模型

use epoll;

worker_connections  64;

}


http {

include       /host/nginx/conf/mime.types;

default_type  application/octet-stream;


#charset  gb2312;


server_names_hash_bucket_size 128;

client_header_buffer_size 32k;

large_client_header_buffers 4 32k;

keepalive_timeout 60;

fastcgi_connect_timeout 300;

fastcgi_send_timeout 300;

fastcgi_read_timeout 300;

fastcgi_buffer_size 128k;

fastcgi_buffers 4 128k;

fastcgi_busy_buffers_size 128k;

fastcgi_temp_file_write_size 128k;

client_body_temp_path /host/nginx/client_body_temp;

proxy_temp_path /host/nginx/proxy_temp;

fastcgi_temp_path /host/nginx/fastcgi_temp;

gzip **on**;

gzip_min_length  1k;

gzip_buffers     4 16k;

gzip_http_version 1.0;

gzip_comp_level 2;

zip_types       text/plain application/x-javascript text/css application/xml;

gzip_vary **on**;

client_header_timeout  3m;

client_body_timeout    3m;

send_timeout          3m;

sendfile                **on**;

tcp_nopush              **on**;

tcp_nodelay            **on**;


#设定虚拟主机


include       /host/nginx/conf/vhost/www_test_com.conf;

include       /host/nginx/conf/vhost/www_test1_com.conf;

include       /host/nginx/conf/vhost/www_test2_com.conf;

#也可以使用 include /host/nginx/conf/vhost/*.conf 来代替的,这里支持通配符.

}


================================================================

2.在conf目录下创建虚拟主机配置文件目录vhost,在vhost目录下分别根据域名建立相应的www\_test\_com.conf,www\_test1\_com.conf,www\_test2\_com.conf 3个文件.

** www_test_com.conf配置代码如下:**

server {


listen 202.***.***.***:80;            #换成你的IP地址

client_max_body_size 100M;

server_name  www.test.com;  #换成你的域名

charset gb2312;

index index.html index.htm index.php;

root   /host/wwwroot/test;         #你的站点路径


#打开目录浏览，这样当没有找到index文件，就也已浏览目录中的文件


autoindex  **on**;


**if** (-d $request_filename) {

rewrite ^/(.*)([^/])$ http://$host/$1$2/ permanent;

}


error_page  404              /404.html;

location = /40x.html {

root  /host/wwwroot/test;       #你的站点路径

charset    **on**;

}


# redirect server error pages to the  **static** page /50x.html


#


error_page   500 502 503 504  /50x.html;


location = /50x.html {

root   /host/wwwroot/test;      #你的站点路径

charset    **on**;

}


#将客户端的请求转交给fastcgi


location ~ .*\.(php|php5|php4|shtml|xhtml|phtml)?$ {


fastcgi_pass   127.0.0.1:9000;


include /host/nginx/conf/fastcgi_params;


}


#网站的图片较多，更改较少，将它们在浏览器本地缓存15天


location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$


{


expires      15d;


}


#网站会加载很多JS、CSS，将它们在浏览器本地缓存1天


location ~ .*\.(js|css)?$

{

expires      1d;

}


location /(WEB-INF)/ {

deny all;

}


#设定日志格式


log_format  access  ‘$remote_addr – $remote_user [$time_local] “$request” ‘


‘$status $body_bytes_sent “$http_referer” ‘


‘ “$http_user_agent” $http_x_forwarded_for’;


#设定本虚拟主机的访问日志


access_log  /host/nginx/logs/down/access.log  access;   #日志的路径,每个虚拟机一个,不能相同


#防止nginx做web服务的时候,多server_name的问题. [点击这里查看原文](http://blog.haohtml.com/index.php/archives/1220)

server_name_in_redirect  off;

}


================================================================


3.www\_test1\_com.conf和www\_test2\_com.conf,文件和上面的基本相同,具体的日志内容如下:

 **www\_test1\_com.conf配置如下:**

> #设定日志格式
>
>
> log_format  test1  ‘$remote_addr – $remote_user [$time_local] “$request” ‘
>
>
> ‘$status $body_bytes_sent “$http_referer” ‘
>
>
> ‘”$http_user_agent” $http_x_forwarded_for’;
>
>
> #设定本虚拟主机的访问日志
>
>
> access_log  /host/nginx/logs/test1/test1.log  test1;   #日志的路径,每个虚拟机一个,不能相同
>
>
> server_name_in_redirect  off;

 **www\_test2\_com.conf配置如下:**

> #设定日志格式

> log_format  test2  ‘$remote_addr – $remote_user [$time_local] “$request” ‘
>
>
> ‘$status $body_bytes_sent “$http_referer” ‘
>
>
> ‘”$http_user_agent” $http_x_forwarded_for’;
>
>
> #设定本虚拟主机的访问日志
>
>
> access_log  /host/nginx/logs/test2/test2.log  test2;   #日志的路径,每个虚拟机一个,不能相同
>
>
> server_name_in_redirect  off;

虚拟主机配置： [http://blog.haohtml.com/archives/1222](http://blog.haohtml.com/archives/1222)

对于使用nginx做为web服务的请注意对于多个server_name引起的问题及解决办法. [点击查看](http://blog.haohtml.com/index.php/archives/1220)

而对于Nginx下的虚拟主机会产生跨目录的安全问题，解决办法请参考： [http://blog.haohtml.com/index.php/archives/6899](http://blog.haohtml.com/index.php/archives/6899)