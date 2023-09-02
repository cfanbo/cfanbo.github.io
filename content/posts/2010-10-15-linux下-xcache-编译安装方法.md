---
title: Linux下 XCache 编译安装方法
author: admin
type: post
date: 2010-10-15T04:43:25+00:00
url: /archives/6128
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux
 - php扩展

---
大部分的人都说XCache的加速效果比eaccelerator好，这里说说编译安装，

这里选择的是稳定版本的1.2.2版本，2.0版本的不稳定。

> wget [http://xcache.lighttpd.net/pub/Releases/1.2.2/xcache-1.2.2.tar.gz](http://xcache.lighttpd.net/pub/Releases/1.2.2/xcache-1.2.2.tar.gz) （下载）
>
> tar -zxf xcache-1.2.2.tar.gz
> cd xcache-1.2.2
>
> /usr/local/php/bin/phpize
> ./configure –enable-xcache –with-php-config=/usr/local/php/bin/php-config
> make
> make install

记录下xcache的安装目录。

编辑php.ini文件，加入Xcache，作为Zend扩展。

> [xcache-common]
> ;; install as zend extension (recommended), normally “$extension_dir/xcache.so”
> zend_extension = /路径/xcache.so
> ;; or install as extension, make sure your extension_dir setting is correct
> ; extension = xcache.so
>
> [xcache.admin]
> xcache.admin.auth = On
> xcache.admin.user = “mOo”
> ; xcache.admin.pass = md5($your_password)
> xcache.admin.pass = “”
>
> [xcache]
> xcache.shm_scheme =        “mmap”
> xcache.size  =               32M
> xcache.count =                 1
> xcache.slots =                8K
> xcache.ttl   =              3600
> xcache.gc_interval =         300
>
> ; Same as aboves but for variable cache
> ; If you don’t know for sure that you need this, you probably don’t
> xcache.var_size  =            0M
> xcache.var_count =             1
> xcache.var_slots =            8K
> xcache.var_ttl   =             0
> xcache.var_maxttl   =          0
> xcache.var\_gc\_interval =     300
>
> ; N/A for /dev/zero
> xcache.readonly_protection = Off
>
> xcache.mmap_path =    “/dev/zero”
>
> xcache.cacher =               On
> xcache.stat   =               On

然后重启下web服务器即可。