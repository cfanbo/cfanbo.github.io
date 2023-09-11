---
title: 解除phpMyAdmin 导入大型MySQL数据库文件大小限制
author: admin
type: post
date: 2010-01-08T05:32:15+00:00
url: /archives/2818
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php
 - phpmyadmin

---
今天在WPMZ环境下安装了DEDECMS，朋友将以前网站的数据进行导入时出现了一些问题，提示超出导入大小限制。默认MYSQL只能导入最大2MB的数据，于是我在网上找到了修改的方法，事实证明以下方法是可行的。（修改好后必须重启PHP，可）
**
phpMyAdmin 导入大型数据库文件大小限制配置…**

1. 修改 php.ini 文件中下列3项的值：

**upload\_max\_filesize, memory\_limit 和 post\_max_size**

**upload\_max\_filesize，上传文件大小**

**memory_limit 设置内存**

**post\_max\_size 提交数据的最大值**

为你想改的大小值.

2. 在 phpMyAdmin 的配置文件中修改或加入这个设置：

这个文件一般是在phpMyAdmin目录下的config.inc.php文件

**$cfg[‘ExecTimeLimit’]           = 0;    // maximum execution time in seconds (0 for no limit)**

默认为300秒钟，改为0表示不受限制