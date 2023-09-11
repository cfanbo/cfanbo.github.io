---
title: FreeBSD+Nginx+Mysql+PHP安装记
author: admin
type: post
date: 2009-03-16T14:50:08+00:00
excerpt: |
 折腾了一个晚上，基本上都是用packages安装，php用ports安装，由于PHP只用了FastCGI模式，所以phpmyadmin提示缺少模块而无法安装，最后下载的源码安装。整个过程中，竟然发现最耗费时间的PHP的那些模块！
 其实安装完成后再回过头来看，步骤熟练后，加上编译时间，半个小时足够！

 先做个规划，操作步骤分三块，分别用三个帖子来写，分别是：软件的安装，软件的设置，启动调试及遇到错误说明。

 主要思路：用php-fpm来管理FastCGI。在网上的大多数资料都是用lighttp来安装管理，但是据说php-fpm比那个要强，所以就赶了一回时髦，用了一下php-fpm。
url: /archives/1095
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - mysql
 - nginx
 - php

---
折腾了一个晚上，基本上都是用packages安装，php用ports安装，由于PHP只用了FastCGI模式，所以phpmyadmin提示缺少模块而无法安装，最后下载的源码安装。整个过程中，竟然发现最耗费时间的PHP的那些模块！
其实安装完成后再回过头来看，步骤熟练后，加上编译时间，半个小时足够！

先做个规划，操作步骤分三块，分别用三个帖子来写，分别是：软件的安装，软件的设置，启动调试及遇到错误说明。

主要思路：用php-fpm来管理FastCGI。在网上的大多数资料都是用lighttp来安装管理，但是据说php-fpm比那个要强，所以就赶了一回时髦，用了一下php-fpm。

先列一下安装的东西，其实由安装的软件列表，就可以看出用做的过程，今天晚了，明天写步骤：
QUOTE:
autoconf-2.62       Automatically configure source code on many Un*x platforms
autoconf-wrapper-20071109 Wrapper script for GNU autoconf
e2fsprogs-libuuid-1.41.3_1 UUID library from e2fsprogs package
freetype2-2.3.7     A free and portable TrueType font rendering engine
gettext-0.17_1      GNU gettext package
gmake-3.81_3        GNU version of ‘make’ utility
jpeg-6b_7           IJG’s jpeg compression utilities
kbproto-1.0.3       KB extension headers
libICE-1.0.4_1,1    Inter Client Exchange library for X11
libSM-1.1.0,1       Session Management library for X11
libX11-1.1.99.2,1   X11 library
libXau-1.0.4        Authentication Protocol library for X11
libXaw-1.0.5_1,1    X Athena Widgets library
libXdmcp-1.0.2_1    X Display Manager Control Protocol library
libXext-1.0.4,1     X11 Extension library
libXmu-1.0.4,1      X Miscellaneous Utilities libraries
libXp-1.0.0,1       X print library
libXpm-3.5.7        X Pixmap library
libXt-1.0.5_1       X Toolkit library
libiconv-1.11_1     A character set conversion library
libltdl-1.5.26      System independent dlopen wrapper
libmcrypt-2.5.8     Multi-cipher cryptographic library (used in PHP)
libpthread-stubs-0.1 This library provides weak aliases for pthread functions
libxcb-1.1.93       The X protocol C-language Binding (XCB) library
libxml2-2.7.2_1     XML parser library for GNOME
m4-1.4.11,1         GNU m4
mysql-client-5.0.75 Multithreaded SQL database (client)
mysql-server-5.0.75 Multithreaded SQL database (server)
nginx-0.6.35        Robust and small WWW server
pcre-7.8            Perl Compatible Regular Expressions library
perl-5.8.9          Practical Extraction and Report Language
php5-5.2.8          PHP Scripting Language
php5-bz2-5.2.8      The bz2 shared extension for php
php5-ctype-5.2.8    The ctype shared extension for php
php5-dom-5.2.8      The dom shared extension for php
php5-exif-5.2.8     The exif shared extension for php
php5-fpm-5.2.8      PHP Scripting Language with FastCGI Process Manager
php5-gd-5.2.8       The gd shared extension for php
php5-mbstring-5.2.8 The mbstring shared extension for php
php5-mcrypt-5.2.8   The mcrypt shared extension for php
php5-mysql-5.2.8    The mysql shared extension for php
php5-openssl-5.2.8  The openssl shared extension for php
php5-session-5.2.8  The session shared extension for php
php5-simplexml-5.2.8 The simplexml shared extension for php

php5-spl-5.2.8      The spl shared extension for php
php5-xml-5.2.8      The xml shared extension for php
php5-xmlreader-5.2.8 The xmlreader shared extension for php
php5-zlib-5.2.8     The zlib shared extension for php
phpMyAdmin-3.1.2    A set of PHP-scripts to manage MySQL over the web
pkg-config-0.23_1   A utility to retrieve information about installed libraries
png-1.2.34          Library for manipulating PNG images
printproto-1.0.4    Print extension headers
python25-2.5.2_3    An interpreted object-oriented programming language
t1lib-5.1.2_1,1     A Type 1 Rasterizer Library for UNIX/X11
xcb-proto-1.3       The X protocol C-language Binding (XCB) protocol
xextproto-7.0.4     XExt extension headers
xproto-7.0.14       X11 protocol headers
后来找到phpMyAdmin的安装，老提示少一个pdflib，后来加上-f参数强行跳过去了。

来源:http://bbs2.chinaunix.net/thread-1370635-1-1.html 
