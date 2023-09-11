---
title: '[精典教程]freebsd下安装mysql,apache,php,phpmyadmin记录'
author: admin
type: post
date: 2008-10-02T17:42:13+00:00
excerpt: |
 第一次在FREEBSD下配置环境，感觉好爽，安装的时候也参考了别人的介绍，在此表示感谢。
 为了方便以后的操作，现在记录写下来。

 安装MYSQL时要注意：
 mysql默认数据库放在/var分区里，如果你的数据库很大，那么你需要在前面分区的时候把/var分区分到足够大，
 如果你想改变它的安装目录，例如安装到：/usr/db，那么可以按如下方法：
 #mkdir /usr/db
 先在/usr建立一个数据库目录，然后
 #cd /usr/ports/databases/mysql50-server
 #make install clean
 开始下载并开始安装数据库。编译安装完之后，重启机器可以启动mysqld守护进程，可以
 #mysql
 如果能够见到
 mysql>
 提示符，说明安装好了。不过，刚装完的mysql默认的数据库连接是100个，远远不能应付大网站的要求。按照这个办法加大吧...
url: /archives/435
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - ftp
 - memcache
 - mysql
 - php
 - phpmyadmin

---
第一次在FREEBSD下配置环境，感觉好爽，安装的时候也参考了别人的介绍，在此表示感谢。

 为了方便以后的操作，现在记录写下来。

**安装MYSQL时要注意：
** mysql默认数据库放在/var分区里，如果你的数据库很大，那么你需要在前面分区的时候把/var分区分到足够大，
如果你想改变它的安装目录，例如安装到：/usr/db.

> **#cd /usr/ports/databases/mysql51-server
>** **#make WITH_CHARSET=utf8** **WITH_XCHARSET=all** **WITH\_PROC\_SCOPE\_PTH=yes SKIP\_DNS\_CHECK=yes BUILD\_OPTIMIZED=yes install clean**
>
> **#cp /usr/local/share/mysql/my-large.cnf    /etc/my.cnf**
> **#/usr/local/bin/mysql\_install\_db
> #chown -R mysql:mysql /var/db/mysql
> #/usr/local/bin/mysqld_safe & //启动mysql 服务
> #rehash
> #mysqladmin -u root password ‘root密码’
> #mysql -u root -p
> **输入root密码,进入mysql>提示符****

开始下载并开始安装数据库(上面的with_chraset=all不包含一些字符集的，如GBK,需要安装GBK教程参考:)。编译安装完之后，重启机器可以启动mysqld守护进程，可以

#mysql
如果能够见到
mysql>
提示符，说明安装好了。不过，刚装完的mysql默认的数据库连接是100个，远远不能应付大网站的要求。按照这个办法加大吧.

如果用port安装的是mysql55版本的话,在执行mysql\_install\_db的时候会提示”FATAL ERROR: Could not find ./bin/my_print_defaults“错误,解决办法:
最后:**在/etc/rc.conf** 添加一行命令：
**mysql_enable = “YES”
** 或者直接在命令行中输入命令:**
echo ‘mysql_enable=”YES”‘ >> /etc/rc.conf
** 使mysql成为一项服务,随机启动,省去手动启动mysql服务了.

重启和停止MySQL的命令:

>  /usr/local/etc/rc.d/mysql-server start|stop|restart

如果以后要修改mysql的数据保存路径,请参考:

**安装APACHE**

> ****#cd /usr/ports/www/apache22
> #make install clean

[![](http://blog.haohtml.com/wp-content/uploads/2009/03/apache_config.jpg)][1]

以上使用ports方式安装的apache默认目录并非大家习惯使用的/usr/local/apache,而是/usr/local/etc/apache22目录,可以使用以下命令指定apache的安装目录:
_**#make PREFIX=/usr/local/apache install,**_

Unix下apache默认的mpm模块用的是prefork,如果更换为worker的话,请编译的时候添加参数:**–with-mpm=worker**,prefork与worker的区别见: [http://blog.haohtml.com/archives/4656](http://blog.haohtml.com/archives/4656)

****详细使用方法点击 [这里](/index.php/archives/802) 查看(另类用法:make deinstall,make clean,make rmconfig,make reinstall FORCE_PKG_REGISTER=”yes”)

**启动APACHE**

> **/usr/local/sbin/httpd -k start**

****查看是否安装成功
配置httpd.conf
**/usr/local/etc/apache22/httpd.conf**
设置根目录
/home/web/blog.haohtml.com
在AddType application/x-gzip .gz .tgz后面加上下面3行：
#php support

> **AddType application/x-httpd-php .php
> AddType application/x-httpd-php-source .phps**

****随后设置网站默认启动页允许为index.php。同样在httpd.conf里编辑，不必退出。找到
DirectoryIndex index.html index.html.var
添加index.php进去，为：
DirectoryIndex index.php index.html index.html.var
还有其它设置，根据需要自行处理
最后:**在/etc/rc.conf** 添加：

> **apache22_enable = “YES”**

****或者直接在命令行中输入:

> **echo ‘apache22_enable=”YES”‘ >> /etc/rc.conf**

****这样服务器启动时，apache就会启动,注意是apache22_enable,这里是两个数字2.

_**要注意的：**_
(在FreeBSD下使用ports安装apache22会出现类似的warming：
No such file or directory: Failed to enable the ‘httpready’ Accept Filter
解决方法是：

> **#kldload accf_http** (有关kldload命令简介 [点击这里查看](/index.php/archives/804))

并将以下语句写入到/boot/defaults/loader.conf中，以便下次启动自动装载模块

> **accf\_data\_load=”YES”
> accf\_http\_load=”YES”**

****这是因为不能启动FreeBSD自带的一个基于http端口过滤的模块。这个模块的作用很不错–检查HTTP请求是否完整，符合规则accpt一个Http进程，否则就扔掉。)

**安装PHP5**

> **#cd /usr/ports/lang/php5
> #make install clean**

> +——————————————————————–+
> |                      Options for php5 5.3.2                        |
> | +—————————————————————-+ |
> | |        [X] CLI        Build CLI version                        | |
> | |        [X] CGI        Build CGI version                        | |
> | |        [X] APACHE     Build Apache module                      | |
> | |        [ ] DEBUG      Enable debug                             | |
> | |        [X] SUHOSIN    Enable Suhosin protection system         | |
> | |        [ ] MULTIBYTE Enable zend multibyte support            | |
> | |        [ ] IPV6       Enable ipv6 support                      | |
> | |        [ ] MAILHEAD   Enable mail header patch                 | |

**#cp /usr/local/etc/php.ini-dist /usr/local/etc/php.ini**
如果提示php.ini-dist,请使用php.ini-production.

修改 /usr/local/etc/php.ini文件,修改date.timezone = PRC,解决php中相差八小时的问题.

安装完毕后,安装扩展

> **cd /usr/ports/lang/php5-extensions/
> make install clean**

根据需要选择插件包，安装过程中要在弹出的对话框中选中**mysql**选项,否则不支持mysql数据库的.建议把mysqli扩展项也选择上,现在用这个扩展的越来越多.如果没有安装请参考:.当然包越多所需要的时间越长，大概需要30分钟.测试安装是否成功 .

注意这里先安装了apache,再安装了php,这样安装完php后,将自动在php.ini文件里添加php模块(LoadModule php5_module        libexec/apache22/libphp5.so)

**安装Zend Optimizer**

> ****cd /usr/ports/devel/ZendOptimizer/
> make install clean
> ===> ZendOptimizer-3.3.0.a cannot install: doesn’t work with PHP version : 5 (Doesn’t support PHP 5).
> \*** Error code 1
>
> Stop in /usr/ports/devel/ZendOptimizer.

注:如果你用的是FreeBsd8.0版本的可能会出现上面的情况,这里可以使用使用pkg_add命令来安装Zend Optimizer.

> **#pkg_add -r ZendOptimizer**
> **#rehash**
> 执行结果将类似如下:
> Fetching [ftp://ftp.freebsd.org/pub/FreeBSD/ports/i386/packages-8.0-release/Latest/ZendOptimizer.tbz…](ftp://ftp.freebsd.org/pub/FreeBSD/ports/i386/packages-8.0-release/Latest/ZendOptimizer.tbz...) Done.
> pkg_add: warning: package ‘ZendOptimizer-3.3.0.a’ requires ‘libxml2-2.7.5’, but ‘libxml2-2.7.7’ is installed
> pkg_add: warning: package ‘ZendOptimizer-3.3.0.a’ requires ‘php5-5.2.11’, but ‘php5-5.3.2’ is installed
>
> \***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\*****
>
> You have installed the ZendOptimizer package.
>
> Edit /usr/local/etc/php.ini and add:
>
> [Zend]
> zend\_optimizer.optimization\_level=15
> zend\_extension\_manager.optimizer=”/usr/local/lib/php/20060613/Optimizer”
> zend\_extension\_manager.optimizer\_ts=”/usr/local/lib/php/20060613/Optimizer\_TS”
> zend_extension=”/usr/local/lib/php/20060613/ZendExtensionManager.so”
> zend\_extension\_ts=”/usr/local/lib/php/20060613/ZendExtensionManager_TS.so”
>
> NOTE: PHP should be compiled in non-debug mode (default).
>
> \***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\*****
> 虽然居然成功了,但也可能用phpinfo时候还是不行的![可惜最后还是不行，得到的教训是，不要用太新的版本，这样资料和环境的支持会很不完善。]

**安装phpMyAdmin**
\# cd /usr/ports/databases/phpmyadmin/
\# make fetch
接下来是一些提示，下载。
#cd /usr/ports/distfiles/
#tar xvf phpMyadmin-2-11.9-languages.bz2 -C /home/web/phpmyadmin
设置一下就可以了

**设置FTP**
[[教程]FreeBSD vsftpd+pam虚拟用户方案配置http://blog.haohtml.com/archives/7213][2]

**安装memcache**(服务端)
1.首先安装memcache，因为是在FreeBSD环境下，所以我们采用最简单的ports方式来安装memcache

> **cd /usr/ports/databases/memcached/
> make install clean**

****ports会自动寻找源进行下载，然后编译安装
安装好memcache以后，编辑/etc/rc.conf文件，
在最后一行加一句

> **memcached_enable=”YES”**

然后保存退出。
memcache会随着开机自动启动，手动启动的命令是：

> **/usr/local/etc/rc.d/memcached start**

好了，现在memcache已经安装并启动完毕了。
2.安装pecl::memcache扩展(客户端)，这是php的扩展，安装以后可以使用Memcache函数库，php手册上有详细的使用法说明。

> **cd /usr/ports/databases/pecl-memcache/
> make install clean**

****安装好以后，会自动在/usr/local/etc/php/extension.ini 加上一行 extension=memcache.so
用命令查看一下：

> cat /usr/local/etc/php/extensions.ini

如果看见最后一行有 extension=memcache.so

说明已经安装好了
这个时候重新启动一下apache server即可
phpinfo()可以看到memcache扩展的信息。

**本文来自ChinaUnix博客，如果查看原文请点：** [http://blog.chinaunix.net/u/13004/showart_1190048.html](http://blog.chinaunix.net/u/13004/showart_1190048.html)

 [1]: /wp-content/uploads/2009/03/apache_config.jpg
 [2]: http://blog.haohtml.com/archives/7213