---
title: nginx上如何支持.htaccess伪静态转向
author: admin
type: post
date: 2010-12-27T03:51:31+00:00
url: /archives/7251
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx

---

我们知道在apache上有一个常用的功能.htaccess转向，只要apache编译的时候指明支持rewrite模块就可以了。

但是换到nginx上方法会有一点不一样，网上很多人说把.htaccess转向规则写到nginx的配置文件里面，这个办法是官方提供的方法之一，肯定是可行的。但是这个方法有一个缺陷：不方便，下次你要更改一个伪静态转向规则的时候还得去nginx的配置文件或者nginx的虚拟网站的配置文件里面去改，相比apache直接在目录下放置.htaccess文件，nginx的这个办法显然很原始。

不过不要紧，其实是有办法的，在nginx的配置文件中 **include .htaccess** 文件就可以实现相同的功能了。

举个例子，我现在要把www.blogguy.cn的.htaccess从apache上迁移到nginx上，可以需要以下几个步骤：

**第一步：修改.htaccess文件**，因为apache的rewrite转向规则跟nginx的转向规则还是有一些不一样的，典型的不一样有nginx的根目录需要写在每行转向的地址前，每行规则必须以分号(;)结束，301或者404等跳转使用不同的格式。

我的apache上htaccess转换到nginx上大致如下

>

> RewriteEngine On
>

>
>

> RewriteBase /
>

>
>

> RewriteRule ^show-([0-9]+)-([0-9]+).html$ index.php?action=show&id=$1&page=$2
>

>
>

> RewriteRule ^category-([0-9]+)-([0-9]+).html$ index.php?action=index&cid=$1&page=$2
>

>
>

> RewriteRule ^archives-([0-9]+)-([0-9]+).html$ index.php?action=index&setdate=$1&page=$2
>

>
>

> RewriteRule ^(archives|search|reg|login|index|links).html$ index.php?action=$1
>

>
>

> RewriteRule ^(comments|tagslist|trackbacks|index)-([0-9]+).html$ index.php?action=$1&page=$2
>

>
>

> rewriteCond %{http_host} ^blogguy.cn [NC]
>

>
>

> rewriteRule ^(.*)$ http://www.blogguy.cn/$1 [R=301,L]
>

>
>

> ErrorDocument 404 http://www.blogguy.cn/
>

转换成nginx的规则

>

> rewrite ^/show-([0-9]+)-([0-9]+).html$ /index.php?action=show&id=$1&page=$2;
>

>
>

> rewrite ^/category-([0-9]+)-([0-9]+).html$ /index.php?action=index&cid=$1&page=$2;
>

>
>

> rewrite ^/archives-([0-9]+)-([0-9]+).html$ /index.php?action=index&setdate=$1&page=$2;
>

>
>

> rewrite ^/(archives|search|reg|login|index|links).html$ /index.php?action=$1;
>

>
>

> rewrite ^/(comments|tagslist|trackbacks|index)-([0-9]+).html$ /index.php?action=$1&page=$2;
>

>
>

> if ($host != ‘www.blogguy.cn’ ) {
>

>
>

> rewrite ^/(.*)$ http://www.blogguy.cn/$1 permanent;
>

>
>

> }
>

>
>

> error_page  404 http://www.blogguy.cn/;
>

**第二步：修改nginx的配置文件，增加include该.htaccess文件**

>

> vi /etc/nginx/sites-available/www.blogguy.cn
>

增加一行：include /var/www/www.blogguycn/.htaccess

修改为你相应的地址。

**第四步：测试并重启**

>

> /etc/init.d/nginx -configtest
>

没有问题的话重启一下：/etc/init.d/nginx  restart

看看效果吧，是不是已经可以了呢？