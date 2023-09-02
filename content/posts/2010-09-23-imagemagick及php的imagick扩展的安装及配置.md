---
title: ImageMagick及PHP的imagick扩展的安装及配置
author: admin
type: post
date: 2010-09-23T10:06:38+00:00
url: /archives/5793
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - php

---
在看在 ”
[基于CentOS 5.5 搭建nginx +php +php-fpm+mysql高性能php平台](http://blog.haohtml.com/index.php/archives/5791)“的1.7的时候，发现以下两个包，

> wget http://blog.s135.com/soft/linux/nginx_php/imagick/ImageMagick.tar.gz
> wget http://pecl.php.net/get/imagick-2.3.0.tgz

不太清楚两个包的区别是什么的，在网上查了一下，注意区别如下：

[imagick](http://pecl.php.net/package/imagick) 是一个 [PHP](http://www.php.net/) 的扩展，用 [ImageMagick](http://www.imagemagick.org/) 提供的API来进行图片的创建与修改，不过这些操作已经包装到扩展imagick中去了，最终调用的是ImageMagick提供的API.

 ImageMagick是一套软件系列，主要用于图片的创建、编辑以及转换等，详细的解释见ImageMagick的官方网站 [http://www.imagemagick.org/](http://www.imagemagick.org/)，ImageMagick与GD的性能要高很多，如果是在处理大量的图片时更加能体现ImageMagick的性能。

 英文原文介绍如下：

 imagick is a native php extension to create and modify images using the ImageMagick API.


 ImageMagick is a software suite to create, edit, and compose bitmap images.. It can read, convert and write images in a variety of formats (over 100) including DPX, EXR, GIF, JPEG, JPEG-2000, PDF, PhotoCD, PNG, Postscript, SVG, and TIFF.


 著名的图片服务提供商 [Flickr](http://www.flickr.com/) 使用的是ImageMagick，还有 [Yupoo](http://www.yupoo.com/)、 [手机之家](http://www.imobile.com.cn/) 使用的也是ImageMagick。

 1.安装ImageMagick


* * *

 这里主要说说Linux下的安装，Windows下的安装就不说了，Windows下的安装相对简单一些，参考 [http://www.imagemagick.org/script/install-source.php#windows](http://www.imagemagick.org/script/install-source.php#windows)

由于安装imagick扩展时需要依赖ImageMagick的函数库，因此必须要先安装ImageMagick

 从地址ftp://ftp.imagemagick.org/pub/ImageMagick/可以找到ImageMagick的地址

 [root@CentOS_Test_Server software]# wget ftp://ftp.imagemagick.org/pub/ImageMagick/ImageMagick-6.5.3-10.tar.gz

 –19:26:09–  ftp://ftp.imagemagick.org/pub/ImageMagick/ImageMagick-6.5.3-10.tar.gz

 => `ImageMagick-6.5.3-10.tar.gz’

 正在解析主机 ftp.imagemagick.org… 74.63.13.227

 Connecting to ftp.imagemagick.org|74.63.13.227|:21… 已连接。

 正在以 anonymous 登录 … 登录成功！

 ==> SYST … 完成。    ==> PWD … 完成。

 ==> TYPE I … 完成。  ==> CWD /pub/ImageMagick … 完成。

 ==> SIZE ImageMagick-6.5.3-10.tar.gz … 11151919

 ==> PASV … 完成。    ==> RETR ImageMagick-6.5.3-10.tar.gz … 完成。

 长度：11151919 (11M)


 100%[=====================================================================================>] 11,151,919  33.4K/s   in 3m 9s


 19:29:42 (57.7 KB/s) – `ImageMagick-6.5.3-10.tar.gz’ saved [11151919]


 tar zxvf ImageMagick-6.5.3-10.tar.gz

 cd ImageMagick-6.5.3-10

 ./configure –prefix=/usr/local/imagemagick

 make

 make install

 ImageMagick安装完成以后的目录结构如下：

 [root@CentOS_Test_Server imagemagick]# pwd

 /usr/local/imagemagick

 [root@CentOS_Test_Server imagemagick]# ll

 总计 32

 drwxr-xr-x 2 root root 4096 07-21 19:59 bin

 drwxr-xr-x 3 root root 4096 07-21 20:02 include

 drwxr-xr-x 4 root root 4096 07-21 20:07 lib

 drwxr-xr-x 5 root root 4096 07-21 20:07 share

 bin目录下的这些命令都可以通过命令行方式来操作图片

 [root@CentOS_Test_Server imagemagick]# ll bin/

 总计 356

 -rwxr-xr-x 1 root root 24261 07-21 19:59 animate

 -rwxr-xr-x 1 root root 24711 07-21 19:59 compare

 -rwxr-xr-x 1 root root 24273 07-21 19:59 composite

 -rwxr-xr-x 1 root root 24261 07-21 19:59 conjure

 -rwxr-xr-x 1 root root 24261 07-21 19:59 convert

 -rwxr-xr-x 1 root root 24261 07-21 19:59 display

 -rwxr-xr-x 1 root root 24717 07-21 19:59 identify

 -rwxr-xr-x 1 root root 24259 07-21 19:59 import

 -rwxr-xr-x 1 root root  1402 07-21 19:59 Magick-config

 -rwxr-xr-x 1 root root  1458 07-21 19:59 Magick++-config

 -rwxr-xr-x 1 root root  1620 07-21 19:59 MagickCore-config

 -rwxr-xr-x 1 root root  1428 07-21 19:59 MagickWand-config

 -rwxr-xr-x 1 root root 24261 07-21 19:59 mogrify

 -rwxr-xr-x 1 root root 24261 07-21 19:59 montage

 -rwxr-xr-x 1 root root 24259 07-21 19:59 stream

 -rwxr-xr-x 1 root root  1410 07-21 19:59 Wand-config

 [root@CentOS_Test_Server imagemagick]# ll include/

 总计 8

 drwxr-xr-x 5 root root 4096 07-21 20:07 ImageMagick

 [root@CentOS_Test_Server imagemagick]# ll include/ImageMagick/

 总计 32

 drwxr-xr-x 2 root root 4096 07-21 20:07 magick

 drwxr-xr-x 2 root root 4096 07-21 20:07 Magick++

 -rw-r–r– 1 root root  419 07-21 20:07 Magick++.h

 drwxr-xr-x 2 root root 4096 07-21 20:07 wand

 [root@CentOS_Test_Server imagemagick]# ll lib/

 总计 17884

 drwxr-xr-x 4 root root    4096 07-21 20:02 ImageMagick-6.5.3

 -rw-r–r– 1 root root 3123344 07-21 19:59 libMagick++.a

 -rw-r–r– 1 root root 5225066 07-21 19:59 libMagickCore.a

 -rwxr-xr-x 1 root root    1036 07-21 19:59 libMagickCore.la

 lrwxrwxrwx 1 root root      22 07-21 19:59 libMagickCore.so -> libMagickCore.so.2.0.0

 lrwxrwxrwx 1 root root      22 07-21 19:59 libMagickCore.so.2 -> libMagickCore.so.2.0.0

 -rwxr-xr-x 1 root root 3681379 07-21 19:59 libMagickCore.so.2.0.0

 -rwxr-xr-x 1 root root    1089 07-21 19:59 libMagick++.la

 lrwxrwxrwx 1 root root      20 07-21 19:59 libMagick++.so -> libMagick++.so.2.0.0

 lrwxrwxrwx 1 root root      20 07-21 19:59 libMagick++.so.2 -> libMagick++.so.2.0.0

 -rwxr-xr-x 1 root root 2060411 07-21 19:59 libMagick++.so.2.0.0

 -rw-r–r– 1 root root 2360930 07-21 19:59 libMagickWand.a

 -rwxr-xr-x 1 root root    1080 07-21 19:59 libMagickWand.la

 lrwxrwxrwx 1 root root      22 07-21 19:59 libMagickWand.so -> libMagickWand.so.2.0.0

 lrwxrwxrwx 1 root root      22 07-21 19:59 libMagickWand.so.2 -> libMagickWand.so.2.0.0

 -rwxr-xr-x 1 root root 1727376 07-21 19:59 libMagickWand.so.2.0.0

 drwxr-xr-x 2 root root    4096 07-21 20:07 pkgconfig


 通过命令man ImageMagick可以查看ImageMagick手册的内容，特别要注意ImageMagick中的大小写，不要写错了


 2.安装PHP的扩展imagick


* * *

 安装imagick扩展时需要PHP >= 5.1.3，ImageMagick >= 6.2.4

 从 [http://pecl.php.net/package/imagick](http://pecl.php.net/package/imagick) 找到imagick的最新的stable版本

 [root@CentOS_Test_Server software]# wget http://pecl.php.net/get/imagick-2.2.2.tgz

 –23:08:04–  http://pecl.php.net/get/imagick-2.2.2.tgz

 正在解析主机 pecl.php.net… 216.92.131.66

 Connecting to pecl.php.net|216.92.131.66|:80… 已连接。

 已发出 HTTP 请求，正在等待回应… 200 OK

 长度：77212 (75K) [application/octet-stream]

 Saving to: `imagick-2.2.2.tgz.1′


 100%[=====================================================================================>] 77,212      35.1K/s   in 2.1s


 23:08:08 (35.1 KB/s) – `imagick-2.2.2.tgz.1′ saved [77212/77212]

 cd imagick-2.2.2


 用tar zxvf解压.tgz文件时报错，网上说的也是这么解压，还是不行啊，不知道解压报错跟什么有关

 [root@CentOS_Test_Server software]# tar zxvf imagick-2.2.2.tgz

 package.xml

 imagick-2.2.2/examples/polygon.php

 imagick-2.2.2/examples/captcha.php

 imagick-2.2.2/examples/thumbnail.php

 imagick-2.2.2/examples/watermark.php

 imagick-2.2.2/config.m4

 imagick-2.2.2/config.w32

 imagick-2.2.2/CREDITS

 imagick-2.2.2/imagick.c

 imagick-2.2.2/imagick_class.c


 gzip: stdin: invalid compressed data–format violated

 tar: 归档文件中异常的 EOF

 tar: 归档文件中异常的 EOF

 tar: 错误不可恢复：现在退出


 phpize是一个shell脚本，主要是用来进行编译环境的准备，执行以后会生成一些新的文件，为配置、编译及安装作好准备

 [root@CentOS_Test_Server imagick-2.2.2]# /usr/local/webserver/php/bin/phpize

 Configuring for:

 PHP Api Version:         20041225

 Zend Module Api No:      20060613

 Zend Extension Api No:   220060519


 执行phpize命令以前目录imagick-2-2-2中的内容

 [root@CentOS_Test_Server imagick-2.2.2]# ll

 总计 260

 -rw-r–r– 1 root root   4479 02-07 06:11 config.m4

 -rw-r–r– 1 root root   2254 02-07 06:11 config.w32

 -rw-r–r– 1 root root     39 02-07 06:11 CREDITS

 drwxr-xr-x 2 root root   4096 07-21 23:09 examples

 -rw-r–r– 1 root root  94719 02-07 06:11 imagick.c

 -rw-r–r– 1 root root 111616 07-21 23:09 imagick_class.c


 执行phpize命令以后目录imagick-2-2-2中的内容

 [root@CentOS_Test_Server imagick-2.2.2]# ll

 总计 1504

 -rw-r–r– 1 root root  74808 07-21 23:12 acinclude.m4

 -rw-r–r– 1 root root 295888 07-21 23:12 aclocal.m4

 drwxr-xr-x 2 root root   4096 07-21 23:12 autom4te.cache

 drwxr-xr-x 2 root root   4096 07-21 23:12 build

 -rwxr-xr-x 1 root root  43499 07-21 23:12 config.guess

 -rw-r–r– 1 root root   1684 07-21 23:12 config.h.in

 -rw-r–r– 1 root root   4479 02-07 06:11 config.m4

 -rwxr-xr-x 1 root root  31743 07-21 23:12 config.sub

 -rwxr-xr-x 1 root root 453751 07-21 23:12 configure

 -rw-r–r– 1 root root   4646 07-21 23:12 configure.in

 -rw-r–r– 1 root root   2254 02-07 06:11 config.w32

 -rw-r–r– 1 root root     39 02-07 06:11 CREDITS

 drwxr-xr-x 2 root root   4096 07-21 23:09 examples

 -rw-r–r– 1 root root  94719 02-07 06:11 imagick.c

 -rw-r–r– 1 root root 111616 07-21 23:09 imagick_class.c

 -rw-r–r– 1 root root      0 07-21 23:12 install-sh

 -rw-r–r– 1 root root 186760 07-21 23:12 ltmain.sh

 -rw-r–r– 1 root root   5694 07-21 23:12 Makefile.global

 -rw-r–r– 1 root root      0 07-21 23:12 missing

 -rw-r–r– 1 root root      0 07-21 23:12 mkinstalldirs

 -rw-r–r– 1 root root  65036 07-21 23:12 run-tests.php


 让我们来看看imagick的扩展的配置选项

 [root@CentOS_Test_Server imagick-2.2.2]# ./configure –help | more

 `configure’ configures this package to adapt to many kinds of systems.


 Usage: ./configure [OPTION]… [VAR=VALUE]…


 To assign environment variables (e.g., CC, CFLAGS…), specify them as

 VAR=VALUE.  See below for descriptions of some of the useful variables.


 Defaults for the options are specified in brackets.


 Configuration:

 -h, –help              display this help and exit

 –help=short        display options specific to this package

 –help=recursive    display the short help of all the included packages

 -V, –version           display version information and exit

 -q, –quiet, –silent   do not print `checking…’ messages

 –cache-file=FILE   cache test results in FILE [disabled]

 -C, –config-cache      alias for `–cache-file=config.cache’

 -n, –no-create         do not create output files

 –srcdir=DIR        find the sources in DIR [configure dir or `..’]


 Installation directories:

 –prefix=PREFIX         install architecture-independent files in PREFIX

 [/usr/local]

 –exec-prefix=EPREFIX   install architecture-dependent files in EPREFIX

 [PREFIX]


 By default, `make install’ will install all the files in

 `/usr/local/bin’, `/usr/local/lib’ etc.  You can specify

 an installation prefix other than `/usr/local’ using `–prefix’,

 for instance `–prefix=$HOME’.


 For better control, use the options below.


 Fine tuning of the installation directories:

 –bindir=DIR           user executables [EPREFIX/bin]

 –sbindir=DIR          system admin executables [EPREFIX/sbin]

 –libexecdir=DIR       program executables [EPREFIX/libexec]

 –datadir=DIR          read-only architecture-independent data [PREFIX/share]

 –sysconfdir=DIR       read-only single-machine data [PREFIX/etc]

 –sharedstatedir=DIR   modifiable architecture-independent data [PREFIX/com]

 –localstatedir=DIR    modifiable single-machine data [PREFIX/var]

 –libdir=DIR           object code libraries [EPREFIX/lib]

 –includedir=DIR       C header files [PREFIX/include]

 –oldincludedir=DIR    C header files for non-gcc [/usr/include]

 –infodir=DIR          info documentation [PREFIX/info]

 –mandir=DIR           man documentation [PREFIX/man]


 System types:

 –build=BUILD     configure for building on BUILD [guessed]

 –host=HOST       cross-compile to build programs to run on HOST [BUILD]

 –target=TARGET   configure for building compilers for TARGET [HOST]


 Optional Features:

 –disable-FEATURE       do not include FEATURE (same as –enable-FEATURE=no)

 –enable-FEATURE[=ARG]  include FEATURE [ARG=yes]

 –enable-shared=PKGS  build shared libraries default=yes

 –enable-static=PKGS  build static libraries default=yes

 –enable-fast-install=PKGS  optimize for fast installation default=yes

 –disable-libtool-lock  avoid locking (might break parallel builds)


 Optional Packages:

 –with-PACKAGE[=ARG]    use PACKAGE [ARG=yes]

 –without-PACKAGE       do not use PACKAGE (same as –with-PACKAGE=no)

 –with-libdir=NAME      Look for libraries in …/NAME rather than …/lib

 –with-php-config=PATH  Path to php-config php-config

 –with-imagick=DIR     Enables the imagick extension. DIR is the prefix to Imagemagick installation directory.

 –with-imagick-gm=DIR  GraphicsMagick backend. NO LONGER SUPPORTED!

 –with-gnu-ld           assume the C compiler uses GNU ld default=no

 –with-pic              try to use only PIC/non-PIC objects default=use both

 –with-tags=TAGS      include additional configurations automatic


 Some influential environment variables:

 CC          C compiler command

 CFLAGS      C compiler flags

 LDFLAGS     linker flags, e.g. -L if you have libraries in a

 nonstandard directory

 CPPFLAGS    C/C++ preprocessor flags, e.g. -I if you have

 headers in a nonstandard directory

 CPP         C preprocessor


 Use these variables to override the choices made by `configure’ or to help

 it to find libraries and programs with nonstandard names/locations.


 ./configure –with-php-config=/usr/local/webserver/php/bin/php-config –with-imagick=/usr/local/imagemagick


 编译过程中如果报错，则通过ln -s把ImageMagick的so及la文件链接到/usr/lib目录下，如果不报错，则不需要做链接，我原来是由于安装包不对，导致用tar zxvf解压的时候文件解出来的不全，例如一些.h文件都没有解压出来，所以编译当然会报错了，因此我才加了下面这些链接，后来编译还是报错。

 [root@CentOS_Test_Server lib]# ln -s /usr/local/imagemagick/lib/libMagickCore.so /usr/lib/libMagickCore.so

 [root@CentOS_Test_Server lib]# ln -s /usr/local/imagemagick/lib/libMagickCore.so.2 /usr/lib/libMagickCore.so.2

 [root@CentOS_Test_Server lib]# ln -s /usr/local/imagemagick/lib/libMagickCore.la /usr/lib/libMagickCore.la


 [root@CentOS_Test_Server lib]# ln -s /usr/local/imagemagick/lib/libMagick++.so /usr/lib/libMagick++.so

 [root@CentOS_Test_Server lib]# ln -s /usr/local/imagemagick/lib/libMagick++.so.2 /usr/lib/libMagick++.so.2

 [root@CentOS_Test_Server lib]# ln -s /usr/local/imagemagick/lib/libMagick++.la /usr/lib/libMagick++.la


 [root@CentOS_Test_Server lib]# ln -s /usr/local/imagemagick/lib/libMagickWand.so /usr/lib/libMagickWand.so

 [root@CentOS_Test_Server lib]# ln -s /usr/local/imagemagick/lib/libMagickWand.so.2 /usr/lib/libMagickWand.so.2

 [root@CentOS_Test_Server lib]# ln -s /usr/local/imagemagick/lib/libMagickWand.la /usr/lib/libMagickWand.la


 [root@CentOS_Test_Server lib]# ln -s /usr/local/imagemagick/lib/libMagickCore.so.2.0.0 /usr/lib/libMagickCore.so.2.0.0

 [root@CentOS_Test_Server lib]# ln -s /usr/local/imagemagick/lib/libMagick++.so.2.0.0 /usr/lib/libMagick++.so.2.0.0

 [root@CentOS_Test_Server lib]# ln -s /usr/local/imagemagick/lib/libMagickWand.so.2.0.0 /usr/lib/libMagickWand.so.2.0.0


 make

 小记编译imagick扩展遇到的郁闷的问题

 由于原来在目录/home/software下已经有文件imagick-2.2.2.tgz，这个文件是在vista下面下载后，通过Vmware共享给CentOS的，后来我通过wget又下载了一个，很巧的是也同样还是这个版本，文件名一模一样，因此新下载的imagick扩展自动命名为imagick-2.2.2.tgz.1，wget的下载日志“23:08:08 (35.1 KB/s) – `imagick-2.2.2.tgz.1′ saved [77212/77212]”，解压的时候解的仍然是vista下面下载的那个文件，因此导致解压的时候报错，解压出来的文件不全，因此导致编译报错，可能是与安装包的格式有关吧，参考 [http://linux.chinaunix.net/bbs/thread-1026205-1-1.html](http://linux.chinaunix.net/bbs/thread-1026205-1-1.html)，与这个有点类似吧，通过wget下载的安装包应该是不会出现这个问题的，然后rm imagick-2.2.2.tgz，mv imagick-2.2.2.tgz.1 imagick-2.2.2.tgz，tar zxvf imagick-2.2.2.tgz，解压后正常，说明是安装包的问题，然后后来的一切就都正常了。

 make install也就是把编译生成的so文件拷贝到php的扩展目录下

 [root@CentOS_Test_Server imagick-2.2.2]# make install

 /bin/sh /home/software/imagick-2.2.2/libtool –mode=install cp ./imagick.la /home/software/imagick-2.2.2/modules

 cp ./.libs/imagick.so /home/software/imagick-2.2.2/modules/imagick.so

 cp ./.libs/imagick.lai /home/software/imagick-2.2.2/modules/imagick.la

 PATH=”$PATH:/sbin” ldconfig -n /home/software/imagick-2.2.2/modules

 ———————————————————————-

 Libraries have been installed in:

 /home/software/imagick-2.2.2/modules


 If you ever happen to want to link against installed libraries

 in a given directory, LIBDIR, you must either use libtool, and

 specify the full pathname of the library, or use the `-LLIBDIR’

 flag during linking and do at least one of the following:

 – add LIBDIR to the `LD_LIBRARY_PATH’ environment variable

 during execution

 – add LIBDIR to the `LD_RUN_PATH’ environment variable

 during linking

 – use the `-Wl,–rpath -Wl,LIBDIR’ linker flag

 – have your system administrator add LIBDIR to `/etc/ld.so.conf’


 See any operating system documentation about shared libraries for

 more information, such as the ld(1) and ld.so(8) manual pages.

 ———————————————————————-

 Installing shared extensions:     /usr/local/webserver/php/lib/php/extensions/no-debug-non-zts-20060613/


 查看so文件是否已经生成，发现imagick.so文件已经在php的扩展目录下面了，证明已经安装成功

 [root@CentOS_Test_Server imagick-2.2.2]# ll /usr/local/webserver/php/lib/php/extensions/no-debug-non-zts-20060613/

 总计 3040

 -rwxr-xr-x 1 root root  172209 05-17 19:07 db4.so

 -rwxr-xr-x 1 root root  517843 04-28 20:55 eaccelerator.so

 -rwxr-xr-x 1 root root  748762 07-22 03:30 imagick.so

 -rwxr-xr-x 1 root root  185627 04-28 20:38 memcache.so

 -rwxr-xr-x 1 root root  118580 04-28 22:23 pdo_mysql.so

 -rwxr-xr-x 1 root root 1308827 07-08 19:10 xapian.so


 3.相关配置


* * *

 vi /usr/local/webserver/php/etc/php.ini，加入如下的一行

 extension = “imagick.so”

 然后执行php -m | grep imagick，发现已经出现imagick模块了，证明imagick已经生效了

 [root@CentOS_Test_Server imagick-2.2.2]# php -m | grep imagick

 imagick


 上面只是命令行生效了，为了让web服务器也生效，必须要重启php-fpm(我的环境是Nginx+fastcgi方式执行的php)

 [root@CentOS_Test_Server imagick-2.2.2]# /usr/local/webserver/php/sbin/php-fpm restart

 Shutting down php_fpm . done

 Starting php_fpm  done


 vi index.php,输入如下的内容

 phpinfo()

 ?>

 在浏览器执行index.php即可看到模块imagick已经生效

 4.测试


* * *

 PHP的imagick安装以后，提供4个PHP的图片处理类，分别是Imagick、ImagickDraw、ImagickPixel、ImagickPixelIterator，可以参考PHP的英文手册(一般指的是chm格式的手册，可以下载到本地查阅)或者在线手册( [http://cn.php.net/imagick](http://cn.php.net/imagick))

 可以用两种方式进行调用，一种方式是直接调用/usr/local/imagemagick/bin目录下的命令，第二种方式是调用PHP的imagick扩展提供的api


 测试一,通过php扩展进行调用

 vi /home/htdocs/www/imagick_test1.php，输入如下的代码

 header(‘Content-type: image/jpeg’);

 $image = new Imagick(‘old_s.jpg’);

 // If 0 is provided as a width or height parameter,

 // aspect ratio is maintained

 $image->thumbnailImage(100, 0);

 echo $image;

 ?>

 然后在浏览中执行，即可看到一张100px × 80px的图片，old_s是大S的意思，也就是我是用大S的照片做的测试，在这里就不截图了，其它的例子大家看手册基本就会用了


 另外可以在上述文件中的第一行加入判断模块imagick已经安装或加载的代码，extension_loaded(‘imagick’) or die(‘imagick not loaded’);


 测试二,通过命令行进行调用

 进入到图片所在目录

 cd /home/htdocs/www

 /usr/local/imagemagick/bin/convert old_s.jpg -resize 50% old_s.png

 原来的图片old_s.jpg是800*640，转换后生成的图片old_s.png是400*320，刚好减少一半

 每个命令的选项少的有几十个，多的则有上百个选项，非常复杂，功能非常的强大，参考 [http://www.imagemagick.org/script/command-line-tools.php](http://www.imagemagick.org/script/command-line-tools.php)

 另外，Discuz采用了ImageMagick及GD两种方式来处理图片，Discuz7.0版本有个文件Discuz_7.0.0_SC_GBK/upload/include/image.class.php,基中的方法Thumb_IM及Watermark_IM使用的就是ImageMagick，大家可以参考一下Discuz的代码。

 BTW:我写这篇博客是用的笔记本电脑的电池，因为有一段时间没有用笔记本电脑的电池了，以后每月尽量充放电一次，以保持电池的容量，电池大概用了二个小时零10分钟吧，还算可以


 参考：

 ImageMagick百度百科： [http://baike.baidu.com/view/1109708.htm](http://baike.baidu.com/view/1109708.htm)

ImageMagick中文站： [http://www.imagemagick.com.cn/](http://www.imagemagick.com.cn//)

ImageMagick英文站： [http://www.imagemagick.org/](http://www.imagemagick.org/)

imagemagick安装英文文档： [http://www.imagemagick.org/script/install-source.php](http://www.imagemagick.org/script/install-source.php)

ImageMagick的ftp下载地址： [ftp://ftp.imagemagick.org/pub/ImageMagick/](ftp://ftp.imagemagick.org/pub/ImageMagick///)

ImageMagick的api: [http://www.imagemagick.org/script/api.php](http://www.imagemagick.org/script/api.php)

ImageMagick的命令行参考文档： [http://www.imagemagick.org/script/command-line-tools.php](http://www.imagemagick.org/script/command-line-tools.php)

ImageMagick的PHP手册： [http://cn.php.net/imagick](http://cn.php.net/imagick)

ImageMagick的PHP扩展： [http://pecl.php.net/package/imagick](http://pecl.php.net/package/imagick)