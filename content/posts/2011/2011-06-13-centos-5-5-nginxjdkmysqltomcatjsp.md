---
title: CentOS 5.5 Nginx+JDK+MySQL+Tomcat(jsp)
author: admin
type: post
date: 2011-06-13T04:15:34+00:00
url: /archives/9774
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - jdk
 - tomcat

---
**一.安装Nginx**

> [http://blog.haohtml.com/archives/6051](http://blog.haohtml.com/archives/6051)

**二．安装jdk**

> [http://blog.haohtml.com/archives/9765](http://blog.haohtml.com/archives/9765)

**三、安装apache tomcat**
1、下载apache tomcat并安装tomcat

> wget http://labs.renren.com/apache-mirror/tomcat/tomcat-7/v7.0.14/bin/apache-tomcat-7.0.14.tar.gz
> tar zxvf apache-tomcat-7.0.14.tar.gz
> mv apache-tomcat-7.0.14 /usr/local/tomcat
> cp -rf /usr/local/tomcat/webapps/* /www/

2、配置tomcat的server.xml文件，并启动或停止tomcat

> #vim /usr/local/tomcat/conf/server.xml

查找appBase=”webapps”,修改为appBase=”/www”,其中/www 即为网页的根目录。
安装完成后，启动tomcat，默认监听端口为8080

> #/usr/local/tomcat/bin/startup.sh
>
> Using CATALINA_BASE:   /usr/local/tomcat
> Using CATALINA_HOME:   /usr/local/tomcat
> Using CATALINA_TMPDIR: /usr/local/tomcat/temp
> Using JRE\_HOME:        /usr/java/jdk1.6.0\_26/
> Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar

这时可以用netstat -an | grep 8080查看8080端口是否已经监听．

停止tomcat可以使用以下命令：

> #/usr/local/tomcat/bin/shutdown.sh

**四、nginx与tomcat整合**

Nginx与tomcat的整合其实就是只要配置好nginx.conf文件就可以了。
#vim /etc/nginx/nginx.conf  //配置好的nginx.conf文件如下（注意红色部分)

> user              nginx;
> worker_processes  1;
> error_log  /var/log/nginx/error.log;
> pid        /var/run/nginx.pid;
> events {
> use epoll;
> worker_connections  65535;
> }
> http {
> include       /etc/nginx/mime.types;
> default_type  application/octet-stream;
> log\_format  main  ‘$remote\_addr – $remote\_user [$time\_local] “$request” ‘
> ‘$status $body\_bytes\_sent “$http_referer” ‘
> ‘”$http\_user\_agent” “$http\_x\_forwarded_for”‘;
> access_log  /var/log/nginx/access.log  main;
> server\_names\_hash\_bucket\_size  128;
> client\_header\_buffer_size  32k;
> large\_client\_header_buffers  4  32K;
> client\_max\_body_size 8m;
> sendfile        on;
> tcp_nopush     on;
> keepalive_timeout  65;
> #tomcat add start<<
> tcp_nodelay on;
> client_body_buffer_size 512k;
> proxy_connect_timeout 5;
> proxy_read_timeout 60;
> proxy_send_timeout 5;
> proxy_buffer_size 16k;
> proxy_buffers 4 64k;
> proxy_busy_buffers_size 128k;
> proxy_temp_file_write_size 128k;
> #tomcat add end>>
> gzip  on;
> gzip\_min\_length 1k;
> gzip_buffers 4  16k;
> gzip\_http\_version 1.1;
> gzip\_comp\_level 2;
> gzip_types text/plain application/x-javascript text/css application/xml;
> gzip_vary  on;
> #tomcat add start<<
> upstream tomcat_server {
> server 127.0.0.1:8080;
> }
> #tomcat add end>>
> server {
> listen       80;
> server\_name  \_;
> #charset koi8-r;
> #access_log  logs/host.access.log  main;
> location / {
> root   /www;
> index  index.html index.htm index.jsp default.jsp index.do default.do;
> }
> #tomcat add start<<
> if (-d $request_filename)
> {
> rewrite ^/(.*)([^/])$http://$host/$1$2/ permanent;
> }
> location ~ \.(jsp|jspx|do)?$ {
> proxy_set_header Host $host;
> proxy_set_header X-Forwarded-For $remote_addr;
> proxy_passhttp://tomcat_server;
> }
> #tomcat add end>>
> error_page  404              /404.html;
> location = /404.html {
> root   /www;
> }
> \# redirect server error pages to the static page /50x.html
> #
> error_page   500 502 503 504  /50x.html;
> location = /50x.html {
> root   /www;
> }
> \# proxy the PHP scripts to Apache listening on 127.0.0.1:80
> #
> #location ~ \.php$ {
> #    proxy_pass   http://127.0.0.1;
> #}
> \# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
> #
> #ocation ~ \.php$ {
> #    root           html;
> #    fastcgi_pass   127.0.0.1:9000;
> #    fastcgi_index  index.php;
> #    fastcgi\_param  SCRIPT\_FILENAME  /www$fastcgi\_script\_name;
> #    include        fastcgi_params;
> #}
> \# deny access to .htaccess files, if Apache’s document root
> \# concurs with nginx’s one
> #
> #location ~ /\.ht {
> #    deny  all;
> #}
> }
> \# Load config files from the /etc/nginx/conf.d directory
> include /etc/nginx/conf.d/*.conf;
> }

**五、测试**

启动nginx

> #/usr/local/nginx/sbin/nginx

Nginx启动后，可以访问以下URL中的jsp实例程序，检查jsp程序能否运行。

> http://192.168.0.41/examples/jsp/

注意：nginx与tomcat的工作原理是由nginx代理tomcat输出网页，因此如果开启了防火墙，防火墙不用打开8080端口，也一样可以访问jsp页面。

这里没有启用php脚本。