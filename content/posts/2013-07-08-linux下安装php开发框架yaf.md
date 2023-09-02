---
title: Linux下安装php开发框架yaf
author: admin
type: post
date: 2013-07-08T09:20:36+00:00
url: /archives/14080
categories:
 - 程序开发
tags:
 - yaf

---
[https://github.com/laruence/php-yaf](https://github.com/laruence/php-yaf)

yaf框架中文手册：

yaf手册：

**1.下载并安装yaf扩展** [http://pecl.php.net/package/yaf]( http://pecl.php.net/package/yaf)

```
#wget http://pecl.php.net/get/yaf-2.2.9.tgz
#tar zxvf yaf-2.2.9.tgz
#cd yaf-2.2.9

[root@bogon yaf-2.2.9]# whereis phpize
phpize: /usr/bin/phpize /usr/share/man/man1/phpize.1.gz
/usr/bin/phpize

[root@bogon yaf-2.2.9]# /usr/bin/phpize
Configuring for:
PHP Api Version: 20090626
Zend Module Api No: 20090626
Zend Extension Api No: 220090626
#whereis php-config
php-config: /usr/bin/php-config /usr/share/man/man1/php-config.1.gz

#./configure --with-php-config=/usr/bin/php-config
#make && make install
```

**2.添加扩展配置到php.ini**

在/etc/php.ini文件里添加一行

> extension=yaf.so

重启webserver　即可.

===========================
如果在./configure的时候遇到以下错误：

> configure: error: in \`/root/yaf-2.2.9′:configure: error: no acceptable C compiler found in $PATHSee \`config.log’ for more details.

说明，没有安装gcc．用 **yum -y install gcc** 安装一下即可．