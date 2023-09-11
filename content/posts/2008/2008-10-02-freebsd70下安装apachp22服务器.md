---
title: Freebsd7.0下安装APACHP22服务器
author: admin
type: post
date: 2008-10-02T17:29:51+00:00
excerpt: |
 |
 (1)最小化安装FREEBSD7.0-RELEASE
 (2)安装APACHE22
 b2sun.com#cd /usr/ports/www
 这个目录下会有apache22这个目录.安装它就OK了.
 b2sun.com#setenv PACKAGESITE ftp://ftp.freebsdchina.org/pub/FreeBSD/ports/i386/packages-7.0-release/Latest/
 b2sun.con#pkg_add -f -r apache22
 这时系统会自动下载文件并安装
 apache22_enable="YES" 这行加入/etc/rc.conf中.系统会自动启动这个服务.
 安装完成后您需要在/usr/local/www/apache22下面建立一个data的目录及一个index.html文件.这样就可正常启动apache22 并在其它客户端中访问您建立的服务器.
url: /archives/433
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache

---
(1)最小化安装FREEBSD7.0-RELEASE
(2)安装APACHE22
b2sun.com#cd /usr/ports/www
这个目录下会有apache22这个目录.安装它就OK了.
b2sun.com#setenv PACKAGESITE ftp://ftp.freebsdchina.org/pub/FreeBSD/ports/i386/packages-7.0-release/Latest/
**b2sun.con#pkg_add -f -r apache22**
这时系统会自动下载文件并安装
**apache22_enable=”YES”** 这行加入/etc/rc.conf中.系统会自动启动这个服务.
安装完成后您需要在/usr/local/www/apache22下面建立一个data的目录及一个index.html文件.这样就可正常启动apache22 并在其它客户端中访问您建立的服务器.
**(b2sun.com#apachectl start(stop restart))**这个非常关键.
最好 安装完后重新启动您的FreeBSD7操作系统.

**FAMP架构的建立**

LAMP架构早就闻名遐迩了，所谓的LAMP架构就是指Linux+Apache+MySQL+PHP(或Python或Perl)，是一组常用来搭建动态网站或者服务器的开源软件，本身都是各自独立的程序，但是因为常被放在一起使用，拥有了越来越高的兼容度，共同组成了一个强大的Web应用程序平台。

显然LAMP名字来源于其中每个程序的第一个字母，而这每个程序都是开源软件：Linux是开源的操作系统，Apache是最通用的网络服务器，MySQL是带有基于网络管理附加工具的关系数据库，PHP是流行的对象脚本语言。 随着开源潮流的蓬勃发展，开放源代码的LAMP已经与J2EE和.Net商业软件形成三足鼎立之势，并且该软件开发的项目在软件方面的投资成本较低，因此受到整个IT界的关注。

其实后三者都可以跨平台安装使用，如果将Linux系统换做Windows操作系统，那就叫WAMP架构，而如果把Linux换做FreeBSD系统，则叫做FAMP架构。可以说FAMP具有和LAMP同样的优势。

FreeBSD的软件安装大致分为三种：
1 pkg_add直接安装可执行程序。
2 ports编译安装。
3 源码安装。
以下的安装都是在FreeBSD 7.0版本上选择第二种安装方式即ports安装完成，其实选择package方式的话也大致相同。(发现7.0版本诸软件的安装与之前版本又有些不一样)。

**一 Apache安装：**
Apache 是 UNIX 系统中普遍使用的WWW服务器软件。根据Netcraft的统计 (http://news.netcraft.com/archives/web\_server\_survey.html)，目前因特网中，有超过百分之六十的服务器是使用Apache来提供网页浏览的服务。Apache可以说是目前世界上使用人数最多的网页服务器软件，它不仅可以在 FreeBSD、UNIX、Linux 中运行，也可以安装在 Windows 操作系统中。

Apache和FreeBSD一样，在软件版本上也有多个分支，FB7中存在着1.3、2.0和2.2版本。Apache1.3系列开发已久，已经十分稳定了，不会再有重大的修改。而Apache2及2.2系列是一个开发较活跃的版本，它和1.3最大的不同在于对多线程(multithreaded)的支持。我当然选择新潮的2.2系列，当前最新发布的版本是2.2.8。

在FreeBSD上安装Apache软件非常方便,以下使用ports方式安装:
\# cd /usr/ports/www/apache22
\# make install clean

以下是一些在实际使用中常常会遇到的问题：
1) 配置文件的位置：
在FreeBSD中位于/usr/local/etc/apache22/httpd.conf，在其他版本可能位置和名称有所不同。

2) 缺省的主目录：
/usr/local/www/apache22/data
系统安装好后,我在该目录下写入了一个简单的index.html文件,只一句:
\# nano index.html
This is a Debian server
然后在客户端检查是否输出正确。

3) log文件的位置
log文件的作用是很大的，Apache有二个log文件，一个是所有登陆本apache服务器的IP记录，/var/log/httpd-access.log，文件记录了登陆的ip，时间，浏览器类型等；另一个是联机错误记录文件， /var/log/httpd-error.log，这个文件对于调试apache参数是很有作用的。两个文件都是文本文件，可以由nano等文本编辑器来浏览、编辑，记录文件的位置及文件名是由 httpd.conf中的相应配置来改变。

4) 启动、停止和重新启动httpd服务器的运行：
#apachectl start(stop restart graceful)
这个命令比较有用，尤其是在修改配置文件之后。

5) 开机自动启动apache22服务：
需要编辑/etc/rc.conf文件，在其中加入以下语句:
apache22_enable=”YES”

6) 自动支持中文的问题
网页的缺省字符集有参数 AddDefaultCharset ISO-8859-1
这时候在浏览器浏览中文网页的时候，会乱码，需要手动设置编码方式为GBK或GB2312才能显示中文
去掉注释，修改为AddDefaultCharset GB2312就可以了。

7) Apache状态信息
在安装完 Apache 后，我们需要不断了解服务器的系统各方面的情况。Apache2内建了server-status及server-info二种查看服务器信息的方法。server-status是指服务器状态信息，我们可以了解Apache目前运行的情形，包括占用的系统资源、目前联机数量等。server-info主要是显示Apache的版本、加载的模块信息等。
为使用这两项功能，我们必须先修改 httpd.conf。
首先要把ExtendedStatus On前面的注释去掉。
然后分别找到和这两段，把两段内前面的注释都去掉，并设置好访问权限。不重视安全的话，可以设置allow from all.
然后就可以在浏览器以http://hostname/server-info访问了。

个人用户目录的问题：
修改主配置文件，注释掉#UserDir public_html这句，再在用户test的主目录/home/test下面创建一个index.html文件，就可以浏览：
http://yourip/~test了。

9) 其他一些我认为比较重要的配置参数：
ServerRoot:指出服务器保存其配置、出错和日志文件等的根目录。
Listen:允许你绑定Apache服务到指定的IP地址和端口上，以取代默认值
DocumentRoot:你的文档的根目录。默认情况下，所有的请求从这个目录进行应答。
HostnameLookups：指定记录用户端的名字还是IP地址

(在FreeBSD下使用ports安装apache22会出现类似的warming：
**No such file or directory: Failed to enable the ‘httpready’ Accept Filter**
解决方法是：
**#kldload accf_http**

并将以下语句写入到**/boot/defaults/loader.conf**中，以便下次启动自动装载模块
**accf\_data\_load=”YES”
accf\_http\_load=”YES”**
这是因为不能启动FreeBSD自带的一个基于http端口过滤的模块。这个模块的作用很不错——检查HTTP请求是否完整，符合规则accpt一个Http进程，否则就扔掉。)

值得说明的是，过去的开源WWW服务器几乎是Apache一统天下，近年来，则有两个小型的www服务器lighttp和nginx逐渐流行，也是值得考虑部署的好东西。

**二 PHP的安装：**
当前的FB7的ports中有两个php版本，即php4和php5，后者发布的时间已经很长了，我想现在应该很少应用系统非要选择安装php4吧，所以当然选择安装PHP5。
安装：
**\# cd /usr/ports/lang/php5
\# make install clean**
需要注意的是，缺省的php5配置参数当中没有选择将php5编译为apache的模块，而这是apache+php配合的主要模式，一般情况下都应该把这个选项选上。
编译安装完成之后，按照HandBook还应该在apache的配置文件(文件位置见上)中加入以下语句：
**LoadModule php5_module libexec/apache/libphp5.so
AddModule mod_php5.c**

DirectoryIndex index.php index.html

**AddType application/x-httpd-php .php
AddType application/x-httpd-php-source .phps**

但我在实践操作中，第一句实际上安装php5的时候就已经自动加上，而第二句在启动apache22的时候报错，不知道是不是apache13才那样，总之，实际修改apache配置文件的时候，前面两句不要。

现在在/usr/local/www/apache22/data下编写测试文件wen1.php文件如下：

再到客户端去打开该文件，如果出现以下界面，则意味着系统的php解析正确，还是目前最新的php5.2.5版本。

**三 MySQL的安装：**
FB7中的MySQL有三个版本，分别为4.0 5.0和5.1，下面我仍然安装最新潮的版本：
**\# cd /usr/ports/databases/mysql51-server
\# make install clean**
安装完成之后，还需要：
**\# /usr/local/bin/mysql\_install\_db
\# chown -R mysql /var/db/mysql**
这一步一定不能少，否则mysql将启动不起来。

启动mysql的方法是：
**\# /usr/local/bin/mysqld_safe &**
如果要设置开机就自动启动的话，需要编辑**/etc/rc.conf**文件，在其中加入：
**mysql_enable=”YES”**

因为缺省情况下MySQL的管理员帐户的密码为空，很不安全，所以修改管理员帐号密码这一步骤最好不要省略：
**\# /usr/local/bin/mysqladmin -u root password ‘123456’**

衡量你的MySQL服务器是否正常启动的方法之一是在命令行下面运行mysql的客户端命令(ports安装mysql-server的话，会一并连mysql-client也安装的)：
\# mysql
如果没有出错的提示而显示出mysql客户端控制台(如下)则表明Mysql服务器正常运行了：
\# mysql
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.1.24-rc FreeBSD port: mysql-server-5.1.24_1
Type ‘help;’ or ‘\h’ for help. Type ‘\c’ to clear the buffer.
mysql>

**四 安装php5-mysql**
现在php和mysql都安装好了，但现在的php5还不能操作Mysql数据库，或者说现在的php还缺乏mysql的驱动，另一方面，现在的php功能还很弱，相当多重要的也是常用的扩展还没有安装，这两个问题可以一并解决，就是安装ports里面的php5-extensions:
**\# cd /usr/ports/lang/php5-extensions
\# make install clean
\# apachectl graceful**
当前的php5-extensions里的选择一共有65个，我就不一一列举，一般至少如mysql GD zlib iconv等肯定是要选择的。
当安装完毕之后，再次运行上面所述的测试文件，发现现在的内容就很多了。

**五 其他可选的软件：**
1 PHP加速软件：
一般小型的应用使不使用PHP加速软件都没有问题，但是稍微大一点的应用若没有使用PHP加速软件，性能上就会相差比较大。这类软件比较多，出名的有ZendOptimizer和eAccelator，在FB7的ports中都有，以下为安装前者：
**\# cd /usr/ports/devel/zendoptimizer
\# make install clean
\# apachectl graceful**
注意这里编译安装后，系统提示，需要修改**/usr/local/etc/php.ini**文件，加入以下内容：

**[Zend]
zend\_optimizer.optimization\_level=15
zend\_extension\_manager.optimizer=”/usr/local/lib/php/20060613/Optimizer”
zend\_extension\_manager.optimizer\_ts=”/usr/local/lib/php/20060613/Optimizer\_TS”
zend_extension=”/usr/local/lib/php/20060613/ZendExtensionManager.so”
zend\_extension\_ts=”/usr/local/lib/php/20060613/ZendExtensionManager_TS.so”**

但是存在两个问题，一是/usr/local/etc目录下面并没有php.ini文件，需要自己把文件php.ini-dist复制为php.ini；但紧接着出现第二个问题：zendoptimizer仍然启动不了，报错说找不到libm.so.4文件，我不知道在FB7的ports里面，这算zendoptimizer的bug，还是compat-6.x的错误，反正我自己在/lib目录下这样做了一个连接解决问题：
**\# ln -s /usr/local/lib/compat/libm.so.4 /lib/libm.so.4**

再次运行上面的测试文件(wen1.php)，里面出现如下界面说明安装成功：

**2 phpmyadmin**
phpmyadmin就是一个操作MySQL数据库的Web界面，适合于不熟悉SQL语法的懒人们：
**\# cd /usr/ports/databases/phpmyadmin
\# make install clean**

就写到这里。

(今后但愿能改进本文档，希望能将apache扩展到lighttp和nginx；将php扩展到python和perl—其实上述安装后perl已经安装了；将mysql扩展到postgresql;)

作者： wenheping@gmail.com
时间： 20080502
来源： [][1]

 [1]: http://www.freebsdchina.org/forum/viewtopic.php?t=41804