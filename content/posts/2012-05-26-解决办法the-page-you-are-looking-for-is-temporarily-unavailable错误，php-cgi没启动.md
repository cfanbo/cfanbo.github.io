---
title: 解决办法The page you are looking for is temporarily unavailable错误，php-cgi没启动
author: admin
type: post
date: 2012-05-26T12:59:18+00:00
url: /archives/13045
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - php-cgi

---
今天访问WordPress程序做的网站，突然出现**The page you are looking for is temporarily unavailable**错误，服务器环境为： Linux+Nginx+MySQL+PHP。于是上网查找解决方法，找到以下两个解决方法，作为参考：

**  解决方法一：**

访问discuz论坛很正常，但是一旦访问uc_server的后台就这样nginx就提示以下错误：

> The page you are looking for is temporarily unavailable.
> Please try again later.

1.先检查PHP FastCGI进程数是否够用：

netstat -anpo|grep “php-cgi”|wc -l
如果输出为0的话，则表示FastCGI 进程数够大，可通过修改php-fpm.conf或者使用 [http://blog.haohtml.com/archives/5530](http://blog.haohtml.com/archives/5530) 介绍的命令修改

2.此时则修改scgi_params文件，找到：

**scgi_param SCGI 1;**

把它改为：

**scgi_param SCGI 5;**

3.PHP程序如果的执行时间超过了Nginx的等待时间，就可适当地增加nginx.conf配置文件中FastCGI的timeout时间，例如：

http
{
……
**fastcgi\_connect\_timeout 300;
fastcgi\_send\_timeout 300;
fastcgi\_read\_timeout 300;
fastcgi\_buffer\_size 64k;
fastcgi_buffers 4 64k;**
……
}

4.重启FastCGI

先杀掉进程：# pkill -9 php-cgi
然后重启：# /usr/local/bin/spawn-fcgi -a 127.0.0.1 -p 9000 -u www -g www -f /usr/local/bin/php-cgi

5.重启Nginx

先杀掉进程:# killall -9 nginx
然后重启：# /usr/local/sbin/nginx



其它可能情况：

1)访问任意PHP文件，出现

The page you are looking for is temporarily unavailable.
Please try again later.

2)访问html页面，正常

原因:
nginx不能正常通过FastCGI结果访问PHP

1)如果是以tcp socket形式，可能是进程用户权限设置得不对

spawn-fcgi -a 127.0.0.1 -p 9000 -C 2 -u www-data -g www-data -f /usr/bin/php-cgi

可以改为 www-data 或者 nobody, 重启php-cgi进程

比如，这样写也可以：

**/usr/bin/spawn-fcgi  -a 127.0.0.1 -p 9000 -C 2 -u nobody -g nobody -f /usr/bin/php-cgi**

**（  nobody  是Linux系统中内置的匿名账号 ）**

**
**

2)如果是unix socket,可能 socket文件权限没有写入能力

srwxrwxr-x 1 gavin gavin 0 11-12 10:18 php-fcgi.sock

为其他用户添加写入能力

chmod o+w php-fcgi.sock



———————————————————————————-



**解决方法二：**

今天网站突然出现如下错误：

The page you are looking for is temporarily unavailable.Please try again later.

很奇怪，我对服务器端的技术不是很熟悉，于是查询了下google，在https://wiki.archlinux.org/index.php/Nginx

上面的解决方法：

Error: The page you are looking for is temporarily unavailable. Please try again later.

This is because the FastCGI server has not been started.

如何解决呢？

刚开始我怀疑是不是nginx挂了，我首先通过**ps aux | grep nginx**，结果出现：

root      3769  0.0  0.0   5760   692 ?        Ss   Apr21   0:00 nginx: master process /usr/local/nginx/sbin/nginx

www       3770  0.0  0.1  18680 14252 ?        S    Apr21   0:03 nginx: worker process

www       3771  0.0  0.1  18680 14252 ?        S    Apr21   0:03 nginx: worker process

www       3772  0.0  0.1  18712 14276 ?        S    Apr21   0:03 nginx: worker process

www       3774  0.0  0.1  18680 14248 ?        S    Apr21   0:03 nginx: worker process

www       3776  0.0  0.1  18712 14240 ?        S    Apr21   0:03 nginx: worker process

www       3777  0.0  0.1  18680 14252 ?        S    Apr21   0:03 nginx: worker process

www       3778  0.0  0.1  18680 14232 ?        S    Apr21   0:02 nginx: worker process

root     24068  0.0  0.0   5196   756 pts/1    S+   14:33   0:00 grep nginx



可见nginx是正常的，本来打算重启nginx的：

/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf的，



突然觉得有没有其他方法，有同事提示先在一个目录下运行下test.html和test.php，结果html可以运行，php无法运行。

证实是php没有启动，我刚才也检测过php的进程，的确是没有php进程，这台服务器我不熟悉，同事帮忙查看了下

cd /etc/init.d，就是web管理员经常看的地方，是随着系统自动启动的服务，程序等。可以看看：

http://blog.wgzhao.com/2008/12/27/talk-about-rc-local.html的《 说说？**/etc/rc.d/rc.local **》

找到：

**/usr/local/php/sbin/php-fpm start  **，首先什么是php-fpm呢？

就是FastCGI Process Manager，是一种可选的PHP FastGCI执行模式，有一点很有特点的应用，尤其是一个繁忙的网站中：

(1)可适应的进行再生(NEW!)

(2)基本的统计功能（Apache’s mod_status)

(3)高级进程管理功能，能够优雅的停止/开始

(4)能够使用不同的工作用户和不同的php.ini

(5)输入，输出日志记录…



开启后，一切恢复正常！自己的服务器端技术还是有很多地方使用的不够。需要多学习使用！