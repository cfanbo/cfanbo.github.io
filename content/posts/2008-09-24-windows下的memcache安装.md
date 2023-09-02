---
title: Windows下的Memcache安装
author: admin
type: post
date: 2008-09-24T15:52:00+00:00
excerpt: |
 Windows下的Memcache安装

 很多phper不知道如何在Windows下搭建Memcache的开发调试环境，最近个人也在研究Memcache，记录下自己安装搭建的过程
 其实我开始研究Memcache的时候并不知道居然还有memcached for Win32这个鸟东西，害得我在CnetOS下折腾1天才搞定，今天突然发现Windows下的Memcache进行开发调试完全没有问题，所以写篇Memcache的文档分享给大家
 Windows下的Memcache安装：
 1. 下载memcache的windows稳定版，解压放某个盘下面，比如在c:\memcached
 2. 在终端（也即cmd命令界面）下输入 c:\memcached\memcached.exe -d install 安装
 3. 再输入： c:\memcached\memcached.exe -d start 启动NOTE: 以后memcached将作为windows的一个服务每次开机时自动启动这样服务器端已经安装完毕了
 4.下载php_memcache.dll，请自己查找对应的php版本的文件
 5. 在C:\winnt\php.ini 加入一行 extension=php_memcache.dll
 6.重新启动Apache，然后查看一下phpinfo，如果有memcache，那么就说明安装成功！
url: /archives/402
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - memcache

---
## Windows下的Memcache安装

不用说了，这便是 [memcached](http://www.oschina.net/project/355) 在Windows系统下的版本。[（点击这里下载memcached for win32）](http://blog.haohtml.com/wp-content/uploads/2008/09/memcached_win.zip)

**Windows下的Memcache安装**：

1. 下载 [memcache](http://jehiah.cz/projects/memcached-win32) 的windows稳定版，解压放某个盘下面，比如在c:\memcached

2. 在终端（也即cmd命令界面）下输入 ‘c:\memcached\memcached.exe -d install’ 安装

3. 再输入： ‘c:\memcached\memcached.exe -d start’ 启动。NOTE: 以后memcached将作为windows的一个服务每次开机时自动启动。这样服务器端已经安装完毕了。

4.下载 [php_memcache.dll](http://pecl4win.php.net/list.php)，请自己查找对应的php版本的文件

5. 在C:\windows\php.ini 加入一行 ‘extension=php_memcache.dll’

6.重新启动Apache，然后查看一下phpinfo，如果有memcache，那么就说明安装成功！

7.如果要卸载的话,可以执行c:\memcached\memcached.exe -d install 即可.


**memcached的基本设置**：

[blockquote]-p 监听的端口

 -l 连接的IP地址, 默认是本机

 -d start 启动memcached服务

 -d restart 重起memcached服务

 -d stop|shutdown 关闭正在运行的memcached服务

 -d install 安装memcached服务

 -d uninstall 卸载memcached服务

 -u 以的身份运行 (仅在以root运行的时候有效)

 -m 最大内存使用，单位MB默认64MB

 -M 内存耗尽时返回错误，而不是删除项

 -c 最大同时连接数，默认是1024

 -f 块大小增长因子，默认是1.25

 -n 最小分配空间，key+value+flags默认是48

 -h 显示帮助


**Memcache环境测试**：

运行下面的php文件，如果有输出This is a test!，就表示环境搭建成功开始领略Memcache的魅力把！


> < ?php
>
> $mem = new Memcache;
>
> $mem->connect(127.0.0.1, 11211);
>
> $mem->set(key, This is a test!, 0, 60);
>
> $val = $mem->get(key);
>
> echo $val;
>
> ?>