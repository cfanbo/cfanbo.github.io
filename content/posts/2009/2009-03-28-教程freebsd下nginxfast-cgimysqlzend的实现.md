---
title: '[教程]FreeBSD下nginx+fast-cgi+mysql+zend的实现（php-fpm和spawn-fcgi）'
author: admin
type: post
date: 2009-03-28T05:03:04+00:00
excerpt: |
 首先在安装所有软件之前新系统ports，方法如上一贴

 然后 再进行下面的工作

 1）安装mysql
 #cd /usr/ports/databases/mysql51-server
 #make WITH_CHARSET=uft8(我选择了这个，情况自己定) WITH_XCHARSET=all install clean
 #cp /usr/local/share/mysql/my-medium.cnf /etc/my.cnf
 #rehash
 !!!—–WITH_CHARSET=utf8(我选择了这个，情况自己定，可以使用gbk)
 # mysql_install_db –user=mysql ##初始化mysql
 #/usr/local/bin/mysqld_safe & ##启动mysql
 #/usr/local/bin/mysqladmin -u root password ‘newpass’ ##修改root密码，newpass是你需要改的密码
url: /archives/1133
IM_contentdowned:
 - 1
categories:
 - MySQL
 - 服务器
tags:
 - fastcgi
 - mysql
 - nginx

---
另一篇文章是用php-fpm方式安装的,用的人也比较的多,推荐使用,这里介绍的是用fastcgi方式安装的.

首先在安装所有软件之前新系统ports，然后 再进行下面的工作

**1）安装mysql****#cd /usr/ports/databases/mysql51-server**

**#make WITH\_CHARSET=gbk WITH\_XCHARSET=all ** ****WITH\_PROC\_SCOPE\_PTH=yes SKIP\_DNS\_CHECK=yes BUILD\_OPTIMIZED=yes** install clean** //(utf8我选择了这个，情况自己定)

**#cp /usr/local/share/mysql/my-medium.cnf /etc/my.cnf
#rehash**
!!!—–WITH_CHARSET=utf8(我选择了这个，情况自己定，可以使用gbk)
**\# mysql\_install\_db** ##初始化mysql,如果在命令行后面添加上 –user=mysql 的话，会失败，不清楚什么原因

#**chown -R mysql:mysql /var/db/mysql** ##目录权限修改
**#/usr/local/bin/mysqld_safe &** ##启动mysql
**#/usr/local/bin/mysqladmin -u root password ‘newpass’** ##修改root密码，newpass是你需要改的密码

最后:**在/etc/rc.conf** 添加一行命令：
**mysql_enable = “YES”
** 或者直接在命令行中输入命令:**
echo ‘ mysql_enable = “YES” ‘ >> /etc/rc.conf
** 使mysql成为一项服务,随机启动,省去手动启动mysql服务了.

注意如果安装的是mysql5.5的话，可能会在初始化表的时候提示＂ATAL ERROR: Could not find ./bin/my_print_defaults＂，**解决办法是**：

/usr/local/bin/mysql_install_db –user=mysql –basedir=/usr/local/ –datadir=/var/db/mysql &

**2）安装php
\# cd /usr/ports/lang/php5
\# make config**
[X] CLI        Build CLI version
[X] CGI        Build CGI version
[ ] APACHE     Build Apache module   //不安装这个
[ ] DEBUG      Enable debug
[X]] SUHOSIN Enable Suhosin protection system
[X] MULTIBYTE Enable zend multibyte support
[ ] IPV6       Enable ipv6 support
[ ] REDIRECT   Enable force-cgi-redirect support (CGI only)
[ ] DISCARD    Enable discard-path support (CGI only)
[X] FASTCGI    Enable fastcgi support (CGI only)
[X] FPM        Enable fastcgi process manager (CGI only)
[X] PATHINFO   Enable path-info-check support (CGI only)
**\# make install clean**
 **#cp /usr/local/etc/php.ini-dist /usr/local/etc/php.ini**
注意：上面“[X]FPM …” 这一项在以php-fpm方式运行nginx运行php脚本的时候才选择的。

如果没有php.ini-dist，也可以直接复制php.ini-production，对于php5.3.6用的是这个文件．

**3）安装php5-extensions
\# cd /usr/ports/lang/php5-extensions/
\# make config**
Options for php5-extensions 1.0
————————————————-
[X] FTP        FTP support
[X] GD
[X] GETTEXT
[X] MBSTRING
[X] MYSQL
[ ] POSIX //去掉.
[ ] SQLITE //去掉.
[X] ZLIB

**\# make install clean**

**#rehash**
**4)安装Zend Optimizer(PHP5.3好像不支持的)
**

对于php5.3,可以用安装eaccelerator.参考: [http://blog.haohtml.com/archives/9718](http://blog.haohtml.com/archives/9718)

**#cd /usr/ports/devel/ZendOptimizer/**



**方法一:**

>

> #make install clean
>

然后在php.ini文件最后加上下面的代码:

>

> [Zend]
>

>
>

> zend_optimizer.optimization_level=15
>

>
>

> zend_extension_manager.optimizer=”/usr/local/lib/php/20060613/Optimizer”
>

>
>

> zend_extension_manager.optimizer_ts=”/usr/local/lib/php/20060613/Optimizer_TS”
>

>
>

> zend_extension=”/usr/local/lib/php/20060613/ZendExtensionManager.so”
>

>
>

> zend_extension_ts=”/usr/local/lib/php/20060613/ZendExtensionManager_TS.so”
>

**方法二:**

#make #不要安装，只需要下载解包
**#cd /usr/ports/devel/ZendOptimizer/work/ZendOptimizer-*
#./install-tty** ##会出现一个文字的安装界面，只是最后一步，不要选择apache就可以了
**#vi /usr/local/etc/php.ini** #插入zend的路径，一般来说，上面的安装会自动加入下面的文字。

[Zend]
zend\_extension\_manager.optimizer=/usr/local/Zend/lib/Optimizer-3.3.0
zend\_extension\_manager.optimizer\_ts=/usr/local/Zend/lib/Optimizer\_TS-3.3.0
zend_optimizer.version=3.3.0a
zend_extension=/usr/local/Zend/lib/ZendExtensionManager.so
zend\_extension\_ts=/usr/local/Zend/lib/ZendExtensionManager_TS.so

**5)安装nginx**

 **#cd /usr/ports/www/nginx/
#make install clean
#rehash**
**6)安装lighttpd（只针对spawn-fcgi方式,fpm直接跳过）
#cd /usr/ports/www/lighttpd/
#make install clean
#rehash**

**nginx+mysql开机后自动运行
#cat>>/etc/rc.conf
mysql_enable=”YES”
nginx_enable=”YES”**
#
**7)配置nginx
** 编辑nginx配置文件_/usr/local/etc/nginx/nginx.conf
_ **#user   nobody**
删除前面的注释#，改成 user   www

location / {
root    /usr/local/www/nginx;
index    index.html index.htm;
}
在index.html前面添加一个index.php
location / {
root    /usr/local/www/nginx;
index    index.php index.html index.htm;
}

#location ~ \.php$ {
#    fastcgi_pass    127.0.0.1:9000;
#           fastcgi_index   index.php;
#           fastcgi\_param     SCRIPT\_FILENAME     /scripts$fastcgi_script.name;
#    include      fastcgi_params;
#}
将前面的#去掉，修改为
location ~ \.php$ {
fastcgi_pass    127.0.0.1:9000;
fastcgi_index   index.php;
fastcgi\_param     SCRIPT\_FILENAME     /usr/local/www/nginx$fastcgi_script.name;
include      fastcgi_params;
}

#为了使SCRIPT\_FILENAME 有效，更改php.ini里面的cgi.fix\_pathinfo=1；

**_这个地方非常重要，如果红色部分/usr/local/www/nginx不配置的话，如果执行php文件，就会出现No input file specified 错误提示。这个在网上查了半天才找到解决办法。切记切记！！_**

注意，到此只是安装了nginx，但还不支持php脚本，下面的两种办法可以实现此功能。

**8.配置fastcgi,支持PHP**

这里有两种选择,一种是spawn-fcgi,另一个种php-fpm.对于spawn-fcgi,目前FreeBSD新版本系统中已经有了spawn-fcgi包,不再依赖lighttpd软件.

如果您的系统里提示没有这个命令,很可能是lightttpd的版本高于ighttpd-1.4.15这个,最新版本已经将spawn-fcgi单独作为一个项目使用.目前FreeBSD已经有了自己的spawn-fcgi了,安装方式如下:

> #cd /usr/ports/www/spawn-fcgi
> #make install clean

**8.1）配置spawn-fcgi支持PHP(方式一)**

/usr/local/bin/spawn-fcgi -a 127.0.0.1 -p 9000 -u www -f /usr/local/bin/php-cgi
参数说明:
-f 指定调用FastCGI的进程的执行程序位置，根据系统上所装的PHP的情况具体设置
-a 绑定到地址addr
-p 绑定到端口port
-s 绑定到unix socket的路径path
-C 指定产生的FastCGI的进程数，默认为5（仅用于PHP）
-P 指定产生的进程的PID文件路径
-u和-g FastCGI使用什么身份（-u 用户 -g 用户组）运行，Ubuntu下可以使用www-data，其他的根据情况配置，如nobody、apache等

/usr/local/bin/spawn-fcgi -a 127.0.0.1 -p 9000 -u www -f /usr/local/bin/php-cgi

或直接让它设置成开机服务启动:

**#vi/etc/rc.local**

/usr/local/bin/spawn-fcgi -a 127.0.0.1 -p 9000 -u www -g www -C 25(进程数) -f /usr/local/bin/php-cgi

或者修改/etc/rc.conf配置文件.添加这行

**spawn_fcgi_enable=”YES”**

这样spawn-fcgi就能开机自启动了.

**8.2、使用php-fpm支持php(方式二)**

在安装php的时候只要选中php-fpm这个模块即可。

安装完以后执行以下操作也可以,,安装完以后需要在/etc/rc.conf中加入 **php_fpm_enable=”YES”**。

>

> #echo ‘php_fpm_enable=”YES”‘ >> /etc/rc.conf

#/usr/local/etc/rc.d/php-fpm start
>

见: [http://blog.haohtml.com/index.php/archives/5530](http://blog.haohtml.com/index.php/archives/5530)

_**9)启动Nginx服务**_

#nginx


完成了，测试了phpinfo() ,前面由于 **_No input file specified_** 错误弄了很久啊


现在搞定了，上个图吧


[![](http://blog.haohtml.com/wp-content/uploads/2009/03/nginx_ps-aux3.jpg)](/wp-content/uploads/2009/03/nginx_ps-aux3.jpg)

还有一个啊


[![](http://blog.haohtml.com/wp-content/uploads/2009/03/nginx_phpinfo.jpg)](/wp-content/uploads/2009/03/nginx_phpinfo.jpg)

第一次安装 ，也是在网上找了很多教程，也参考了他们的成果，多谢了：）


如果有多台虚拟主机的话，可以参考这里的配置, [点击查看](/index.php/archives/1222)

如果要配置nginx启用SSI支持的话,打开nginx 的配置文件nginx.conf，在里面的http里添加下面几行配置


> ssi on;
>
> ssi_silent_errors on;
>
> ssi_types text/shtml;

如果需要安装Memcached的话，请参考这篇文章： [http://blog.haohtml.com/index.php/archives/435](http://blog.haohtml.com/index.php/archives/435),只要执行一下


> #/usr/local/etc/rc.d/php-fpm reload

命令即可，不用重启nginx的。


**安全：**

[解决Nginx文件类型错误解析漏洞的方法](http://blog.haohtml.com/index.php/archives/7165)

[Nginx禁止通过IP,未绑定域名访问服务器](http://blog.haohtml.com/index.php/archives/6885)

[PHP安全配置](http://blog.haohtml.com/index.php/archives/3114)

[PHP安全配置详解](http://blog.haohtml.com/index.php/archives/3116)

**技巧：**

[用include指令实现nginx多虚拟主机配置](http://blog.haohtml.com/index.php/archives/6203)

[nginx上如何支持.htaccess伪静态转向](http://blog.haohtml.com/index.php/archives/7251)

**优化:**

[nginx下对域名进行301重定向-优化篇](http://blog.haohtml.com/index.php/archives/7526)(haohtml.com => www.haohtml.com)