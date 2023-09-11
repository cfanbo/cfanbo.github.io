---
title: nginx配置支持php的pathinfo模式配置方法
author: admin
type: post
date: 2010-10-08T08:07:51+00:00
url: /archives/5941
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx
 - pathinfo

---
nginx模式不支持pathinfo模式，类似info.php/hello形式的url会被提示找不到页面。下面的通过正则找出实际文件路径和pathinfo部分的方法，让nginx支持pathinfo。

```
location ~ \.php$ {
root           html;
fastcgi_pass   127.0.0.1:9000;
fastcgi_index  index.php;

##通过设置模拟出pathinfo
set $path_info “”;
set $real_script_name $fastcgi_script_name;
if ($fastcgi_script_name ~ “^(.+?\.php)(/.+)$”) {
  set $real_script_name $1;
  set $path_info $2;
}
fastcgi_param SCRIPT_FILENAME $document_root$real_script_name;
fastcgi_param SCRIPT_NAME $real_script_name;
fastcgi_param PATH_INFO $path_info;

include        fastcgi_params;
}

```

**要点：**

1.~ \.php 后面不能有$  以便能匹配所有 \*.php/\* 形式的url

2. 通过设置更改 SCRIPT_FILENAME

我在实际使用张将这段代码融合到了fastcgi_params中。下面是我的nginx配置文件示例：

配置虚拟主机部分，支持pathinfo的nginx代码如下：

\## 在nginx.conf的server部分：

```
server {
listen       8080;
server_name  localhost;

location ~ \.php {
include        fastcgi.conf;
}
}

```

##要点：  \.php 后面没有$，以便匹配所有 \*.php/\* 形式
##重点代码见 fastcgi.conf 开头部分

fastcgi.conf 代码如下：

```
fastcgi_pass   127.0.0.1:9000;
##fastcgi_index  index.php;

set $path_info "";
set $real_script_name $fastcgi_script_name;
if ($fastcgi_script_name ~ "^(.+?\.php)(/.+)$") {
set $real_script_name $1;
set $path_info $2;
}
fastcgi_param SCRIPT_FILENAME $document_root$real_script_name;
fastcgi_param SCRIPT_NAME $real_script_name;
fastcgi_param PATH_INFO $path_info;
## 以上是支持pathinfo的重点部分

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx;

fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

#fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
#fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
#fastcgi_param  REDIRECT_STATUS    200;

```

自己的配置：

```
server
{
listen       80;
server_name  www.touchopenid.com;
index index.html index.htm index.php;
root  /data0/htdocs/openid;

location ~ \.php($|/) {
set  $script     $uri;
set  $path_info  "";
if ($uri ~ "^(.+\.php)(/.+)") {
set  $script     $1;
set  $path_info  $2;
}
fastcgi_pass   127.0.0.1:9000;
include        fastcgi_params;
fastcgi_param  PATH_INFO                $path_info;
fastcgi_param  SCRIPT_FILENAME          $document_root$script;
fastcgi_param  SCRIPT_NAME              $script;
}

```