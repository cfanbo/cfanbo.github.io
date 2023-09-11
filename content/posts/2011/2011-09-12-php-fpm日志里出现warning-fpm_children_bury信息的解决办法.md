---
title: 'php-fpm日志里出现[WARNING] fpm_children_bury()信息的解决办法'
author: admin
type: post
date: 2011-09-12T14:49:24+00:00
url: /archives/11417
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - php-fpm

---
最近接手nginx+php的WEB环境维护，发现PHP-cgiCPU很好，也造成负载很高，于是在网上找了些资料，并且针对自己的错误，将问题收集再次，并且网上还给了解决方案，所以放在这里留作以后查询

an 11 08:54:01.164292 [NOTICE] fpm_children_make(), line 352: child 10088 (pool default) started

Jan 11 08:54:01.164325 [WARNING] fpm_children_bury(), line 215: child 7985 (pool default) exited on signal 15 SIGTERM after 63.778601 seconds from start

Jan 11 08:54:01.165485 [NOTICE] fpm_children_make(), line 352: child 10089 (pool default) started

Jan 11 08:54:01.165514 [WARNING] fpm_children_bury(), line 215: child 7999 (pool default) exited on signal 15 SIGTERM after 60.297326 seconds from start

Jan 11 08:54:01.166696 [NOTICE] fpm_children_make(), line 352: child 10090 (pool default) started

Jan 11 08:54:01.166727 [WARNING] fpm_children_bury(), line 215: child 8000 (pool default) exited on signal 15 SIGTERM after 60.296946 seconds from start

Jan 11 08:54:01.167855 [NOTICE] fpm_children_make(), line 352: child 10091 (pool default) started

* * *

以上是php日志的警告信息

**解决方法**

**1、提升服务器的文件句柄打开打开**

/etc/security/limits.conf : (增加)

*    soft    nofile    51200

*    hard    nofile    51200

# vi /etc/security/limits.conf 加上

* soft nofile 51200

* hard nofile 51200

**2、提升nginx的进程文件打开数**

nginx.conf : worker_rlimit_nofile 51200;

**3、修改php-fpm.conf文件，主要需要修改2处**。

命令 ulimit -n 查看限制的打开文件数，php-fpm.conf 中的选项rlimit_files 确保和此数值一致。

10240

51200

**4、修改系统内核**

# vi /etc/sysctl.conf

底部添加

fs.file-max=51200

到此结束.

次优化NGINX+php-fpm上传教程： [http://blog.haohtml.com/archives/9277](http://blog.haohtml.com/archives/9277)