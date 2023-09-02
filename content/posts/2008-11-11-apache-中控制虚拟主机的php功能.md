---
title: apache 中控制虚拟主机的php功能
author: admin
type: post
date: 2008-11-11T05:19:49+00:00
excerpt: |
 |
 使用情况分以下两种:

 一,在httpd.conf中配置了全局使用php脚本,则使用下面的方法
 在虚拟主机的设置小节中添加php_flag engine on/off 字串7

 如： 字串9


 ServerName xxxxxx.com
 php_flag engine off
 serveralias www.xxxxxx.com
 ServerAdmin webmaster@hanxiao2000.com
 DocumentRoot "/home/xxxxxx/htdocs"

url: /archives/530
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - 虚拟主机
 - php

---
使用情况分以下两种:

**一,在httpd.conf中配置了全局使用php脚本,则使用下面的方法**
在虚拟主机的设置小节中添加php_flag engine on/off 字串7

如： 字串9
ServerName xxxxxx.com
php_flag engine off
serveralias www.xxxxxx.com
ServerAdmin webmaster@hanxiao2000.com
DocumentRoot “/home/xxxxxx/htdocs”
haohtml.com

**二.没有在httpd.conf中配置执行php脚本功能**

在虚拟主机配置中这样改：

  把 AddType   application/x-httpd-php   .php   这句话放到需要运行php的虚拟主机的配置中

  #这个虚拟主机不能运行php

          ServerAdmin   sc@lin.net.cn
          DocumentRoot   d:/www.haohtml.com/
          ServerName   [www.haohtml.com][1]


  #这个虚拟主机可以运行php

          ServerAdmin   sc@lin.net.cn
          DocumentRoot   d:/www.haohtml.com/
          ServerName   [www.haohtml.com][1]
          AddType   application/x-httpd-php   .php
  **说明:如果要禁止某个目录禁止执行php脚本,则为**
                      Deny   from   all



 [1]: http://www.haohtml.com/