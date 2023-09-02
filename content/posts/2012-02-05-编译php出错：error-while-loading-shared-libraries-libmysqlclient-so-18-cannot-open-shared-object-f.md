---
title: '编译php出错：error while loading shared libraries: libmysqlclient.so.18: cannot open shared object f'
author: admin
type: post
date: 2012-02-05T06:38:16+00:00
url: /archives/12479
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - php

---
近日在编译php，make的时候出错：

> /root/dev/php-5.3.6/sapi/cli/php: error while loading shared libraries:  libmysqlclient.so.18: cannot open shared object file: No such file or  directory
> make: \*** [ext/phar/phar.php] Error 127

===================================================

**网上找到的解决办法是:**

> ln -s /usr/local/mysql/lib/libmysqlclient.so.18  /usr/lib/

照做后仍然报错，原因是该方法适用于32位系统，64位系统应使用下面的这行

> ln -s /usr/local/mysql/lib/libmysqlclient.so.18  /usr/lib64/

另外：在编译的时候，不写mysql的路径，而使用mysqlnd代替，也可解决该问题的出现。