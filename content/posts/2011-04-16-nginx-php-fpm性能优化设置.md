---
title: nginx php-fpm性能优化设置
author: admin
type: post
date: 2011-04-16T17:18:49+00:00
url: /archives/9280
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx
 - php-fpm

---
When you running a highload website with PHP-FPM via FastCGI, the following tips may be useful to you : )
如果您高负载网站使用PHP-FPM管 理FastCGI，这些技巧也许对您有用：)
1. Compile PHP’s modules as less as possible, the simple the best (fast);
1.尽量少安装PHP模块，最简单是最好（快）的
2. Increas PHP FastCGI child number to 100 and even more. Sometime, 200 is OK! ( On 4GB memory server);
2.把您的PHP FastCGI子进程数调到100或以上，在4G内存的服务器上200就可以


注：我的1g测试机，开64个是最好的，建议使用压力测试获取最佳值

[![](http://blog.haohtml.com/wp-content/uploads/2011/04/nginx_php-fpm.png)](http://blog.haohtml.com/wp-content/uploads/2011/04/nginx_php-fpm.png)

3. Using SOCKET PHP FastCGI, and put into /dev/shm on Linux;
3.使用socket连接FastCGI，linux操作系统可以放在 /dev/shm中注： 在php-fpm.cnf 里设置/tmp/nginx.socket就可以通过socket连接 FastCGI了，/dev/shm是内存文件系统，放在内存中肯定会快了

4. Increase Linux “max open files”, using the following command (must be root):
\# echo ‘ulimit -HSn 65536′ >> /etc/profile
\# echo ‘ulimit -HSn 65536 >> /etc/rc.local
\# source /etc/profile
4.调高linux内核打开文件数量，可以使用这些命令(必须是root帐号)echo ‘ulimit -HSn 65536’ >> /etc/profileecho ‘ulimit -HSn 65536’ >> /etc/rc.localsource /etc/profile注：我是修改/etc/rc.local，加入ulimit -SHn 51200的
5. Increase PHP-FPM open file description rlimit:
\# vi /path/to/php-fpm.conf
Find “1024”
Change 1024 to 4096 or higher number.
Restart PHP-FPM.
5. 增加 PHP-FPM 打开文件描述符的限制:# vi /path/to/php-fpm.conf找到“1024”把1024 更改为 4096 或者更高.重启 PHP-FPM.
6. Using PHP code accelerator, e.g eAccelerator, XCache. And set “cache_dir” to /dev/shm on Linux.
6.使用php代码加速器，例如 eAccelerator, XCache.在linux平台上可以把`cache_dir`指向 /dev/shma

**推荐:**
Nginx优化配置