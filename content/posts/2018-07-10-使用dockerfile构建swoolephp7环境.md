---
title: 使用Dockerfile构建Swoole+php7环境
author: admin
type: post
date: 2018-07-10T05:42:55+00:00
url: /archives/17945
categories:
 - 系统架构
tags:
 - docker
 - dockerfile

---


```
FROM php:7.2.7-cli
RUN apt-get update 
    && apt-get  install -y libmemcached-dev zlib1g-dev
RUN pecl  install redis-4.0.1 
    && pecl  install swoole-4.0.1 
    && pecl  install memcached-3.0.4 
    && pecl  install xdebug-2.6.0 
    && docker-php-ext- enable redis swoole memcached xdebug
COPY .  /usr/src/myapp
WORKDIR  /usr/src/myapp
CMD [ "php", "-m" ]
```
构建完环境后，使用方法见： [https://blog.haohtml.com/archives/17925](https://blog.haohtml.com/archives/17925)

这里推荐另一种更简单的方法 [https://github.com/mlocati/docker-php-extension-installer](https://github.com/mlocati/docker-php-extension-installer)，同时支持多个PHP版本，唯一的不足可能是安装时没有办法指定扩展的版本号或者手动修改脚本文件来完成。

推荐文章： [Dockerfile 最佳实践](https://mp.weixin.qq.com/s/BD6YI9cGcfhU2lCvugqOmA)