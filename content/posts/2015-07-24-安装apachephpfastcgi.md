---
title: 安装apache+php(fastcgi)
author: admin
type: post
date: 2015-07-24T09:11:45+00:00
url: /archives/15886
categories:
 - 程序开发
tags:
 - php

---
最近有下二次开发的程序，由于源程序使用了zend压缩，提示需要安装 Zend Guard Loader 扩展。而安装zend guard 7后，发现在phpinfo()里检测不到，后来才发现原来zend guard只能使用nfs非安全线程的php，没有办法，重新下载NFS版本的php版本。以下为安装要点：

我用的是WampServer集成环境，于是就想到了把 Apache 换成 FastCGI 模式来跑 PHP5.3 nts 版，这样就可以使用Zend Guard Loader 扩展了。

1、下载 [PHP5.3.28](http://windows.php.net/downloads/releases/php-5.3.28-nts-Win32-VC9-x86.zip) ，解压到 F:/php5.3.28nts ，配置好 php.ini，也顺便把 Zend Guard Loader 扩展配置好。

2、下载 [mod_fcgid-2.3.6-win32-x86.zip](http://archive.apache.org/dist/httpd/binaries/win32/) 或 [http://www.apachelounge.com/download/](http://www.apachelounge.com/download/) 解压 manual、modules 目录中的文件到 f:\wamp\bin\apache\apache2.2.22 对应目录里去。

3、打开 Apache 配置文件 F:\wamp\bin\apache\apache2.2.22\conf\httpd.conf ，用#号注释掉 LoadModule php5\_module “F:/wamp/bin/php/php5.3.13/php5apache2\_2.dll” ，并在下面一行加入 ：

```
LoadModule fcgid_module modules/mod_fcgid.so
```

4、在 httpd.conf 配置文件的最后加入下面的配置：

```
<IfModule mod_fcgid.c>
AddHandler fcgid-script .fcgi .php
#php.ini的存放目录
FcgidInitialEnv PHPRC "F:/php5.3.28nts"
# 设置PHP_FCGI_MAX_REQUESTS大于或等于FcgidMaxRequestsPerProcess，防止php-cgi进程在处理完所有请求前退出
FcgidInitialEnv PHP_FCGI_MAX_REQUESTS 1000
#php-cgi每个进程的最大请求数
FcgidMaxRequestsPerProcess 1000
#php-cgi最大的进程数
FcgidMaxProcesses 3
#最大执行时间
FcgidIOTimeout 120
FcgidIdleTimeout 120
#php-cgi的路径
FcgidWrapper "F:/php5.3.28nts/php-cgi.exe" .php
AddType application/x-httpd-php .php
</IfModule>

```

5、告诉 Apache 执行方式，修改配置如下：

```
<Directory “D:/Web”>
Options Indexes FollowSymLinks Includes ExecCGI
AllowOverride None
Order allow,deny
Allow from all
</Directory>

```

6、最后重启 Apache。
[![apache_factcgi_phpinfo](http://blog.haohtml.com/wp-content/uploads/2015/07/apache_factcgi_phpinfo.png)][1]



参考： [http://www.myxzy.com/post-372.html](http://www.myxzy.com/post-372.html) [Apache下FastCGI模块的众多版本](http://www.javatang.com/archives/2010/01/07/3629356.html)

 [1]: http://blog.haohtml.com/wp-content/uploads/2015/07/apache_factcgi_phpinfo.png