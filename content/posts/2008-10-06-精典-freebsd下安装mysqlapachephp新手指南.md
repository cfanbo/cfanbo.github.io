---
title: '[精典] FreeBSD下安装MySQL+Apache+PHP新手指南'
author: admin
type: post
date: 2008-10-06T12:09:51+00:00
excerpt: |
 安装FreeBSD就不讲了,只要稍微定制一下就可以了,过程我就不说了,我用的FreeBSD版本是5.2.1,应该是现在比较新的版本,以后就不知道了 :) .


 一. 安装MySQL

 我使用的的Mysql是4.0.20,源代码版,你也可以使用RPM包或者二进制版,安装方法可能不一样,请参考其它文章.
 先下载Mysql2.0.20的源代码版,地址: http://dev.mysql.com/downloads/mysql/4.0.html
 把它下到/usr/local/src目录下,如果没有该目录,就自己建一个.下载回来的包名字叫 mysql-4.0.20.tar.gz,然后我们把它解压出来:

 # tar -zxvf mysql-4.0.20.tar.gz

 解压后生成mysql-4.0.20目录,我们进入该目录:
url: /archives/438
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - mysql
 - php

---
作者: heiyeluren
QQ群: 5415735 (Linux/BSD安装维护群)
日期: 2004/8/18
— 特别感谢QQ群好友阿南,本文在他的耐心指导下才产生 —

看到朋友们在Unix/Linux上装mysql有点麻烦,我也好不容易装完了,所以就来讲件,也许能帮帮大家的忙. 我使用的操作系统是FreeBSD5.2.1,如果别的操作系统安装方法也许不一样,请酌情处理.
安装FreeBSD就不讲了,只要稍微定制一下就可以了,过程我就不说了,我用的FreeBSD版本是5.2.1,应该是现在比较新的版本,以后就不知道了 🙂 .

**一. 安装MySQL**

我使用的的Mysql是4.0.20,源代码版,你也可以使用RPM包或者二进制版,安装方法可能不一样,请参考其它文章.
先下载Mysql2.0.20的源代码版,地址: http://dev.mysql.com/downloads/mysql/4.0.html
把它下到/usr/local/src目录下,如果没有该目录,就自己建一个.下载回来的包名字叫 mysql-4.0.20.tar.gz,然后我们把它解压出来:

**\# tar -zxvf mysql-4.0.20.tar.gz**

解压后生成mysql-4.0.20目录,我们进入该目录:

**\# cd mysql-4.0.20**

进入后就开始配置mysql了,配置过程中我们要给mysql设置一个安装目录,我们设置在 /usr/local/mysql 下,以为把文件放到一个地方比较容易管理,如果你还想获得更多的配置信息,使用 ./configure –help:

**\# ./configure –prefix=/usr/local/mysql**

上面的命令指定mysql的安装路径,然后等几秒钟,配置完成后就编译源代码:

**\# make**

这个编译的过程比较长,如果机器比较慢的话,可能要近二十分种 ( PS:我的机器是很普通的机器,所有用了差不多15,6分种 🙁 ) .
编译完成后就安装:

**\# make install**

等上几秒钟,安装完成.下面就到了最关键的部分了,为什么老安装不成功,(PS:至少我是安装了N次,N > 10 ,呵呵),问题关键就在这里,访问mysql要一个专门的用户,而且必须给相应的访问权限,这里我们就设置root和mysql有权限访问.
我们先建立一个mysql和mysql用户来访问mysql:

**\# pw groupadd mysql** #建立mysql组
**\# pw useradd mysql -g mysql** #建立mysql用户并且加入到mysql组中，_**最好重新使用chpass把mysql用户的登陆shell去掉，比如改成/sbin/nologin，为了防止未来授权用户访问。**_

建立用户后我们就初始化表 (注意:必须先执行本步骤后才能进行以下步骤)

**\# ./scripts/mysql\_install\_db –user=mysql** #初试化表并且规定用mysql用户来访问

初始化表以后就开始给mysql和root用户设定访问权限, 我们先到安装mysql的目录:

**\# cd /usr/local/mysql**

然后设置权限

**\# chown -R root** . #设定root能访问/usr/local/mysql
**\# chown -R mysql var** #设定mysql用户能访问/usr/local/mysql/var ,里面存的是mysql的数据库文件
**\# chown -R mysql var/.** #设定mysql用户能访问/usr/local/mysql/var下的所有文件
**\# chown -R mysql var/mysql/.** #设定mysql用户能访问/usr/local/mysql/var/mysql下的所有文件
**\# chgrp -R mysql .** #设定mysql组能够访问/usr/local/mysql

// 补充，因为以上权限设置有点问题，于是参考其他资料，设置如下：

**chown -R root /usr/local/mysql**
**chgrp -R mysql /usr/local/mysql
chown -R root /usr/local/mysql/bin
chgrp -R mysql /usr/local/mysql/bin
chown -R root /usr/local/mysql/var
chgrp -R mysql /usr/local/mysql/var
chmod 777 /usr/local/mysql/var
chown -R root /usr/local/mysql/var/mysql
chgrp -R mysql /usr/local/mysql/var/mysql
chmod 777 /usr/local/mysql/var/mysql
chown -R root /usr/local/mysql/var/mysql/*
chgrp -R mysql /usr/local/mysql/var/mysql/*
chmod 777 /usr/local/mysql/var/mysql/*
chmod 777 /usr/local/mysql/lib/mysql/libmysqlclient.a**

设置完成后,基本上就装好了,好了,我们运行一下我们的mysql:

**\# /usr/local/mysql/bin/mysqld_safe –user=mysql &**

如果没有问题的话,应该会出现类似这样的提示:

[1] 42264
**\# Starting mysqld daemon with databases from /usr/local/mysql/var**

这就证明你安装成功了,如果出现:

[1] 42264
\# Starting mysqld daemon with databases from /usr/local/mysql/var
040818 10:53:45 mysqld ended

则证明你的mysql运行不来,请查看错误日志: /usr/local/mysql/var/*.err 然后确定安装是否成功,如果没有成功,请检查上面的步骤是否正确.
安装完成后,能够通过 /usr/local/mysql/bin/mysql 来连接mysql进行管理,如果你装了apache并且能够解析php的话,也能使用phpMyadmin来管理你的mysql,记得装完后使用mysql或者mysqladmin来修改root的密码,这里我们就不说了,请参考相关的文章.

控制mysql就通过 /usr/local/mysql/libexec/mysqld 来控制启动或者停止mysql:

**\# /usr/local/mysql/libexec/mysqld start** #启动mysql
**\# /usr/local/mysql/libexec/mysqld stop** #停止mysql
**\# /usr/local/mysql/libexec/mysqld restart** #重启mysql

为了每次系统重启后都能运行mysql,编辑 /etc/rc.conf 配置文件,在文件最后面添加一新行:**mysql_enable = “YES”** ,也可以写一个脚本放到 /usr/local/etc/rc.d目录下,用来运行mysql,我们写一个脚本mysql_start.sh

**#! /bin/sh
/usr/local/mysql/bin/mysqld_safe&**

然后保存到/usr/local/etc/rc.d目录下,那么以后reboot系统后都能启动mysql了.

**二. 安装Apache**

安装Apache要简单点,我这里安装的Apache版本是 httpd-2.0.50,去下载压缩包: http://httpd.apache.org/download.cgi.
下载回来的包叫做 httpd-2.0.50.tar.gz 我们放在 /usr/local/src目录下.
首先进入目录后解压缩:

**\# cd /usr/local/src
\# tar -zxvf httpd-2.0.50.tar.gz**

然后就会得到 httpd-2.0.50目录,我们进入目录

**\# cd httpd-2.0.50**

首先配置:

**\# ./configure \
? –prefix=/usr/local/apache \ #我们要把Apache安装在那个目录,我们这里装在 /usr/local/apache下
? –enable-shared=max \
? –enable-module=rewrite \
? –enable-module=so**

执行上面的命令,如果没有错误信息,证明配置成功,然后进行编译:

**\# make**

一两分钟就编译完了,然后进行安装:

**\# make install**

安装完成后,Apache就存放在 /usr/local/apache目录下了, bin是执行文件的目录,conf是配置文件目录,htdocs是网页的主目录,logs是日志目录.
Apache通过 bin/apachectl或者bin/httpd来控制启动或者停止.

**\# /usr/local/apache/bin/httpd -k start** #启动apache
**\# /usr/local/apache/bin/httpd -k stop** #停止apache
**\# /usr/local/apache/bin/httpd -k restart** #重启apache

然后你可以通过 http://localhost 来测试apache是否安装成功,如果出现apache的页面则安装成功,否则请检查上面的步骤.

**三. 安装PHP**

我们使用的PHP版本是4.3.8,先去下载: http://www.php.net/downloads.php, 下回来的包叫做 php-4.3.8.tar.gz, 放到/usr/local/src目录下.
首先进入该目录后解压缩:

**\# cd /usr/local/src
\# tar -zxvf php-4.3.8.tar.gz**

解压后进入目录:

**\# cd php-4.3.8**

进行配置,这一步比较关键,一定要设置好,特别是要考虑到你要支持什么,比如GD库,xml,mysql等等,如果想知道详细的配置,执行 ./configure –help来获得:

**\# ./configure \
? –with-apxs2=/usr/local/apache/bin/apxs \
? –disable-debug \ #关闭php内部调试
? –enable-safe-mode \ #打开php的安全模式
? –enable-trans-sid \
? –with-xml \ #支持xml
? –with-mysql \ #支持mysql
? –enable-short-tags \ #支持PHP的短标记
? –with-gd \ #支持GD库
? –with-zlib \ #支持zlib
? –with-jpeg \
? –with-png \
? –enable-memory-limit \
? –disable-posix \
? –with-config-file-path=/usr/local/lib**

如果上面的配置没有错误的话,那么应该最后会显示感谢使用PHP等字样,那么证明配置成功,如果上面的配置选项不支持的话,会提示错误.
比如你没有安装mysql,那么–with-mysql就无法使用,所以一定要注意对应选项系统是否能够支持,如果出现错误,那么就先安装对应的程序,或者去掉相关选项.
配置之后就进行编译:

**\# make**

编译成功后出现”Build complete.”字样,那么就可以进行安装了:

**\# make install**

安装完成后把/usr/local/src/php-4.3.8/php.ini-dist复制到/usr/local/lib/，并重命名为php.ini

**\# cp /usr/local/src/php-4.3.8/php.ini-dist /usr/local/lib/php.ini**

基本到这里PHP就安装成功了,如果中间出现错误,除了在配置的时候没有选对选项之后一般都不出现错误.

**四. 整合Apache+PHP**

为了让Apache能够直接解析php,我们还要进行一些配置.
首先进入apache的配置文件目录:

**\# cd /usr/local/apache/conf**

然后用vi打开配置文件httpd.conf:

**\# vi httpd.conf**

在httpd.conf文件中，添加

**AddType application/x-httpd-php .php
AddType application/x-httpd-php-source .phps**

应该将以上两句添加在其他AddType之后。

确保文件中有以下一句话，没有就自己添加在所有LoadModule之后。

**LoadModule php4_module modules/libphp4.so**

好了,在vi中使用”:wq”保存httpd.conf文件，退出vi。启动apache server:

**\# /usr/local/apache/bin/httpd start**

现在apache就能够运行php了,写个文件测试一下,在/usr/local/apache/htdocs目录下，新建一个phpinfo.php文件，
文件中只有一行代码:

保存此文件, 在你的浏览器中输入http://localhost/phpinfo.php，你应该看到PHP的系统信息。
如果出现错误,比如提示你下灾phpinfo.php,那么apache就是还无法解析php文件,那么请仔细检查以上的操作是否正确.

好,到这里,基本上Mysql+Apache+PHP安装完成,那么就能做Web服务器了,比如传个论坛,
同时提醒可以传个phpMyadmin去管理你的mysql,现在最新版本是phpMyadmin2.6-beta

WriteTime 2004-8-18 19:30