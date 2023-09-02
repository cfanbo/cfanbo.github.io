---
title: linux nginx php木马排查及加固整理
author: admin
type: post
date: 2012-06-04T09:24:53+00:00
url: /archives/13065
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - 安全

---
**1、改变目录和文件属性，禁止写入**

> find -type f -name \*.php –exec chmod 444 {} \;
> find -type d -exec chmod 555 {} \;

注：当然要排除上传目录、缓存目录等；
同时最好禁止chmod函数，攻击者可通过chmod来修改文件只读属性再修改文件

**2、php配置**
禁用危险函数

> passthru,exec,system,chroot,scandir,chgrp,chown,shell\_exec,proc\_open,proc\_get\_status,ini_alter,
> ini\_alter,ini\_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream\_socket\_server,escapeshellcmd,popen,dl,
> syslog,show_source

**3、nginx配置**

限制一些目录执行php文件

> location~^/images/.*\.(php|php5)$
> {
> denyall;
> }
>
> location~^/static/.*\.(php|php5)$
> {
> denyall;
> }
>
> location~\*^/data/(attachment|avatar)/.\*\.(php|php5)$
> {
> denyall;
> }

注：这些目录的限制必须写在

> location~.*\.(php|php5)$
> {
> fastcgi_pass  127.0.0.1:9000;
> fastcgi_index  index.php;
> include fcgi.conf;
> }

的前面，否则限制不生效!
path_info漏洞修正：
在通用fcgi.conf顶部加入

> if ($request_filename ~\* (.\*)\.php) {
> set $php_url $1;
> }
> if (!-e $php_url.php) {
> return 404;
> }

**4、木马查找**

php木马一般含有或者

> find /data/wwwroot/\* -type f -name “\*.php” |xargs grep “eval(” > /root/scan.txt

另外也有上传任意文件的后门,可以用上面的方法查找所有包括”move\_uploaded\_file”的文件.

还有常见的一句话后门：

> grep -r –include=*.php  ‘[^a-z]eval($_POST’ . > grep.txt
> grep -r –include=\*.php  ‘file\_put\_contents(.\*$_POST\[.*\]);’ . > grep.txt

把搜索结果写入文件，下载下来慢慢分析，其他特征木马、后门类似。有必要的话可对全站所有文件来一次特征查找，上传图片肯定有也捆绑的，来次大清洗。

 **5、查找近3天被修改过的文件：**

> find /data/www -mtime -3 -type f -name \*.php

注意：攻击者可能会通过touch函数来修改文件时间属性来避过这种查找，所以touch必须禁止

**6、查找所有图片文件**

查找所有图片文件gif,jpg.有些图片内容里被添加了php后门脚本.有些实际是一个php文件,将扩展名改成了gif了.正常查看的情况下也可以看到显示的图片内容的.这一点很难发现.

曾给客户处理过这类的木马后门的.发现在php脚本里include一个图片文件,经过查看图片文件源代码,发现竟然是php脚本.