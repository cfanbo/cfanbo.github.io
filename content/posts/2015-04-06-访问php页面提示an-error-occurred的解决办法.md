---
title: 访问php页面提示An Error Occurred的解决办法
author: admin
type: post
date: 2015-04-05T19:47:43+00:00
url: /archives/15555
categories:
 - 服务器
tags:
 - nginx
 - php-fpm

---
今天帮一个朋友解决php页面出现的”An Error Occurred”的错误，返回http的502错误。这里第一反映就是php-fpm.conf配置问题。目前只有这一个网站的，肯定是php这一块出问题了，而php-fpm.conf文件是没有动过的。只有检查php-fpm和nginx.conf的配置了，基本上没有发现什么不当的现象。无意中发现一个nginx和php配置不一致的地方，那就是php-fpm.conf文件里配置的通讯方式为

> **/tmp/php-cgi.sock**

而在nginx.conf里配置的是

fastcgi_pass  127.0.0.1:9000

两者不一致，所以导致了502错误，这里将nginx.conf文件里的配置修改成

\# fastcgi_pass 127.0.0.1:9000;
fastcgi_pass unix:/tmp/php-cgi.sock;

重启nginx即可解决502错误。