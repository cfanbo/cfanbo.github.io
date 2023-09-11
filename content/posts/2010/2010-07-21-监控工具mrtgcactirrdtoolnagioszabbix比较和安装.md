---
title: 监控工具mrtg,cacti,rrdtool,nagios,zabbix比较和安装
author: admin
type: post
date: 2010-07-21T11:59:55+00:00
url: /archives/4738
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - CACTI
 - mrtg
 - nagios
 - rrdtool
 - zabbix

---

[cacti](http://www.cacti.net/) 是一个用 [rrdtool](http://people.ee.ethz.ch/~oetiker/webtools/rrdtool/) 来画图的网络监控系统, 通常一说到网络管理, 大家首先想到的经常是 [mrtg](http://people.ee.ethz.ch/~oetiker/webtools/mrtg/), 但是 mrtg 画的图简单且难看, rrdtool 虽然画图本领一流, 画出来的图也漂亮, 但是他也就是一个画图工具, 不像 mrtg 那样本身还集成了数据收集功能. cacti 则是集成了各种数据收集功能,然后用 rrdtool 画出监控图形. 其本身界面比起同类系统要漂亮不少. 推荐所有有监控需求的人都去研究一下.


cacti 和 [nagios](http://www.nagios.org/) 是不同功用的系统, nagios 适合监视大量服务器上面的大批服务是否正常, 重点并不在图形化的监控, 其集成的很多功能例如报警,都是 cacti 没有或者很弱的. cacti 主要用途还是用来收集历史数据和画图, 所以界面比 nagios 漂亮很多.

[net-snmp](http://net-snmp.sourceforge.net/) 是一套广泛使用在类 unix 系统上的 [snmp](http://www.cisco.com/univercd/cc/td/doc/cisintwk/ito_doc/snmp.htm) 软件, 包含一套 snmp agent 框架 ,一个 snmpd 和 一堆 snmp 工具 , 其前身为 ucd-snmp. 关于 snmp 是什么, 以及如何配置的文章,网上搜一下有一堆一堆的. 在这里就不重复了.

[squid](http://www.squid-cache.org/) 是一个 web 缓存加速程序, 本来跟监控没有太大关系, 只是因为他支持 snmp 查询,而我要用 cacti 监控他, 然后遇到了他的缺陷被折腾了一阵子,所以也拉进今天的讨论.


我跟这三个东西斗争的过程如下…


首先先把 cacti 架起来, 在架的过程中我没有遇到问题,但是把 czz 搞了一下, 因为 cacti 要调用外部程序, 不能开 safe_mode, 如果开了就会出奇怪问题.


接下来配置 squid 的查询, squid 的查询数据比较多且复杂,自己做模版的话很麻烦,于是 google 了一下,找了一个 SquidStats ( [见附件](http://kangkang.org/wordpress/wp-content/uploads/2006/01/SquidStats-0.1.zip)) 的模版, 按照他的 readme 一步一步来, 就可以正常安装. 于是我就遇到了第一个坎…


设置完成以后执行 poller 的时候总是无法产生 rrd 数据, 给 php 里面加 log 也没有看出来什么, google 换了很多关键词, 总算发现了 [原因](http://bugs.cacti.net/view.php?id=644): cacti 在进行 snmp 查询之前会先确定对方是否在运行, 他用的方法是查询 .1.3.6.1.2.1.1.3.0 这个 oid, 但是 squid 不支持这个 oid , 于是 cacti 就以为 squid down 了,不去真正查询. 临时解决方法是在 cacti 的 settings 里面, poller 页的 Downed Host Detection 选择 Ping, 不要选择带有 snmp 字样的.

然后在弄 64 位机的时候遇到了第二个坎, 发现 64 位 linux 机器的流量图总是不正确. 大部分时候没有结果,有时候又特大, 其实这个应该很容易想到是 counter 回绕不正确的问题, 但是我第一次 google 出来的结果是一个说和 tunnel 设备有关的 bug, 这两台 x64 机器上面确实有 tunnel ,于是我就一直以为是 tunnel 的问题. 折腾了好久. 最后才发现原来就是简单的 counter 回绕不正确的问题. 从 Fedora Core 5 的开发目录里面下一个 net-snmp 5.3 的 srpm 在 centos 4.2 上 build 一下, 就搞定了. 注意 FC4 里面的 net-snmp 5.2.x 也是有 bug 的,一定要 5.3 的.


最后推荐所有研究 cacti 的人,一定不要放过 cacti 的官方论坛 [扩展脚本版面](http://forums.cacti.net/forum-12.html) . 里面有很多的第三方的模版和脚本, 支持很多的网络设备.


[http://tewuxiaoqiang.blog.51cto.com/279711/161207](http://tewuxiaoqiang.blog.51cto.com/279711/161207) **Cacti Nagios比较**

[http://www.oschina.net/p/zabbix](http://www.oschina.net/p/zabbix) zabbix是一个基于WEB界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案。zabbix能监视各种网络参数，保证服务器系统的安全运营；并提供柔软的通知机制以让系统管理员快速定位/解决存在的各种问题。

zabbix由2部分构成，zabbix server与可选组件zabbix agent。

zabbix server可以通过SNMP，zabbix agent，ping，端口监视等方法提供对远程服务器/网络状态的监视，数据收集等功能，它可以运行在Linux, Solaris, HP-UX, AIX, Free BSD, Open BSD, OS X等平台之上。

zabbix agent需要安装在被监视的目标服务器上，它主要完成对硬件信息或与操作系统有关的内存，CPU等信息的收集。zabbix agent可以运行在Linux ,Solaris, HP-UX, AIX, Free BSD, Open BSD, OS X, Tru64/OSF1, Windows NT4.0, Windows 2000/2003/XP/Vista)等系统之上。

zabbix server可以单独监视远程服务器的服务状态；同时也可以与zabbix agent配合，可以轮询zabbix agent主动接收监视数据（trapping方式），同时还可被动接收zabbix agent发送的数据（trapping方式）。

另外zabbix server还支持SNMP (v1,v2)，可以与SNMP软件(例如：net-snmp)等配合使用。

zabbix的主要特点：

– 安装与配置简单，学习成本低

– 支持多语言（包括中文）

– 免费开源

– 自动发现服务器与网络设备

– 分布式监视以及WEB集中管理功能

– 可以无agent监视

– 用户安全认证和柔软的授权方式

– 通过WEB界面设置或查看监视结果

– email等通知功能

等等

Zabbix主要功能：

– CPU负荷

– 内存使用

– 磁盘使用

– 网络状况

– 端口监视

– 日志监视

zabbix的License：GPL v2

标签： Linux PHP C/C++ 系统监控

开发语言： PHP C/C++

项目主页： [http://www.zabbix.com/](http://www.zabbix.com/)

文档地址： [http://www.zabbix.com/documentation.php](http://www.zabbix.com/documentation.php)

下载地址： [http://www.zabbix.com/download.php](http://www.zabbix.com/download.php)

收录时间：2008年09月16日


首先简单介绍一下: [cacti](http://www.cacti.net/) 是一个用 [rrdtool](http://people.ee.ethz.ch/~oetiker/webtools/rrdtool/) 来画图的网络监控系统, 通常一说到网络管理, 大家首先想到的经常是 [mrtg](http://people.ee.ethz.ch/~oetiker/webtools/mrtg/), 但是 mrtg 画的图简单且难看, rrdtool 虽然画图本领一流, 画出来的图也漂亮, 但是他也就是一个画图工具, 不像 mrtg 那样本身还集成了数据收集功能. cacti 则是集成了各种数据收集功能,然后用 rrdtool 画出监控图形. 其本身界面比起同类系统要漂亮不少. 推荐所有有监控需求的人都去研究一下.


cacti 和 [nagios](http://www.nagios.org/) 是不同功用的系统, nagios 适合监视大量服务器上面的大批服务是否正常, 重点并不在图形化的监控, 其集成的很多功能例如报警,都是 cacti 没有或者很弱的. cacti 主要用途还是用来收集历史数据和画图, 所以界面比 nagios 漂亮很多


1. 主要对流量及主机在线状态监控软件,如最初的MRTG,PRGT,CACTI,Hobbit,

2. 能对服务器的关键服务及进程进行监控的软件,如Big Brother,Nagios,


[http://blog.chinaunix.net/u/12909/showart_1073431.html](http://blog.chinaunix.net/u/12909/showart_1073431.html) mrtg,cacti,rrdtool,nagios, zabbix安装


安装net-snmp

下载net-snmp-5.3.0.1-1.EL4.i386.rpm

安装mrtg：www.mrtg.org

下载

[mrtg-2.12.2.tar.gz](http://people.ee.ethz.ch/~oetiker/webtools/mrtg/pub/mrtg-2.12.2.tar.gz)

./configure   –prefix=/usr/local/mrtg & make & make install   cp /usr/local/mrtg/bin/*   /usr/bin

安装rrdtool：

[www.rrdtool.org](http://www.rrdtool.org/)

[http://people.ee.ethz.ch/~oetiker/webtools/rrdtool/doc/rrdbuild.en.html](http://people.ee.ethz.ch/~oetiker/webtools/rrdtool/doc/rrdbuild.en.html)

(以下部分可以直接copy到linux shell下 自动安装,我是分段copy,整体copy未尝试)

BUILD_DIR=/tmp/rrdbuild

INSTALL_DIR=/usr/local/rrdtool

mkdir -p $BUILD_DIR

mkdir $BUILD_DIR/lb

cd $BUILD_DIR

#####zlib

wget

[http://people.ee.ethz.ch/oetiker/webtools/rrdtool/pub/libs/zlib-1.2.2.tar.gz](http://people.ee.ethz.ch/oetiker/webtools/rrdtool/pub/libs/zlib-1.2.2.tar.gz)

tar zxvf zlib-1.2.2.tar.gz

cd zlib-1.2.2

env CFLAGS=”-O3 -fPIC” ./configure –prefix=$BUILD_DIR/lb

make

make install

cd ..

rm -fR zlib*

#####libpng

wget

[http://people.ee.ethz.ch/oetiker/webtools/rrdtool/pub/libs/libpng-1.2.8-config.tar.gz](http://people.ee.ethz.ch/oetiker/webtools/rrdtool/pub/libs/libpng-1.2.8-config.tar.gz)

tar zxvf libpng-1.2.8-config.tar.gz

cd libpng-1.2.8-config

env CPPFLAGS=”-I$BUILD_DIR/lb/include” LDFLAGS=”-L$BUILD_DIR/lb/lib” \

CFLAGS=”-O3 -fPIC” ./configure –disable-shared –prefix=$BUILD_DIR/lb

make

make install

cd ..

rm -fR libpng*

#########freetype

wget

[http://people.ee.ethz.ch/oetiker/webtools/rrdtool/pub/libs/freetype-2.1.9.tar.gz](http://people.ee.ethz.ch/oetiker/webtools/rrdtool/pub/libs/freetype-2.1.9.tar.gz)

tar zxvf freetype-2.1.9.tar.gz

cd freetype-2.1.9

env CPPFLAGS=”-I$BUILD_DIR/lb/include” LDFLAGS=”-L$BUILD_DIR/lb/lib” CFLAGS=”-O3 -fPIC” ./configure –disable-shared –prefix=$BUILD_DIR/lb

make

make install

cd ..

rm -fR freetype*

#######libart_lgpl

wget

[http://people.ee.ethz.ch/oetiker/webtools/rrdtool/pub/libs/libart_lgpl-2.3.17.tar.gz](http://people.ee.ethz.ch/oetiker/webtools/rrdtool/pub/libs/libart_lgpl-2.3.17.tar.gz)

tar zxvf libart_lgpl-2.3.17.tar.gz

cd libart_lgpl-2.3.17

env CFLAGS=”-O3 -fPIC” ./configure –disable-shared –prefix=$BUILD_DIR/lb

make

make install

cd ..

rm -fR libart*

########cgilib

wget http://people.ee.ethz.ch/~oetiker/webtools/rrdtool/pub/libs/cgilib-0.5.tar.gz

tar zxvf cgilib-0.5.tar.gz

cd cgilib-0.5

make CC=gcc CFLAGS=”-O3 -fPIC -I.”

mkdir -p $BUILD_DIR/lb/include

cp *.h $BUILD_DIR/lb/include

mkdir -p $BUILD_DIR/lb/lib

cp libcgi* $BUILD_DIR/lb/lib

cd ..

rm -fR cgilib*

#########install rrdtool

ranlib $BUILD_DIR/lb/lib/*.a # 优化

IR=-I$BUILD_DIR/lb/include

CPPFLAGS=”$IR $IR/libart-2.0 $IR/freetype2 $IR/libpng”

LDFLAGS=”-L$BUILD_DIR/lb/lib”

CFLAGS=-O3

export CPPFLAGS LDFLAGS CFLAGS

cd $BUILD_DIR/

wget http://people.ee.ethz.ch/~oetiker/webtools/rrdtool/pub/rrdtool-1.2.12.tar.gz

tar zxvf rrdtool-1.2.12.tar.gz

cd   rrdtool-1.2.12

./configure –prefix=$INSTALL_DIR –disable-python –disable-tcl

make

make install

cd ..

rm -fR rrdtool*

***********************RRDTOOL INSTALL SUCCESSFULLY**********************


安装CACTI：www.cacti.net

cacti-0.8.6h.tar.gz

tar zxvf cacti-0.8.6h.tar.gz

cp –R cacti-0.8.6h   /usr/local/cacti/

配置Mysql：

group-add cacti

useradd -g cacti cactiuser

/usr/local/mysql/bin/mysql – –user=root – –password=yanhannet

mysql> create database cactidb;

mysql> grant all on cactidb.* to _cactiuser@localhost;_

mysql> set password for cactiuser@localhost=password(‘yanhannet’);

mysql> exit

# mysql –user=root –password=yanhannet cactidb

# chown -R cactiuser rra/ log/

# chmod –R 777 rra/ log/

# vi cacti/include/config.php

$database_type = “mysql”;

$database_default = “cactidb”;

$database_hostname = “localhost”;

$database_username = “cactiuser”;

$database_password = “yanhannet”;

$database_port = “3306”;

vi /etc/ctrontab (crontab -u cactiuser –e）

*/5 * * * * cactiuser php /usr/local/cacti/poller.php > /dev/null 2>&1

配置apache

vi /usr/local/apache/conf/conf.d/cacti.conf

Alias /cacti /usr/local/cacti

Options None

AllowOverride None

Order allow,deny

Allow from all

# AuthName “XXXXX”

# AuthType Basic

# AuthUserFile /XXX/htpasswd.users

# Require   valid-user


******************CACTI   INSTALL   SUCCESSFULLY ***********************


安装nagios

[www.nagios.org](http://www.nagios.org/)

useradd nagios

mkdir /usr/local/nagios

chown nagios.nagios /usr/local/nagios/

./configure –prefix=/usr/local/nagios –with-gd-lib=/usr/lib –with-gd-inc=/usr/include

注：红色部分为gd库位置，如果不加，这会出现 **The statusmap, trends and histogram CGIs are missing or dont work!**

查看3-D status map 需要在本机下载插件contvrml

http://www.parallelgraphics.com/bin/cortvrml.exe

在apache配置文档目录下

vi nagios.conf

scriptalias   /nagios/cgi-bin /usr/local/nagios/sbin

allowoverride authconfig

options execcgi

order allow,deny

allow from all

alias /nagios /usr/local/nagios/share

options none

allowoverride authconfig

order allow,deny

allow from all

在nagios sbin/ share/目录下

vi .htaccess

authname “nagios access”

authtype basic

authuserfile   /usr/local/nagios/etc/.nagios.users

require valid-user

生成用户文件

htpasswd -c /usr/local/nagios/etc/.nagios.users nagiosadmin

具体参数配置参考官方文档


工具插件：

[www.nagiosexchange.org](http://www.nagiosexchange.org/)

fruity 要求php5以上

下载：

[https://sourceforge.net/project/showfiles.php?group_id=136248](https://sourceforge.net/project/showfiles.php?group_id=136248)

http://pear.php.net/get/HTML_TreeMenu-1.2.0.tgz

[http://sourceforge.net/project/showfiles.php?group_id=42718](http://sourceforge.net/project/showfiles.php?group_id=42718)

http://puzzle.dl.sourceforge.net/sourceforge/adodb/adodb471-1.tgz

直接解压复制到fruity 下 分别改名为HTML 和adodb 其他不做修改

修改 fruity/includes下的config.ifg,需要更改的地方有，路径，mysql信息

mysql 添加fruity 数据库和user,password.

*************************NAGIOS INSTALL SUCCESSFULLY*****************


安装zabbix：

wget http://belnet.dl.sourceforge.net/sourceforge/zabbix/zabbix-1.1beta6.tar.gz

tar zxvf zabbix-1.1beta6.tar.gz

mysql -u   -p

> creat database zabbix;

>quit;

cd creat/mysql

mysql -u   -p zabbix

cd ../data

mysql -u   -p zabbix

cd ..

./configure –prefix=/usr/local/zabbix –with-mysql=/usr/local/mysql –enable-server –enable-agent

make

make install


cp misc/conf/* /etc/zabbix/conf/

cp frontends/php/* /usr/local/zabbix/php

修改apache添加zabbix.conf