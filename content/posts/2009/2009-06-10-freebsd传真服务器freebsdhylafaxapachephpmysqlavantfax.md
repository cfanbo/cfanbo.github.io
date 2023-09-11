---
title: FreeBSD传真服务器(FreeBSD+HylaFax+Apache+php+Mysql+AvantFax)
author: admin
type: post
date: 2009-06-10T18:21:34+00:00
excerpt: |
 FreeBSD安装选择Minimal+Ports
 域名：fax.test.org IP:192.168.1.203 新建用户:vincent 属于wheel组

 Handbook
 http://cnsnap.cn.freebsd.org/doc ... ndbook/install.html

 开启FTP服务

 编辑/etc/inetd.conf文件去掉ftp前的注释'#'。

 #vi /etc/inetd.conf
 ftp stream tcp nowait root /usr/libexec/ftpd ftpd -l

 启动inetd程序
 #/etc/rc.d/inetd start

 添加vincent用户，用于FTP登录上传文件
 #pw useradd vincent -s /bin/csh -d /home/vincent -m -g wheel -h 0
url: /archives/1673
IM_data:
 - 'a:2:{s:55:"http://bbs3.chinaunix.net/images/smilies/icon_smile.gif";s:70:"http://blog.haohtml.com/wp-content/uploads/2009/06/924d_icon_smile.gif";s:52:"http://bbs3.chinaunix.net/images/attachicons/rar.gif";s:63:"http://blog.haohtml.com/wp-content/uploads/2009/06/25bb_rar.gif";}'
IM_contentdowned:
 - 1
categories:
 - 服务器

---
[http://bbs3.chinaunix.net/thread-1456005-1-1.html](http://bbs3.chinaunix.net/thread-1456005-1-1.html)

FreeBSD安装选择Minimal+Ports
域名：fax.test.org IP:192.168.1.203 新建用户:vincent 属于wheel组

Handbook
[http://cnsnap.cn.freebsd.org/doc … ndbook/install.html](http://cnsnap.cn.freebsd.org/doc/zh_CN.GB2312/books/handbook/install.html)

开启FTP服务

编辑/etc/inetd.conf文件去掉ftp前的注释’#’。

#vi /etc/inetd.conf
ftp     stream  tcp     nowait  root    /usr/libexec/ftpd       ftpd -l

启动inetd程序
#/etc/rc.d/inetd start

添加vincent用户，用于FTP登录上传文件
#pw useradd vincent -s /bin/csh -d /home/vincent -m -g wheel -h 0

—————————————————————————-

使用wget加快ports软件下载

安装wget程序，加快软件包下载速度。
#cd /usr/ports/net/wget
#make install clean

编辑/etc/make.conf
#vi /etc/make.conf

FETCH_CMD=wget -c -t 1
DISABLE_SIZE=yes

MASTER\_SITE\_OVERRIDE= \
[ftp://ftp.tw.freebsd.org/pub/FreeBSD/ports/distfiles/](ftp://ftp.tw.freebsd.org/pub/FreeBSD/ports/distfiles/) \
[ftp://ftp.freebsdchina.org/pub/FreeBSD/ports/distfiles/](ftp://ftp.freebsdchina.org/pub/FreeBSD/ports/distfiles/)

设置使用ftp.tw.freebsd.org为主下载站点，加快Package软件下载，编辑用户目录下的.cshrc文件加入

#vi .cshrc     //编辑完后记得重新登录
setenv PACKAGEROOT      [ftp://ftp.tw.freebsd.org](ftp://ftp.tw.freebsd.org/)

——————————————————————————————–

HylaFAX    ( WebSite [http://www.hylafax.org/](http://www.hylafax.org/) )

HylaFAX是一个基于C/S 架构,企业级的收发传真系统，高效稳固。局域网中只要有一台连接Modem的HylaFAX服务器，就能为局域网所有用户提供传真服务。

软件安装

Package方法安装
#pkg_add -r hylafax

或者

Ports方法安装
#cd /usr/ports/comms/hylafax
#make install clean

软件设置

#faxsetup

Should an entry be added for the FaxMaster to /etc/aliases [yes]?
应该在/etc/aliases中增加一个条FaxMaster记录[yes]? yes

Users to receive fax related mail [root]?
输入接收传真相关信息的Email用户[root]? vincent

Are these ok [yes]?
确认以上信息是否正确[yes]? yes

Country code [1]?
国家代码[1]? 0086

Area code []?
区号[]? 0750

Long distance dialing prefix [1]?
长途拨号前缀 [1]? 0

International dialing prefix [011]?
国际拨号前缀 [001]? 0750

Dial string rules file (relative to /var/spool/hylafax)[“etc/dialrules”]?
拨号规则文件( /var/spool/hylafax )[“etc/dialrule”]? 按enter默认

Tracing during normal server operation [1]?
追踪正常服务程序[1]? 1

Default tracing during send and receive session [0xfffffffff]?
默认追查在发送和接收 session [0xfffffffff]? 按enter默认

Continuation cover page (relative to /var/spool/hylafax) []?
传真封面页所在目录 ( /var/spool/hylafax )[]? 按enter默认

Timeout when converting PostScript documents (secs) [180]?
转换PostScript文件逾时时间[180]? 180

Maximum number of concurrent jobs to a destination[1]?
一个目的地最大数量的并行工作[1]? 1

Define a group of modems []
定义一组调制解调器[] 按enter默认

Time of day restrictions for outbound jobs [“Any”]?
一天中限制传真外发时间[“Any”]? 按enter默认

Pathname of destination controls file (relative to /var/spool/hylafax) []?
控制文件的路径( /var/spool/hylafax )[]? 按enter默认

Timeout before purging a stale UUCP lock file (secs) [30]
超时前清除旧的UUCP锁定文件[30]？30

Max number of pages to permit in an outbound job [0xffffffff]?
允许在出站的最大页数[0xffffffff]? 按enter默认

Syslog facility name for ServerTracing messages [daemon]?
系统日志跟踪记录程序[daemon]? 按enter默认

Are these ok [yes]?
确认以上信息是否正确[yes]? yes

Should I restart the HylaFAX process [yes]?
应该重新启动HylaFAX进程[yes]? yes

You do not appear to have any modem configured for use. Modems are
configured for use with HylaFax with the faxaddmodem command.
Do you want to run faxaddmomdem to configure a modme [yes]?
您似乎没有任何调制解调器配置为使用。调制解调器配置为使用HylaFax与faxaddmodem命令。
你想运行faxaddmomdem配置modme[yes]? yes

Serial port that modem is connected to []?
调制解调器连接到那个串行端口[]? ttyd0    //我的是com1,所以是ttyd0；请根据实际配置。

country code[1]
国家代码[1]? 0086

Area code [415]?
区号[]? 0750

Phone number of fax modem [+1,9999.5555.1212]?
传真的电话号码[+1,9999.5555.1212]? 8607501234567

Local Identifications string (for TS/CIG) [“NothingEtup”]?
本地传真机标识(for TS/CIG) [“NothingEtup”]? FreeBSD.org

Long distance dialing prefix [1]?
长途拨号前缀 [1]? 0

International dialing prefix [011]?
国际拨号前缀 [001]? 0750

Dial string rules file (relative to /var/spool/hylafax) [etc/dialrules]?
拨号规则文件( /var/spool/hylafax )[“etc/dialrule”]? 按enter默认

Tracing during normal server operation [1]?
追踪正常服务程序[1]? 1

Tracing during send and receive sessions [11]?
追踪发送和接收 session [11]? 按enter默认

Protection mode for received facsimile [0600]?
收到传真的文件权限[0600]？ 0777

Protection mode for session logs [0600]?
记录文件的档案权限[0600]? 0777

Protection mode for ttyd0 [0600]?
端口的访问权限[0600]? 0777

Rings to wait before answering [1]?
响铃几声后，开始接受传真[1]? 2

Modem speaker volume [off]?
Modem的喇叭音量[off]? on

Command line arguments to getty program [“-h %l dx_%s”]?
接收传真的命令行参数[“-h %l dx_%s”]? 按enter默认

Pathname of TSI access control list file (relative to /var/spool/hylafax)[“”]?
访问控制列表的TSI文件路径( /var/spool/hylafax )[“”]? 按enter默认

Pathname of Caller-ID access control list file (relative to /var/spool/hylafax)[“”]?
来电Caller-ID访问控制列表文件路径( /var/spool/hylafax )[“”]? 按enter默认

Tag line font file (relative to /var/spool/hylafax) [etc/lutRS18.pcf]?
标记行字体文件( /var/spool/hylafax ) [etc/lutRS18.pcf]? 按enter默认

Tag line form string [“From %%1|%c|Page %%P of %%T”]?
标记行字符串形式[“From %%1|%c|Page %%P of %%T”]? 按enter默认

Time before purging a stale UUCP lock file (secs) [30]?
超时前清除旧的UUCP锁定文件[30]？30

Hold UUCP lockfile during inbound data calls [Yes]?
当传真进来时，保留UUCP 设定文件[Yes]? yes

Hold UUCP lockfile during inbound voice calls [Yes]?
当语音进来时，保留UUCP 设定文件[Yes]? yes

Percent good lines to accept during copy quality checking [95]?
线路好的时候，在什么百份比时进行检查[95]? 95

Max consecutive bad lines to accept during copy quality checking [5]?
线路不好的时候，在什么百份比时进行检查[5]? 5

Max number of pages to accept in a received facsimile [25]?
每次传真进来的最大可接收页数[25]? 25

Syslog faxility name for ServerTracing messages [daemon]?
系统日志跟踪记录程序[daemon]? 按enter默认

Set UID to 0 to manipulate CLOCAL [“”]?
设置的UID为0操作CLOCAL[“”]? 按enter默认

Use available priority job scheduling mechanism [“”]?
使用现有的优先工作调度机制[“”]? 按enter默认

Are these ok [yes]?
确认以上信息是否正确[yes]? yes

Probing for best speed to talk to modem：38400
探索最佳速度交谈调制解调器： 38400

How should it be configured [1]?
应如何配置[1]? 1

DTE-DCE flow control scheme [default]?
流量控制方案[default]? 按enter默认

Are these ok [yes]?
确认以上信息是否正确[yes]? yes

Do you want to run faxaddmodem to configure another modem [yes]?
你想运行的另一个faxaddmodem配置调制解调器[yes]? no

Should I run faxmodem for each configured modem [yes]?
应该为每个运行faxmodem配置调制解调器[yes]? yes

Done verifying system setup.
完成核查系统设置。

编辑/etc/ttys 文件 ，查找“ttyd0″字节，修改为下面值(如果没有找，就在最后加上）

#vi /etc/ttys
ttyd0   “/usr/local/sbin/faxgetty”      dialup  on

设置开机HylaFax服务自动运行

#cp /usr/local/etc/rc.d/hylafax.sh.sample /usr/local/etc/rc.d/hylafax.sh

启动HylaFax服务

#/usr/local/etc/rc.d/hylafax.sh start

HylaFax命令

faxstat -s （显示队列中等待发送的传真）
faxstat -d （显示已发送的传真）
faxstat -r （显示已接收的传真）
faxrm number\_of\_job    (从队列中去删除一个传真)
faxqclean    (清除缓冲池)
faxcron        (显示统计结果)

——————————————————————————————–

AvantFAX (WebSite [http://www.avantfax.com](http://www.avantfax.com/) )

AvantFAX是一种Web应用管理传真的HylaFAX 服务器。
AvantFAX允许用户在任何平台上，来查看和发送传真，而无需安装特殊的软件。它还允许管理员管理用户，他们的权限，传真线，传真类等
AvantFAX可以从本地网络，并通过互联网远程使用标准的网络设备

安装说明: [http://www.avantfax.com/install.php](http://www.avantfax.com/install.php)

安装AvantFAX之前要先安装以下软件：

HylaFAX 4.4 or HylaFAX EE 3
PHP 5
PHP PEAR 5 including MDB2\_driver\_mysql, Mail and Mail_Mime
PECL FileInfo
PHP mbstring – for improved UTF-8 sorting support (optional)
PHP MySQL 5
MySQL server 4.1.12 or better (see Important Notes below)
Apache
ImageMagick
ghostscript
libtiff
netpbm-progs
libungif
sudo
sendmail/postfix/exim/qmail or use an external SMTP server
cups/lpr and psutils
expect

—————————————–
apache

Package方法安装
#pkg_add -r apache22

或者

Ports方法安装
#cd /usr/ports/www/apache22
#make install clean

—————————————–
mysql51-server

Package方法安装
#pkg_add -r mysql51-server

或者

Ports方法安装

#cd /usr/ports/databases/mysql51-server
#make install clean

—————————————–
PHP5

Ports方法安装
#cd /usr/ports/lang/php5
#make install clean
//安装时记得选上第三项“APACHE Build Apache module”支持

—————————————–
PHP5-session

Package方法安装
#pkg_add -r php5-session

或者

Ports方法安装
#cd /usr/ports/www/php5-session
#make install clean

—————————————–
php5-mysql

Package方法安装
#pkg_add -r php5-mysql

或者

Ports方法安装
#cd /usr/ports/databases/php5-mysql
#make install clean

—————————————–
pear-DB

Package方法安装
#pkg_add -r pear-DB

或者

Ports方法安装
#cd /usr/ports/databases/pear-DB
#make install clean

—————————————–
pear-MDB2\_Driver\_mysql

Ports方法安装
#cd /usr/ports/databases/pear-MDB2\_Driver\_mysql
#make install clean

—————————————–
pear-Auth

Package方法安装
#pkg_add -r pear-Auth

或者

Ports方法安装
#cd /usr/ports/security/pear-Auth
#make install clean

—————————————–
pear-Auth_SASL

Package方法安装
#pkg\_add -r pear-Auth\_SASL

或者

Ports方法安装
#cd /usr/ports/security/pear-Auth_SASL
#make install clean

—————————————–
pear-Net_SMTP

Package方法安装
#pkg\_add -r pear-Net\_SMTP

或者

Ports方法安装
#cd /usr/ports/net/pear-Net_SMTP
#make install clean

—————————————–
pear-Mail

Package方法安装
#pkg_add -r pear-Mail

或者

Ports方法安装
#cd /usr/ports/mail/pear-Mail
#make install clean

—————————————–
pear-Mail_Mime

Package方法安装
#pkg\_add -r pear-Mail\_Mime

或者

Ports方法安装
#cd /usr/ports/mail/pear-Mail_Mime
#make install clean

—————————————–
pear-Mail_mimeDecode

Package方法安装
#pkg\_add -r pear-Mail\_mimeDecode

或者

Ports方法安装
#cd /usr/ports/mail/pear-Mail_mimeDecode
#make install clean

—————————————–
pear-PHP_Compat

Package方法安装
#pkg\_add -r pear-PHP\_Compat

或者

Ports方法安装
#cd /usr/ports/devel/pear-PHP_Compat
#make install clean

—————————————–
pear-HTML_Common

Ports方法安装
#cd /usr/ports/devel/pear-HTML_Common
#make install clean

—————————————–
pear-HTML_QuickForm

Package方法安装
#pkg\_add -r pear-HTML\_QuickForm

或者

Ports方法安装
#cd /usr/ports/devel/pear-HTML_QuickForm
#make install clean

—————————————–
pecl-fileinfo

Package方法安装
#pkg_add -r pecl-fileinfo

或者

Ports方法安装
#cd /usr/ports/sysutils/pecl-fileinfo
#make install clean

—————————————–
php5-mbstring

Package方法安装
#pkg_add -r php5-mbstring

或者

Ports方法安装
#cd /usr/ports/converters/php5-mbstring
#make install clean

—————————————–
ImageMagick

Package方法安装
#pkg_add -r ImageMagick

或者

Ports方法安装
#cd /usr/ports/graphics/ImageMagick
#make install clean

—————————————–
smarty

Package方法安装
#pkg_add -r smarty

或者

Ports方法安装
#cd /usr/ports/www/smarty
#make install clean

—————————————–
netpbm

Package方法安装
#pkg_add -r netpbm

或者

Ports方法安装
#cd /usr/ports/graphics/netpbm
#make install clean

—————————————–
libungif

Package方法安装
#pkg_add -r libungif

或者

Ports方法安装
#cd /usr/ports/graphics/libungif
#make install clean

—————————————–
sudo

Package方法安装
#pkg_add -r sudo

或者

Ports方法安装
#cd /usr/ports/security/sudo
#make install clean

—————————————–
cups

Package方法安装
#pkg_add -r cups

或者

Ports方法安装
#cd /usr/ports/print/cups
#make install clean

—————————————–
psutils-a4

Package方法安装
#pkg_add -r psutils-a4

或者

Ports方法安装
#cd /usr/ports/print/psutils-a4
#make install clean

—————————————–
expect

Package方法安装
#pkg_add -r expect

或者

Ports方法安装
#cd /usr/ports/lang/expect
#make install clean

—————————————————————————————–

软件下载： [http://www.avantfax.com/download.php](http://www.avantfax.com/download.php)

将下载到的软件包通过FTP上传到服务器的vincent目录下,解压:
#cd /home/vincent
#gunzip avantfax-3.1.6.tgz.gz
#tar zxvf avantfax-3.1.6.tgz

移动avantfax Web目录:
#cd avantfax-3.1.6
#mv avantfax /usr/local/www/
#chmod -R 777 /usr/local/www/avantfax/tmp /usr/local/www/avantfax/faxes
#chown -R www:www /usr/local/www/avantfax//includes/templates

\# ln -s /usr/local/www/avantfax/includes/faxrcvd.php /var/spool/hylafax/bin/faxrcvd.php
\# ln -s /usr/local/www/avantfax/includes/dynconf.php /var/spool/hylafax/bin/dynconf.php
\# ln -s /usr/local/www/avantfax/includes/notify.php /var/spool/hylafax/bin/notify.php

编辑config.ttyd0 //我的是外置modem,连接电脑com1，所以是ttyd0
\# vi /var/spool/hylafax/etc/config.ttyd0

//添加
#
\## AvantFAX configuration
#
FaxrcvdCmd:     bin/faxrcvd.php
DynamicConfig:  bin/dynconf.php
UseJobTSI:      true

————————————-
编辑config
\# vi /var/spool/hylafax/etc/config

//添加
#
\## AvantFAX configuration
#
NotifyCmd:      bin/notify.php

————————————–
备份/替换faxcover程序
#mv /usr/local/bin/faxcover /usr/local/bin/faxcover.old
#ln -s /usr/local/www/avantfax/includes/faxcover.php /usr/local/bin/faxcover

设置HylaFax用户支持Avantfax
#/usr/local/sbin/faxadduser -a pwd www
#/usr/local/sbin/faxdeluser localhost
#/usr/local/sbin/faxdeluser 127.0.0.1
#echo 127.0.0.1 >> /var/spool/hylafax/etc/hosts.hfaxd

编辑Avantfax设置文件
#cd /usr/local/www/avantfax/includes
#cp local\_config-example.php local\_config.php
#vi local_config.php

$BINARYDIR                      = ‘/usr/bin’;
//修改为:
$BINARYDIR                      = ‘/usr/local/bin’;

$HYLAFAX_PREFIX                = ‘/usr’;
//修改为:
$HYLAFAX_PREFIX                = ‘/usr/local’;

$WWWUSER                        = ‘apache’;
//修改为:
$WWWUSER                        = ‘www’;

$dft\_config\_lang            = ‘en’;
//修改为:
$dft\_config\_lang             = ‘zh’;

——————————————————-
编辑hfaxd.conf
#vi /usr/local/lib/fax/hfaxd.conf

#JobFmt:                “%-3j %3i %1a %6.6o %-12.12e %5P %5D %7z %.25s”
//修改为:
JobFmt:         “%-3j %3i %1a %15o %40M %-12.12e %5P %5D %7z %.25s”

——————————————————-
软件启动设置:
#vi /etc/rc.conf
//添加
apache22_enable=”YES”
mysql_enable=”YES”

——————————————————-
apache22设置

#vi /usr/local/etc/apache22/httpd.conf

DirectoryIndex index.html
//修改为:
DirectoryIndex index.html index.php

//添加
AddType application/x-httpd-php .php
AddType application/x-httpd-php-source .phps

——————————————————-
新建fax.conf
#vi /usr/local/etc/apache22/Includes/fax.conf

//添加
NameVirtualHost *:80

ServerName fax.test.org
DocumentRoot /usr/local/www/avantfax/


AllowOverride None
Options None
Order allow,deny
Allow from all

——————————————————–
PHP程序连接
#ln-s /usr/local/bin/php /usr/bin/php

启动apache
#/usr/local/etc/rc.d/apapche start

——————————————————–
avantfax数据导入

启动mysql
#/usr/local/etc/rc.d/mysql start

导入数据
#cd /home/vincent/avantfax-3.1.6
#mysql -uroot < create_user.sql
#mysql -uavantfax -pd58fe49 avantfax < create_tables.sql

——————————————————–
编辑/etc/crontab
\# vi /etc/crontab

//添加
\# runs once an hour to update the phone book
0 \* \* \* \* root /usr/local/www/avantfax/includes/phb.php
\# runs once a day to remove old files
0 0 \* \* * root /usr/local/www/avantfax/includes/avantfaxcron.php -t 2

———————————————————
编辑/usr/local/etc/sudoers
#vi /usr/local/etc/sudoers

//添加
#Defaults    requiretty
www ALL = NOPASSWD: /sbin/reboot, /sbin/halt, /usr/local/sbin/faxdeluser, /usr/local/sbin/faxadduser -u \* -p \* *

———————————————————
打开浏览器（IE/firefox/opera)，打上下面的网址：
[http://192.168.1.203/admin/](http://192.168.1.203/admin/)
username: admin
password: password

新建 “传真分类” —> 新建 “Modem” —> 新建 “用户”
到此，服务器基本上可以使用了。

![](http://bbs3.chinaunix.net/images/smilies/icon_smile.gif) Good luck!

———————————————————
参考资料：
《FreeBSD+Hylafax+frogfax配置FAX服務器》
[http://www.extmail.org/forum/viewthread.php?tid=5498](http://www.extmail.org/forum/viewthread.php?tid=5498)

《Centos 5.2 + HylaFAX + Apache + MySQL + PHP + AvantFAX + HylaFAX-Clinet》
[http://bbs.chinaunix.net/viewthread.php?tid=1232431](http://bbs.chinaunix.net/viewthread.php?tid=1232431)

[ _本帖最后由 tdls 于 2009-5-20 00:08 编辑_ ]

2009-5-18 01:24

下载次数: 31


![](http://bbs3.chinaunix.net/images/attachicons/rar.gif)[avantfax.rar](http://bbs3.chinaunix.net/attachment.php?aid=331591)(8.29 KB)

# uname -a

FreeBSD fax.test.org 7.2-RELEASE FreeBSD 7.2-RELEASE #0: Fri May  108:49:13 UTC 2009    [root@walker.cse.buffalo.edu](mailto:root@walker.cse.buffalo.edu):/usr/obj/usr/src/sys/GENERIC  i386


———————————————————-

# pkg_info


fax# pkg_info

ImageMagick-6.5.1.1 Image processing tools

apache-2.2.11_4     Version 2.2.x of Apache web server with prefork MPM.

autoconf-2.62       Automatically configure source code on many Un*x platforms

autoconf-wrapper-20071109 Wrapper script for GNU autoconf

cscope-15.7         An interactive C program browser

ctags-5.7           A feature-filled tagfile generator for vi and emacs clones

cups-1.3.9          Common UNIX Printing System: Metaport to install complete s

cups-base-1.3.9_3   Common UNIX Printing System

cups-pstoraster-8.15.4_2 Postscript interpreter for CUPS printing to non-PS printers

expat-2.0.1         XML 1.0 parser written in C

expect-5.44.1.11_1  A sophisticated scripter based on tcl/tk

fontconfig-2.6.0,1  An XML-based font configuration API for X Windows

freetype2-2.3.9_1   A free and portable TrueType font rendering engine

gettext-0.17_1      GNU gettext package

ghostscript8-8.64_1 Ghostscript 8.x PostScript interpreter

glib-1.2.10_12      Some useful routines of C programming (previous stable vers

gmake-3.81_3        GNU version of ‘make’ utility

gnutls-2.6.4        GNU Transport Layer Security library

gsfonts-8.11_4      Fonts used by GNU Ghostscript (or X)

gtk-1.2.10_20       Gimp Toolkit for X11 GUI (previous stable version)

help2man-1.36.4_2   Automatically generating simple manual pages from program o

hylafax-4.4.4       Fax software

inputproto-1.5.0    Input extension headers

jasper-1.900.1_7    An implementation of the codec specified in the JPEG-2000 s

jbigkit-1.6         Lossless compression for bi-level images such as scanned pa

jpeg-6b_7           IJG’s jpeg compression utilities

kbproto-1.0.3       KB extension headers

lcms-1.18,1         Light Color Management System — a color management library

libICE-1.0.4_1,1    Inter Client Exchange library for X11

libSM-1.1.0_1,1     Session Management library for X11

libX11-1.2.1,1      X11 library

libXau-1.0.4        Authentication Protocol library for X11

libXdmcp-1.0.2_1    X Display Manager Control Protocol library

libXext-1.0.5,1     X11 Extension library

libXft-2.1.13       A client-sided font API for X applications

libXi-1.2.1,1       X Input extension library

libXrender-0.9.4_1  X Render extension library

libXt-1.0.5_1       X Toolkit library

libfpx-1.2.0.12_1   Library routines for working with Flashpix images

libgcrypt-1.4.4     General purpose crypto library based on code used in GnuPG

libgpg-error-1.7    Common error values for all GnuPG components

libiconv-1.11_1     A character set conversion library

libltdl-1.5.26      System independent dlopen wrapper

libpthread-stubs-0.1 This library provides weak aliases for pthread functions

libungif-4.1.4_5    Tools and library routines for working with GIF images

libxcb-1.2_1        The X protocol C-language Binding (XCB) library

libxml2-2.7.3       XML parser library for GNOME

m4-1.4.12,1         GNU m4

mysql-client-5.1.33 Multithreaded SQL database (client)

mysql-server-5.1.33 Multithreaded SQL database (server)

netpbm-10.26.60     A toolkit for conversion of images between different format

p5-gettext-1.05_2   Message handling functions

pcre-7.9            Perl Compatible Regular Expressions library

pear-1.7.2          PEAR framework for PHP

pear-Auth-1.6.1     PEAR class for creating an authentication system

pear-Auth_SASL-1.0.2 PEAR abstraction of various SASL mechanism responses

pear-DB-1.7.13,1    PEAR Database Abstraction Layer

pear-HTML_Common-1.2.4 PEAR::HTML_Common is a base class for other HTML classes

pear-HTML_QuickForm-3.2.10 Provide methods for creating, validating and processing HTM

pear-MDB2-2.4.1     PEAR database abstraction layer

pear-MDB2_Driver_mysql-1.4.1 PEAR mysql MDB2 driver

pear-Mail-1.1.14    PEAR class that provides multiple interfaces for sending em

pear-Mail_Mime-1.5.2,2 PEAR classes to create and decode MIME messages

pear-Mail_mimeDecode-1.5.0 Provides a class to decode mime messages

pear-Net_SMTP-1.3.2 PEAR class that provides an implementation of the SMTP prot

pear-Net_Socket-1.0.9 PEAR Network Socket Interface

pear-PHP_Compat-1.5.0 Provides missing functionality for older versions of PHP

pecl-fileinfo-1.0.4 A PECL extension to retrieve info about files

perl-5.8.9_2        Practical Extraction and Report Language

php5-5.2.9          PHP Scripting Language

php5-mbstring-5.2.9 The mbstring shared extension for php

php5-mysql-5.2.9    The mysql shared extension for php

php5-pcre-5.2.9     The pcre shared extension for php

php5-session-5.2.9  The session shared extension for php

php5-xml-5.2.9      The xml shared extension for php

pkg-config-0.23_1   A utility to retrieve information about installed libraries

png-1.2.35          Library for manipulating PNG images

psutils-a4-1.17_2   Utilities for manipulating PostScript documents

python25-2.5.4_1    An interpreted object-oriented programming language

renderproto-0.9.3   RenderProto protocol headers

smarty-2.6.22       The PHP compiling template engine

sudo-1.6.9.20       Allow others to run commands as root

tcl-8.5.6_3         Tool Command Language

tcl-modules-8.5.6   Tcl common modules

tiff-3.8.2_3        Tools and library routines for working with TIFF images

tk-8.5.6_1          Graphical toolkit for TCL

vim-7.2.132         Vi “workalike”, with many additional features

wget-1.11.4         Retrieve files from the Net via HTTP(S) and FTP

xcb-proto-1.4       The X protocol C-language Binding (XCB) protocol

xextproto-7.0.5     XExt extension headers

xproto-7.0.15       X11 protocol headers


[ _本帖最后由 tdls 于 2009-5-18 01:18 编辑_]