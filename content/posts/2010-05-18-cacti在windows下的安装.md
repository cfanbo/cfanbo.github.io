---
title: Cacti在Windows下的安装
author: admin
type: post
date: 2010-05-18T02:01:53+00:00
url: /archives/3652
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - CACTI

---
官方教程：

>

该安装文档是我参照 [www.cacti.net](http://www.cacti.net/) 上的官方文档进行安装后，总结出来的。平台是winxp或win2k。我把涉及到的软件制作了个安装包,忽略了版本号,这样可以让大家正确选择,少走很多 我弯路,http://www.bgctv.cn/cacti.rar安装手册内的很多细节是针对新手的,希望更多的人可以使用)

**1、安装mysql.(版本mysql-3.23.52)**
安装包内的mysql.rar解压缩,正常安装到c盘的mysql目录;安装后需要手动执行C:\MYSQL\BIN \winmysqladmin.exe文件,其实就是找到他,双击一下就可以了,他会启动mysql要求你设置mysql的用户名密码,我设置的是用户 名:root 密码:cacti 这个用户名和密码会用到几次,请记清楚

**2、安装apache(版本apache_2.0.49-win32-x86)**

正常安装包内的版本,约定一下,我们把软件安装到C:\APACHE2目录下,正常情况下,你在浏览器里打 [http://127.0.0.1](http://127.0.0.1/)
就可以看到apache正常启动了,屏幕右下脚会有个小标志(红色的羽毛),双击一下可以打开,可以控制apache服务停止或启动,你可以试验一下,一 会儿会用到

**3、安装php(版本php-4.3.5RC1-Win32)**

把安装包内的php.rar解压缩,然后拷贝到c盘根目录下,然后进入php文件夹把php4ts.dll这个文件分别拷贝 c:\windows 下和C:\WINDOWS\SYSTEM32文件夹里面;然后把c:\php\php.ini.dist改名成php.ini并编辑这个文件,把下面这四 行添加到文件的末尾
**extension_dir = c:\php\extensions
extension=php_snmp.dll
extension=php_sockets.dll
session.save_path=c:\tmp**
\*****因为 c:\tmp 这个文件夹可能不存在,所以到这步时,你要在c盘根目录下建立一个文件夹名字就是 tmp

**4、设置apache支持php**
打开C:\APACHE2\APACHE2\CONF\httpd.conf , 在文件末尾加上下面几行
**AcceptPathInfo on
AddType application/x-httpd-php .php3
AddType application/x-httpd-php .php4
AddType application/x-httpd-php .php
AddType application/x-httpd-php .
AddType application/x-httpd-php .phtml
Action Application/x-httpd-php “c:/php/php.exe”
LoadModule php4_module c:/php/sapi/php4apache2.dll**

\*****然后找到 DirectoryIndex index.html index.html.var 这行,在后面加上 index.php index.htm,使这行变成
DirectoryIndex index.html index.html.var index.php index.htm
\*****然后重新启动apache就可以了,如果报错,你到事件查看器里看看是什么报错
**5、设置mysql**

你需要建立一个空的cacti数据库表,前面已经设置好了,可以用phpmyadmin这个文件,安装包里提供里,密码设置文件是CONFIG.INC, 我已经设置成
用户名root,密码cacti了,如果你的不同,可以改一下;把phpmyadmin文件夹拷贝到C:\APACHE2\APACHE2\htdocs 文件夹下,在浏览器里执行
[http://127.0.0.1/phpmyadmin](http://127.0.0.1/phpmyadmin)
然后选创建一个数据库,名字就是cacti 选左边的数据库列表,进入cacti数据库,然后点上面的SQL标签,浏览下cacti.sql,这个文件我已经放到安装包里面了,其实他本来是在 cacti的压缩包里的,我简化一下
选中后,按下面的go去执行,马上就建立好了,ok!

**6、安装rrdtool**

安装包内的rrdtool.rar解压缩后放到c盘根目录就可以了
**7、安装net-snmp**

运行安装包内的net-snmp.exe,约定一下,安装到c盘的net-snmp目录就可以了

**8、安装cacti**

解压cacti.rar,拷贝到C:\APACHE2\APACHE2\htdocs目录下就可以了
\*****配置cacti :

进入C:\APACHE2\APACHE2htdocs/cacti/include/config.php

**$database_type = “mysql”;
$database_default = “cacti”;
$database_hostname = “localhost”;
$database_username = “root”;
$database_password = “cacti”;**

核对以上几项是否正确

**9、安装cactid**
解压安装包内的cactid.rar,然后拷贝到c盘根目录,设置cactid.conf
DB_Host localhost
DB_Database cacti
DB_User root
DB_Password cacti
核对以上几项是否正确

**10、页面设置**

在浏览器上输入：

[http://127.0.0.1/cacti](http://127.0.0.1/cacti)

进入cacti的初始设置页面：

在这里我们要输入一些原始的信息：

NEXT－》输入一些信息，如rrdtool、php、snmpwalk、snmpget的位置，使用ucd-snmp

还是net-snmp等－》输入原始的用户和密码：admin/admin－》更改admin用户的密码

－》点击 Save

**11、设置系统路径**
鼠标右键点我的电脑－属性－高级－环境变量－系统变量－新建－MIBDIRS=c:\php\mibs, 新建 PHPRC=c:\php
**12、设置计划任务**
和mrtg一样，cacti也需要每分钟执行采集一下相关snmp信息
开始－设置－控制面板－任务计划－添加任务计划－浏览c:\php\php.exe 设置成每天执行，高级里面选每5分钟执行一次，持续24小时；再返回到属性的首页，运行(R)改成C:\php\php.exe C:/apache2/Apache2/htdocs/cacti/poller.php
起始于改成C:/apache2/Apache2/htdocs/cacti
然后进入cacti里加入一个snmp交换机看看，应该可以画图了，我监控了华为的全系列设备，都没什么问题
**13、设置cacti里的setting参数**
在浏览器里进入cacti，选setting，选path标签，把该填的都填上
PHP Binary Path:
c:\php\php.exe
RRDTool Binary Path:
c:\rrdtool\rrdtool.exe
SNMPGET, SNMPWALK Paths:
c:\net-snmp\bin\snmpget.exe
c:\net-snmp\bin\snmpget.exe
Cactid Path:
c:\cactid\cactid.exe
基本就这样
cacti的论坛帮了我很多忙 [http://forums.cacti.net](http://forums.cacti.net/)