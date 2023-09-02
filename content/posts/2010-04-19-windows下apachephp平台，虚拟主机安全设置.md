---
title: windows下apache+php平台，虚拟主机安全设置
author: admin
type: post
date: 2010-04-19T06:05:19+00:00
url: /archives/3440
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - 虚拟主机
 - php

---
先按这里的文档对服务器系统安全做设置： [http://blog.haohtml.com/index.php/archives/3438](http://blog.haohtml.com/index.php/archives/3438)

对于php.ini的设置有:
1.修改为安全

> safe_mode = true

2.禁用一些系统函数

> disable\_functions = passthru,exec,system,chroot,scandir,chgrp,chown,shell\_exec,proc\_open,proc\_get\_status,ini\_alter,ini\_alter,ini\_restore,dl,pfsockopen,openlog,syslog,readlink,symlink,popepassthru,stream\_socket\_server

3.禁用com组件调用

将 ;com.allow\_dcom = true 修改为　com.allow\_dcom = false 启用并禁用
4.指定上传文件的临时目录

> upload\_tmp\_dir = “d:\php\upload_tmp”

5.启用特别字符转义功能

> magic\_quotes\_gpc = On

6.关闭错误信息

> display_errors＝Off

7.对于虚拟主机配置的安全主要有:

>
> ServerAdmin zbjywl@163.com
> DocumentRoot “d:/site/ceshi.papake.net”
> ServerName ceshi.papake.net
> DirectoryIndex index.php
>
> #限制在固定的目录里，并授权上传文件临时目录
> php\_admin\_value open\_basedir “D:/site/ceshi.papake.net;D:/php/upload\_tmp”
>
>
> Options Indexes MultiViews
> AllowOverride None
> order allow,deny
> Allow from all
> Options FollowSymLinks Includes
>
>
>