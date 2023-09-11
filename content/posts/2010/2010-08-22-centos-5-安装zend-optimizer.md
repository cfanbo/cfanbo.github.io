---
title: CentOs 5 安装Zend Optimizer
author: admin
type: post
date: 2010-08-22T10:36:16+00:00
url: /archives/5251
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos

---
一，下载

```
cd /usr/local/src

wget http://downloads.zend.com/optimizer/3.3.3/ZendOptimizer-3.3.3-linux-glibc23-i386.tar.gz

tar -xzvf ZendOptimizer-3.3.3-linux-glibc23-i386.tar.gz

./ZendOptimizer-3.3.3-linux-glibc23-i386/install.sh
```



二，安装
1，php.ini配置文件目录是：/etc
2，注意Host5156_Vps使用的是lighttpd，而非apache。

三，配置域名目录下php.ini文件
1，/etc/php.ini是总的配置文件。还有一个具体的配置文件位于：/home/httpd/domain.com/php.ini，这个文件也要设置下。
2，把php.ini文件下的[zend]段落复制下来，再添加到/home/httpd/domain.com/php.ini文件中。

```
[Zend]
zend_extension_manager.optimizer=/usr/local/Zend/lib/Optimizer-3.3.3
zend_extension_manager.optimizer_ts=/usr/local/Zend/lib/Optimizer_TS-3.3.3
zend_optimizer.version=3.3.3
zend_extension=/usr/local/Zend/lib/ZendExtensionManager.so
zend_extension_ts=/usr/local/Zend/lib/ZendExtensionManager_TS.so
```



3，重启Apache

`/etc/init.d/httpd restart`