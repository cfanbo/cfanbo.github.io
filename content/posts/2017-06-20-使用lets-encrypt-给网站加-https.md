---
title: 使用Let’s Encrypt 给网站加 HTTPS
author: admin
type: post
date: 2017-06-20T06:50:56+00:00
url: /archives/17422
categories:
 - 服务器
tags:
 - certbot
 - https
 - ssl

---
> 2017.03.27更新：`/usr/bin/letsencrypt` 被 `/usr/bin/certbot` 替代，更新文章中所用到的命令。参考： [Archlinux Let’s Encrypt Wiki](https://wiki.archlinux.org/index.php/Let%E2%80%99s_Encrypt)

_Let’s Encrypt_ 证书生成不需要手动进行，官方推荐 [certbot](https://certbot.eff.org/) 这套自动化工具来实现。3步轻松搞定：

 1. 下载安装 certbot (Let’s Encrypt项目的自动化工具)
 2. 创建配置文件
 3. 执行证书自动化生成命令

环境：
centos7 64位
nginx

**一、安装certbot**

```
$yum -y install certbot
```

#检查安装是否成功

```
$certbot --help
```

如果安装过程中遇到python的问题，解决办法请参考： [https://blog.haohtml.com/archives/17491](https://blog.haohtml.com/archives/17491)
**二、生成域名证书**

这里使用的域名为 blog.haohtml.com, 网站根目录为 **/data/wwwroot/haohtml/blog**

> #为一个已经存在的站点生成证书文件,一个证书可以多个域名共用，一次也可以生成多个域名谈证书(内容放在了同一个文件里),命令格式如下：
> $certbot certonly –webroot -w /data/wwwroot/haohtml/htdocs -d www.example.com -d example.com -w /var/www/other -d other.example.net -d another.other.example.net

在终端里执行命令

```
$certbot certonly --webroot -w /data/wwwroot/haohtml/blog/ -d blog.haohtml.com
```

会看到以下输出信息：
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Starting new HTTPS connection (1): acme-v01.api.letsencrypt.org
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for blog.haohtml.com
Using the webroot path /data/wwwroot/haohtml/blog for all unmatched domains.
Waiting for verification…
Cleaning up challenges

IMPORTANT NOTES:
– Congratulations! Your certificate and chain have been saved at
/etc/letsencrypt/live/blog.haohtml.com/fullchain.pem. Your cert
will expire on 2017-09-18. To obtain a new or tweaked version of
this certificate in the future, simply run certbot again. To
non-interactively renew \*all\* of your certificates, run “certbot
renew”
– If you like Certbot, please consider supporting our work by:

Donating to ISRG / Let’s Encrypt: https://letsencrypt.org/donate
Donating to EFF: https://eff.org/donate-le

表示生成证书成功，证书目录为/etc/letsencrypt/live/blog.haohtml.com/,我们看以下此目录都有哪些文件

```
$ls -al /etc/letsencrypt/live/blog.haohtml.com/
```

cert.pem chain.pem fullchain.pem privkey.pem README
其中前四个文件均为连接文件

下面我们添加一个启用https的虚拟主机配置文件,内容如下

**三、配置虚拟主机**
添加一个blog.conf文件(nginx会加载指定目录下的所有,conf文件)，内容如下(wordpress)

```
server {
    listen 443;
    server_name blog.haohtml.com;
    root /data/wwwroot/haohtml/blog;

    ssl on;
    ssl_certificate /etc/letsencrypt/live/blog.haohtml.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/blog.haohtml.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/blog.haohtml.com/chain.pem;

    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;
    ssl_prefer_server_ciphers on;
    add_header Strict-Transport-Security max-age=15768000;
    ssl_stapling on;
    ssl_stapling_verify on;

    location / {
            index index.html index.php index.htm index.shtml;
    }

    rewrite ^/sitemap(-+([a-zA-Z0-9_-]+))?.xml$ "/index.php?xml_sitemap=params=$2" last;
    rewrite ^/sitemap(-+([a-zA-Z0-9_-]+))?.xml.gz$ "/index.php?xml_sitemap=params=$2;zip=true" last;
    rewrite ^/sitemap(-+([a-zA-Z0-9_-]+))?.html$ "/index.php?xml_sitemap=params=$2;html=true" last;
    rewrite ^/sitemap(-+([a-zA-Z0-9_-]+))?.html.gz$ "/index.php?xml_sitemap=params=$2;html=true;zip=true" last;

    if (-f $request_filename/index.html){
        rewrite (.*) $1/index.html break;
    }
    if (-f $request_filename/index.php){
                rewrite (.*) $1/index.php;
    }
    if (!-f $request_filename){
                rewrite (.*) /index.php;
    }

    location ^~ /.well-known {
        allow all;
        alias /data/wwwroot/haohtml/blog/.well-known/;
        default_type "text/plain";
        try_files $uri =404;
    }

    location ~* ^/(data|images|data|uploads)/.*.(php|php5)$
    {
            deny all;
    }

    location ~ .php$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  /data/wwwroot/haohtml/blog$fastcgi_script_name;
            include        fastcgi_params;
    }

    location ~ .*.(gif|jpg|jpeg|png|bmp|swf|js|css)$
    {
            expires      30d;
    }

    access_log /var/log/httpd/blog/access.log;
    error_log  /var/log/httpd/blog/error.log;
}

```

如果使用的是Apache的话，修改如下：
修改 apache 配置文件 httpd.conf，找到以下内容并去掉前面的“#”：

```
LoadModule ssl_module modules/mod_ssl.so
```

并添加一行

```
Listen 443
```

添加虚拟主机配置文件 www.conf

```
ServerAdmin admin@linuxeye.com
    DocumentRoot "/data/wwwroot/abc.cn/wwwroot"
    ServerName www.abc.cn

    ErrorLog "/data/wwwlogs/abc.cn_error_apache.log"
    CustomLog "/data/wwwlogs/abc.cn_apache.log" common

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/www.abc.cn/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/www.abc.cn/privkey.pem

<Directory "/data/wwwroot/abc.cn/wwwroot">
    SetOutputFilter DEFLATE
    Options FollowSymLinks ExecCGI
    Require all granted
    AllowOverride All
    Order allow,deny
    Allow from all
    DirectoryIndex index.html index.php

```

**四、重启Nginx，测试https是否生效**

测试一下nginx配置文件是否存在错误

```
$/usr/local/nginx/sbin/nginx -t
```

the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
configuration file /usr/local/nginx/conf/nginx.conf test is successful

```
$/usr/local/nginx/sbin/nginx -s reload
```

此时，打开域名 https://xxxxx 应该可以看到效果了

**五、定时更新证书**

由于 [Let’s Encrypt] 的证书目前有效期为3个月，过期后可以免费重新生成证书，可视为免费证书，只需要添加一个crond脚本即可.(好像每天生成证书有次数限制,一般也用不着如此频繁的操作)

将下行添加到 /etc/crontab文件里,并设定每三个月自动更新一次(脚本不确定是否正确)或者按月份写几个，每三个月为一个周期。

```
0 0 1 */2 * certbot renew
```

或者使用静默升级,每月1号5点

```
00 05 01 * * /usr/bin/certbot renew --quiet && apachectl restart
```

命令执行过程会检查证书的到期日期，如果证书还未到期会提示你的证书尚未到期，输出如下：

```hljs shell
Saving debug log to /var/log/letsencrypt/letsencrypt.log
-------------------------------------------------------------------------------
Processing /etc/letsencrypt/renewal/yourdomain.com.conf
-------------------------------------------------------------------------------
Cert not yet due for renewal
The following certs are not due for renewal yet:
  /etc/letsencrypt/live/yourdomain.com/fullchain.pem (skipped)
No renewals were attempted.
```

**注意：**如果你创建了多个域名的证书，这里也只显示基本域名的信息，但证书更新会对证书包含的所有域名有效。
为确保证书永不过期，需要增加一个cron的定期执行任务，由于更新证书脚本首先会检查证书的到期日期，并且仅当证书距离少于**30**天时才会执行更新，因此可以安全的创建每周甚至每天运行的cron任务。

**六、后期优化**

由于我们以前网使用的是http访问，现在想平滑过度到https，而还要考虑到seo效果，如需要当用户访问http网站时让其自动转到https网站相对应的网址即可,nginx的配置：

```
server {
listen 80;
server_name www.abc.cn;
rewrite ^(.*) https://$server_name$1 permanent;
}
```

Apache的配置(在虚拟主机或者.htaccess文件配置)：

```
RewriteEngine on
RewriteCond %{HTTP_HOST} !^443$ [NC]
RewriteRule ^(.*) https://%{SERVER_NAME}%{REQUEST_URI} [R=301,L]
```

.用专业在线工具测试你的服务器 SSL 安全性
[Qualys SSL Labs](https://www.ssllabs.com/ssltest/index.html) 提供了全面的 SSL 安全性测试，填写你的网站域名，给自己的 HTTPS 配置打个分。

参考： [https://ksmx.me/letsencrypt-ssl-https/](https://ksmx.me/letsencrypt-ssl-https/)