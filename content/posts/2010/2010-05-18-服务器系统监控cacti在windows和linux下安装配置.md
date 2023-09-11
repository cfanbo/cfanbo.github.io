---
title: 服务器系统监控CACTI在windows和linux下安装配置
author: admin
type: post
date: 2010-05-18T01:58:50+00:00
url: /archives/3646
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - CACTI

---
**Windows下Cacti安装**
声明：本系列文档出自 [石头记](http://blog.sina.com.cn/5istone)，如若转载请注明出处，本人保留文档的所有权，并欢迎转载。

本系列文档的其他部分链接如下：
一、 [概述及Cacti的工作流程](http://blog.sina.com.cn/s/blog_4e424e2101000b5x.html)
二、 [Cacti安装](http://blog.sina.com.cn/s/blog_4e424e2101000b5y.html)
**(Linux)**
三、 [Cacti的使用](http://blog.sina.com.cn/s/blog_4e424e2101000b6o.html)
四、 [Cacti脚本及模板](http://blog.sina.com.cn/s/blog_4e424e2101000b77.html)
五、 [Cacti插件](http://blog.sina.com.cn/s/blog_4e424e2101000b7j.html)
六、 [Cacti高级应用–打造自己的Cacti模板](http://blog.sina.com.cn/s/blog_4e424e2101000bb7.html)
七、 [Cacti安装](http://blog.sina.com.cn/s/blog_4e424e2101000blp.html)

**（Windows)**
一、软件需求
1、  操作系统：Windows Server 2003企业版（或其他NT系统）。
2、  安装Apache，当然也可以使用IIS。
3、  安装MySQL，下载MySQL的Windows版本并安装到c:/mysql文件夹下。
4、  安装PHP，从www.php.net 下载PHP 5.X并安装到c:/php文件夹下。
5、  安装RRDTool，从www.cacti.net下载Cygwin版RRDTool并安装到c:/cacti文件夹下。
6、  安装Net-SNMP，下载Net-SNMP并安装到c:/net-snmp文件夹下。
7、  安装Cacti，将下载的Cacti 压缩文件解压到WEB目录下，并改名为cacti。
8、  安装Cactid，将下载的Cactid压缩文件解压到c:/cacti文件夹下。
9、  安装Cygwin，从Cygwin站点下载setup.exe文件，安装cygwin到c:/cygwin文件夹下。
10、安装ActivePerl，有些脚本是用perl语言写的，所以需要ActivePerl的支持。
二、Windows下Apache、PHP、MySQL的安装
1、安装Apache
点击安装文件apache\_2.2.4-win32-x86-no\_ssl.msi将apache安装在 c:\apache 目录下（随自己喜好）。
2、安装并配置MYSQL
在windows 下安装Mysql比较简单，和正常软件一样，下一步下一步就可以了，但最好把它的安装目录设置短一点，如：c:\mysql；安装成功后会有一个配置向 导。

点击Execute按钮完成MySQL的安装配置。
如果MySQL出现拒绝访问情况时，请在c:\和c:\mysql下查找my.cnf文件并将其删除后重启MySQL。
3、安装并配置PHP
1）、安装PHP
把php-5.2.1-Win32.zip解压到c:\php目录中，并设置环境变量如下图所示。
新建系统变量：变量名：MIBDIRS，变量值：c:\php\extras\mibs；
新建系统变量：变量名：PHPRC，变量值：c:\php；
编辑系统变量Path，增加“;c:\php;c:\php\ext;”
2）、配置PHP
将c:\php目录中的php.ini-dist重命名为php.ini，编辑php.ini文件，找到extension\_dir = “./” 改为 extension\_dir = “c:/php/ext”，找到
;extension=php_mysql.dll
;extension=php_snmp.dll
;extension=php_sockets.dll
将’;’去掉改为
extension=php_mysql.dll
extension=php_snmp.dll
extension=php_sockets.dll
cgi.force_redirect = 0

3）、配置Apache
在Apache的安装目录下找到并打开conf\httpd.conf文件，
找到 #LoadModule ssl\_module modules/mod\_ssl.so 这行，在此行后加入一行
LoadModule php5\_module c:/php/ php5apache2\_2.dll，其中c:/php/ php5apache2\_2.dll 为你php目录中php5apache2\_2.dll所在的位置
找到 AddType application/x-gzip .gz .tgz 这行，在此行后加入一行
AddType application/x-httpd-php .php
找到 DirectoryIndex index.html在后面加入 index.htm index.php
4）、测试PHP是否安装成功
此时PHP环境已经配置成功，在WEB根目录（如我的c:/Apache/htdocs）里建一个名为test.php的文件内容如下(测试时请将{换 成，将}换成>）
{?php
phpinfo();
?}
重新启动apache服务
用浏览器打开
[url=http://localhost/test.php]http://localhost/test.php
如果可以看到如下图所示的php配置输出信息就OK了。

**一、安装RRDTool**
下载RRDTool：
[http://www.cacti.net/downloads/rrdtool/win32/](http://www.cacti.net/downloads/rrdtool/win32/)
下载完成后解压缩，并解压出来的文件夹里的所有文件复制到c:/cacti下。
**二、安装Net-SNMP**
下载Net-SNMP：
[http://net-snmp.sourceforge.net/](http://net-snmp.sourceforge.net/)
下载最新版本的Win32安装文件，并将它安装到c:/net-snmp下。
**三、安装cactid**
下载Cactid：
[http://www.cacti.net/downloads/cactid/packages/Windows/](http://www.cacti.net/downloads/cactid/packages/Windows/)
解压Cactid，并将解压出的文件夹了的所有文件复制到c:/cacti下，并修改cactid.conf文件。
DB_Host        127.0.0.1 or hostname (请勿输入 localhost)
DB_Database     cacti
DB_User         cacti
DB_Password     cacti
DB_Port         3306
**四、安装Cygwin**
从 [Cygwin.com](http://cygwin.com) 站点下载setup.exe文件，安装cygwin到c:/cygwin文件夹下。
1）、运行刚下载的setup.exe
2）、选择以下安装包进行安装
**Base (include all items)
Libs
** **libart_lgpl
** **libfreetype26
** **libpng12
** **zlib
** **openssl
Utils
** **patch
Web
** **wget**
3）、添加c:\cygwin\bin到你的PATH系统变量中。
**五、安装ActivePerl**
下载最新版本的ActivePerl for windows并安装。
下载地址： [http://www.activestate.com/Products/Download/Download.plex?id=ActivePerl](http://www.activestate.com/Products/Download/Download.plex?id=ActivePerl)
安装完成后不要忘记将ActivePerl的执行文件目录添加到你的PATH系统变量中。
**六、安装并设定cacti**
下载最新版本cacti：
[http://www.cacti.net/downloads/](http://www.cacti.net/downloads/)
1）、解压下载的文件到WEB目录下
2）、打开命令提示符CMD，在MySQL里新建数据库cacti并将cacti.sql导入到数据库中。
C:\>mysql –uroot –p
Password:
mysql> create database cacti;
Query OK, 1 row affected (0.00 sec)

mysql> grant all on cacti.* to cacti@localhost identified by “cacti”;
Query OK, 1 row affected (0.00 sec)

mysql>flush privileges;
mysql>exit
C:\>
C:\>mysql –uroot –p cacti
Password:
3）、修改 cacti\_web\_root/cacti/include/config.php 配置文件。
$database_default = “cacti”;
$database_hostname = “localhost”;
$database_username = “cacti”;
$database_password = “cacti”;
$database_port = “3306”;
4）、打开浏览器输入
[http://your-server/cacti/install](http://your-server/cacti/install)
点击New Install，然后点下一步之后这里需要输入rrdtool、php、snmpwalk、snmpget、cactid的位置，请依照上面的安装路径进 行设置。
PHP Binary Path: ****c:/php/php.exe
RRDTool Binary Path: ****c:/cacti/rrdtool.exe

SNMPGET, SNMPWALK, SNMPBULKWALK, SNMPGETNEXT Paths:**
** c: net-snmp/usr/bin/snmpget.exe
c: net-snmp/usr/bin/snmpwalk.exe
c: net-snmp/usr/bin/snmpbulkwalk.exe
c: net-snmp/usr/bin/snmpgetnext.exe

Cacti Logfile Path: ****c:/apache/htdocs/cacti/log/cacti.log
Cactid Path: c:/cacti/cactid.exe

所有路径都是此安装程序的绝对路径
如果事后无法显示出图形请到Console → Settings → General
→ RRDTool Utility Version 将它改成RRDTool 1.2x
如果有图却没有文字的话，请到paths里的RRDTool Default Font Path – c:/windows/fonts/arial.ttf
注意：如果系统是Windows 2003 Server请将C:\WINDOWS\system32\cmd.exe及rrdTool跟netsnmp的*.exe加入IIS的使用者读取权限，此 举对系统有一定的危险性，如果无相关对策请更改作system。
5）、登录的帐号和密码都是admin，登录后需要立即修改密码。
6）.进入cacti后需确认更改以下位置：（如下图）
**Console>**
[**Settings**](http://192.168.100.76/cactiwww/settings.php)
**>General**
**Console>**
[**Settings**](http://192.168.100.76/cactiwww/settings.php)
**>Poller**

删除Localhost devices，添加一个新的Windows LocalHost，或者修改**Host Template**为**Windows 2000/XP。**
启动本机 SNMP
如果您也要侦测本机的snmp状态请用它
开始 → 控制面板 → 添加删除程序 → 添加删除Windows组件 → Management and Monitoring Tools（管理和监控工具）→ Simple Network Management Protocol（简单网络管理协议）→ 将它打勾后点击确定来启用它.
**7）、测试cacti是否安装正确**
打开命令提示符（CMD），输入c:/php/php.exe c:/cacti\_web\_root/cacti/poller.php
看是否输出下面类似信息：
C:\>c:/php/php.exe c:/cacti\_web\_root/cacti/poller.php
OK u:0.00 s:0.06 r:1.32
OK u:0.00 s:0.06 r:1.32
OK u:0.00 s:0.16 r:2.59
OK u:0.00 s:0.17 r:2.62
10/28/2005 04:57:12 PM – SYSTEM STATS: Time:4.7272 Method:cmd.php Processes:1 Threads:N/A Hosts:1 HostsPerProcess:2 DataSources:4 RRDsProcessed:2
在测试时如果错现snmp模块丢失错物可以试着将MIBDIRS设为：C:\net-snmp\usr\share\snmp\mibs

之后应该确定cacti.log文件在cacti\_web\_root/cacti/log/下出现，*.rrd文件在cacti\_web\_root /cacti/rra/下出现。
**8）、定时执行命令**
浏览c:\php\php.exeａ添加任务计划ａ任务计划ａ控制面板ａ点击开始 设置成每天执行，高级里面选每5分钟执行一次，持续24小时；再返回到属性的首页，运行(R)改成C:\php\php.exe C: /Apache/htdocs/cacti/poller.php
起始于改成C: /Apache/htdocs/cacti
当输入用于执行此任务计划的用户名和密码时，请注意你输入的用户有读和写以下目录的权限：
cacti\_web\_root/cacti/rra
cacti\_web\_root/log
并确认用户有读、写和执行以下目录文件的权限：
c:\php
c:\php\sapi

=====================================================
使用Cacti监控你的网络（二）- Cacti的安装
本系列文档的其他部分链接如下：
一、 [概述及Cacti的工作流程](http://blog.sina.com.cn/s/blog_4e424e2101000b5x.html)
二、 [Cacti的安装](http://blog.sina.com.cn/s/blog_4e424e2101000b5y.html)
三、 [Cacti的使用](http://blog.sina.com.cn/s/blog_4e424e2101000b6o.html)
四、 [Cacti脚本及模板](http://blog.sina.com.cn/s/blog_4e424e2101000b77.html)
五、 [Cacti插件](http://blog.sina.com.cn/s/blog_4e424e2101000b7j.html)
六、 [Cacti高级应用–打造自己的Cacti模板](http://blog.sina.com.cn/s/blog_4e424e2101000bb7.html)

**一、Cacti的安装**
1.安装环境：**RedHat AS 4**
2.安装Apache、MySQL、PHP
（1）.安装MySQL
下载地址：http://dev.mysql.com/downloads/mysql/5.0.html
//查看系统中是否已经安装了MySQL，如果是卸载所有以mysql开头的包。
\# rpm –qa | grep mysql
\# rpm –e mysql-*
//查找/etc/my.cnf（MySQL的选项配置文件），如果有请删除它，以免影响新安装版本的启动。
\# rm –f /etc/my.cnf
\# tar –zxvf mysql-standard-5.0.27-linux-i686-glibc23.tar.gz
\# cp –rf mysql-standard-5.0.27-linux-i686-glibc23 /usr/local/
//建立符号链接，如果以后有新版本的MySQL的话，你可以仅仅将源码解压到新的路径，然后重新做一个符号链接就可以了。这样非常方便，数据也更加安 全。
\# ln –s mysql-standard-5.0.27-linux-i686-glibc23 /usr/local/mysql
//添加用于启动MySQL的用户及用户组（如果以前安装过MySQl，用户及用户组可能已存在）。
\# useradd mysql
\# groupadd mysql
//初始化授权表
\# cd /usr/local/mysql
\# scripts/mysql\_install\_db
//修改MySQl目录的所有权
\# cd /usr/local
\# chgrp –R mysql mysql-standard-5.0.27-linux-i686-glibc23
\# chgrp –R mysql mysql
\# chown –R mysql mysql-standard-5.0.27-linux-i686-glibc23/data
\# chown –R mysql mysql/data
\# ln –s /usr/local/mysql/bin/* /usr/local/bin/
//启动Mysql
\# bin/safe_mysqld –user=mysql &
//配置系统启动时自动启动MySQl
\# cp support-files/mysql.server /etc/rc.d/init.d/mysqld
\# chkconfig –add mysqld
//修改MySQL的最大连接数
\# vi /etc/my.cnf
//添加以下行
[mysqld]
set-variable=max_connections=1000
set-variable=max\_user\_connections=500
set-variable=wait_timeout=200
//max_connections设置最大连接数为1000
//max\_user\_connections设置每用户最大连接数为500
//wait_timeout表示200秒后将关闭空闲（IDLE）的连接，但是对正在工作的连接不影响。
//保存退出，并重新启动MySQL
//重新启动MySQL后使用下面的命令查看修改是否成功
\# mysqladmin -uroot -p variables
Password:
//可以看到以下项说明修改成功
| max_connections                 | 1000
| max\_user\_connections            | 500
| wait_timeout                    | 200

（2）.安装Apache
下载地址：http://httpd.apache.org/
\# tar –zxvf
[httpd-2.2.4.tar.gz](http://apache.mirror.phpchina.com/httpd/httpd-2.2.4.tar.gz)
\# cd httpd-2.2.4
\# ./configure –prefix=/usr/local/apache –enable-so
//编译时加上加载模块参数–enable-so
\# make
\# make install
#vi /usr/local/apache/conf/httpd.conf
//修改Apache配置文件，添加ServerName
[www.yourdomain.com](http://www.yourdomain.com/)
（或ServerName 本机ip）
\# vi /etc/rc.d/rc.local
//在rc.local上加入一行/usr/local/apache/bin/apachectl –k start,系统启动时启动Apache服务。
（3）.安装PHP
先安装zlib,freetype,libpng,jpeg以便于让PHP支持GD库（Cacti的WeatherMap插件必须要较新GD库的支持）
库文件下载地址：http://oss.oetiker.ch/rrdtool/pub/libs/
1）.安装zlib
tar zlib-1.2.3.tar.gz
cd zlib-1.2.3
./configure –prefix=/usr/local/zlib
make
make install
2）.安装libpng
tar zxvf libpng-1.2.16.tar.tar
cd libpng-1.2.16
cd scripts/
mv makefile.linux ../makefile
cd ..
make
make install
注意，这里的makefile不是用./configure生成，而是直接从scripts/里拷一个
3）.安装freetype
tar zxvf freetype-2.3.4 .tar.gz
cd freetype-2.3.4
./configure –prefix=/usr/local/freetype
make
make install
4）.安装Jpeg
tar -zxf jpegsrc-1.v6b.tar.gz
cd jpeg-6b/
mkdir /usr/local/libjpeg
mkdir /usr/local/libjpeg/include
mkdir /usr/local/libjpeg/bin
mkdir /usr/local/libjpeg/lib
mkdir /usr/local/libjpeg/man
mkdir /usr/local/libjpeg/man/man1
//可以用mkdir -p /usr/local/libjpeg/man/man1 一步创建多层目录
./configure –prefix=/usr/local/libjpeg –enable-shared –enable-static
make && make install
注意，这里configure一定要带–enable-shared参数，不然，不会生成共享库

5）.安装Fontconfig
tar -zxvf fontconfig-2.4.2.tar.gz
cd fontconfig-2.4.2
./configure –with-freetype-config=/usr/local/freetype
make
make install
6）.安装GD
tar -zxvf gd-2.0.34.tar.gz
cd gd-2.0.34
./configure –prefix=/usr/local/libgd –with-png –with-freetype=/usr/local/freetype –with-jpeg=/usr/local/libjpeg
make
make install
编译时显示以下信息：
** Configuration summary for gd 2.0.34:
Support for PNG library:          yes
Support for JPEG library:         yes
Support for Freetype 2.x library: yes
Support for Fontconfig library:   yes
Support for Xpm library:          no
Support for pthreads:             yes
7）.编辑/etc/ld.so.conf，添加以下几行到此文件中。
/usr/local/zlib/lib
/usr/local/freetype/lib
/usr/local/libjpeg/lib
/usr/local/libgd/lib
并执行ldconfig命令，使用动态装入器装载找到共享库

8）.安装libxml，RedHat AS 4默认安装libxml包，但版本太低，PHP5需要更高版本的libxml包。
\# tar –zxvf libxml2-2.6.25.tar.gz
\# cd libxml2-2.6.25
\# ./configure
\# make
\# make install

9）.安装PHP
PHP下载地址：http://www.php.net/downloads.php#v5
tar -zxvf  php-5.2.3.tar.gz
cd php-5.2.3
\# ./configure –prefix=/usr/local/php –with-apxs2=/usr/local/apache/bin/apxs –with-mysql=/usr/local/mysql –with-gd=/usr/local/libgd –enable-gd-native-ttf –with-ttf –enable-gd-jis-conv –with-freetype-dir=/usr/local/freetype –with-jpeg-dir=/usr/local/libjpeg –with-png-dir=/usr –with-zlib-dir=/usr/local/zlib –enable-xml –enable-mbstring –enable-sockets
\# make
\# make install
\# cp php.ini-recommended /usr/local/php/lib/php.ini
\# ln –s /usr/local/php/bin/* /usr/local/bin/
\# vi /usr/local/apache/conf/httpd.conf
查找AddType application/x-compress .Z
AddType application/x-gzip .gz .tgz
在其下加入 AddType application/x-tar .tgz
AddType application/x-httpd-php .php
AddType image/x-icon .ico
修改DirectoryIndex 行，添加index.php
修改为DirectoryIndex index.php index.html index.html.var
\# vi /usr/local/apache/htdocs/test.php
添加以下行：
//php标记（用代替[）
[?php
Phpinfo();
?]
wq保存退出。
\# /usr/local/apache/bin/apachectl –k stop
#/usr/local/apache/bin/apachectl –k start
在浏览器中输入：
[http://www.yourdomain.com](http://www.yourdomain.com/)
/test.php进行测试。

对php编译选项的解释：
–prefix=/usr/local/php   //指定PHP的安装目录
–with-apxs2=/usr/local/apache2/bin/apxs      //支持Apache模块
–with-mysql=/usr/local/mysql    //支持MySQl
–with-gd=/usr/local/libgd     //支持GD库
–enable-gd-native-ttf     //激活对本地 TrueType 字符串函数的支持
–with-ttf     //激活对 FreeType 1.x 的支持
–with-freetype-dir=/usr/local/freetype    //激活对 FreeType 2.x 的支持
–with-jpeg-dir=/usr/local/libjpeg //激活对 jpeg-6b 的支持
–with-png-dir=/usr   //激活对 png 的支持
–with-zlib-dir=/usr/local/zlib //激活对zlib 的支持
–enable-mbstring    //激活mbstring模块
–enable-gd-jis-conv //使JIS-mapped可用，支持日文字体
–with-mail   //支持Mail函数
–enable-xml     //支持XML
–enable-sockets      //支持套接字

1.安装RRDTool
由于rrdtool-1.2.23需要一些库文件支持，故需先安装配置支持的环境，然后编译安装。直接运行以下bash脚本就可以完成安装：
注意：将cgilib-0.5.tar.gz、zlib-1.2.3.tar.gz、libpng-1.2.18.tar.gz、freetype- 2.3.5.tar.gz、libart_lgpl-2.3.17.tar.gz、rrdtool-1.2.23.tar.gz放到/root /rrdtool-1.2.23目录下，将脚本保存为/root/rrdtool-1.2.23/rrdtoolinstall.sh，并给执行权限 chmod u+x /root/rrdtool-1.2.23/rrdtoolinstall.sh。
以下链接是我重新打好的一个rrdtool-1.2.23的安装包，里面包括了所有用到的库文件和安装脚本，下载解压后执行脚本 rrdinstall.sh即可以完成RRDTool的安装。
点击下载
[rrdtool-1.2.23.tar.gz](http://61.156.20.41/autodownload/rrdtool-1.2.23.tar.gz)
如果以上脚本安装失败，可以试试以下安装包：
http://61.156.20.41/autodownload/rrdtool-1.2.11.tar.gz
#!/bin/sh
BUILD_DIR=\`pwd\`
INSTALL_DIR=/usr/local/rrdtool
cd $BUILD_DIR
tar zxf cgilib-0.5.tar.gz
cd cgilib-0.5
make CC=gcc CFLAGS=”-O3 -fPIC -I.”
mkdir -p $BUILD_DIR/lb/include
cp *.h $BUILD_DIR/lb/include
mkdir -p $BUILD_DIR/lb/lib
cp libcgi* $BUILD_DIR/lb/lib
cd $BUILD_DIR
tar  zxf zlib-1.2.3.tar.gz
cd zlib-1.2.3
env CFLAGS=”-O3 -fPIC” ./configure –prefix=$BUILD_DIR/lb
make
make install
cd $BUILD_DIR
tar zxvf libpng-1.2.18.tar.gz
cd libpng-1.2.18
env CPPFLAGS=”-I$BUILD\_DIR/lb/include” LDFLAGS=”-L$BUILD\_DIR/lb/lib” CFLAGS=”-O3 -fPIC” \
./configure –disable-shared –prefix=$BUILD_DIR/lb
make
make install
cd $BUILD_DIR
tar zxvf freetype-2.3.5.tar.gz
cd freetype-2.2.5
env CPPFLAGS=”-I$BUILD\_DIR/lb/include” LDFLAGS=”-L$BUILD\_DIR/lb/lib” CFLAGS=”-O3 -fPIC” \
./configure –disable-shared –prefix=$BUILD_DIR/lb
make
make install
cd $BUILD_DIR
tar zxvf libart_lgpl-2.3.17.tar.gz
cd libart_lgpl-2.3.17
env CFLAGS=”-O3 -fPIC” ./configure –disable-shared –prefix=$BUILD_DIR/lb
make
make install
IR=-I$BUILD_DIR/lb/include
CPPFLAGS=”$IR $IR/libart-2.0 $IR/freetype2 $IR/libpng”
LDFLAGS=”-L$BUILD_DIR/lb/lib”
CFLAGS=-O3
export CPPFLAGS LDFLAGS CFLAGS
cd $BUILD_DIR
tar zxf rrdtool-1.2.23.tar.gz
cd rrdtool-1.2.23
./configure –prefix=$INSTALL_DIR –disable-python –disable-tcl && make && make install
//完成后建立符号连接
ln –s /usr/local/rrdtool/bin/* /usr/local/bin/
//执行rrdtool看是否安装正确
2.安装net-snmp
RedHat默认安装了SNMP服务，但好象没有snmpwalk,snmpget这两个命令，所以需要编译安装NET-SNMP。
NET-SNMP官方网站：
[http://www.net-snmp.org/](http://www.net-snmp.org/)
\# tar zxvf net-snmp-5.2.4.tar.gz
#cd net-snmp-5.2.4
#./configure –prefix=/usr/local/net-snmp  –enable-developer
#make
#make install
\# ln –s /usr/local/net-snmp/bin/* /usr/local/bin/
#cp EXAMPLE.conf  /usr/local/net-snmp/share/snmp/snmpd.conf
//修改snmpd.conf（修改COMMUNITY、允许抓取snmp数据的主机、抓取数据范围等）。
\# /usr/local/net-snmp/sbin/snmpd     //启动SNMP服务
\# vi /etc/rc.d/rc.local
//在rc.local上加入一行/usr/local/net-snmp/sbin/snmpd,系统启动时启动SNMP服务。

3.安装Cacti
Cacti官方网站：
[www.cacti.net/](http://www.cacti.net/)
\# tar –zxvf cacti-0.8.6j.tar.gz
\# mv –r cacti-0.8.6j /usr/loca/apache/htdocs/cacti
\# vi /usr/local/apache/htdocs/cacti/include/config.php
$database_type = “mysql”;
$database_default = “cacti”;
$database_hostname = “localhost”;
$database_username = “cacti”;
$database_password = “cacti”;

//添加cacti用户
\# useradd cacti
//将rra目录的所有权给cacti用户
\# chown –R cacti /usr/loca/apache/htdocs/cacti/rra
//修改cacti目录所属组
\# chgrp –R cacti /usr/loca/apache/htdocs/cacti
//为cacti用户添加cron任务
\# su – cacti
\# crontab –e
\*/5 \* \* \* * /usr/local/bin/php /usr/local/apache/htdocs/cacti/poller.php > /dev/null 2>&1
注意：首次执行poller.php时请使用cacti用户，否则生成的rrd文件cacti将没有写入权限。

4.安装Cactid
CACTID 的安装需要以下支持：
o    net-snmp-devel （需要编译安装net-snmp时添加–enable-developer选项）
o    mysql
o    mysql-devel     （mysql源文件编译安装后默认支持）
o    openssl-devel  （Redhat默认安装）
\# tar -zxvf cacti-cactid-0.8.6i.tar.gz
\# cd cacti-cactid-0.8.6i
\# ./configure –with-mysql=/usr/local/mysql –with-snmp=/usr/local/net-snmp
\# make
//这时你将在此目录下看到多出了cactid、cactid.conf两个文件
\# mkdir /usr/local/cactid
\# cp cactid cactid.conf /usr/local/cactid
\# vi  /usr/local/cactid/cactid.conf       //修改cactid配置文件
DB_Host         127.0.0.1
DB_Database     cacti
DB_User         cacti
DB_Pass         cacti

5.数据库配置
#mysql –uroot –p
Password:
mysql> create database cacti;
Query OK, 1 row affected (0.00 sec)

mysql> grant all on cacti.* to cacti@localhost identified by “cacti”;
Query OK, 1 row affected (0.00 sec)

mysql>exit
\# cd /usr/local/apache/htdocs/cacti
\# mysql –uroot –p cacti
Password:

6.完成cacti的安装
1）.在浏览器中输入：
[http://www.yourdomain.com/cacti/](http://www.yourdomain.com/cacti/)
默认用户名：admin 密码：admin
2）.更改密码
3）.设置cacti用到的命令路径
**snmpwalk Binary Path** /usr/local/ bin/snmpwalk
**snmpget Binary Path** /usr/local/ bin/snmpget
**RRDTool Binary Path** /usr/local/ bin/rrdtool
**PHP Binary Path** /usr/local/bin/php
**Cacti Log File Path** /usr/local/apache/htdocs/cacti/log/cacti.log
**Cactid Poller File Path** /usr/local/cactid/cactid
4）.进入cacti后需确认更改以下位置：（如下图）
**Console>**
[**Settings**](http://192.168.100.76/cactiwww/settings.php)
**>General**

**Console>**
[**Settings**](http://192.168.100.76/cactiwww/settings.php)
**>Poller**