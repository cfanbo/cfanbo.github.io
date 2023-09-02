---
title: 运行cacti的问题Cannot connect to MySQL server on ‘localhost’.Please make sure you have specified a valid MySQL database name in ‘include/config.php’
author: admin
type: post
date: 2013-01-17T06:02:44+00:00
url: /archives/13582
categories:
 - 系统架构

---
参考以次的教程 [http://blog.haohtml.com/archives/9428](http://blog.haohtml.com/archives/9428)，在centos安装cacti监控工具，发现在命令行下运行

> php /var/www/html/cacti/poller.php

的时候，提示以下错误

> FATAL: Cannot connect to MySQL server on ‘localhost’. Please make sure you have specified a valid MySQL database name in ‘include/config.php’

而这此配置文件是没有任何问题的,cacti后台访问一切正常的。poller.php是使用/var/lib/mysql/mysql.sock的，

当我在my.cnf里把mysql.sock定义到/var/lib/mysql/mysql.sock时，poller.php可以连接，
但执行mysql就提示错误了，我把mysql.sock的位置改为/tmp/mysql.sock，使用网上提供**解决办法：**

> ln -s /tmp/mysql.sock /var/lib/mysql/mysql.sock

问题解决了。