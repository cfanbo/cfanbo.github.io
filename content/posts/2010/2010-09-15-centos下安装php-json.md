---
title: centos下安装php-json
author: admin
type: post
date: 2010-09-15T09:09:22+00:00
url: /archives/5688
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - json

---
centos5下面yum源中没有json，只能通过编译了。

#cd /usr/local/src
1.下载源文件包：
wget

2.解压
tar xvjf php-json-ext-1.2.0.tar.bz2
使用tar命令解压一定要确认已经安装过bzip2，否则会提示 “tar: bzip2: Cannot exec: No such file or directory” 错误.

3.进入目录
cd php-json-ext-1.2.0
4.初始化PHP环境
phpize

> 如果报错了：phpize commend not found
>
> 需要安装phpize
> 这个可以在yum中安装
> yum -y install php-devel
>
> 如果还不行，说明你的编译工具有问题，安装一下就可以了
> yum -y install autoconf
> yum -y install automake
> yum -y install libtool
> 运行phpize
> (成功了)

5../configure

6.make

7.makeinstall

8.查看有没有安装成功
find / -name ‘*json.so’

./usr/lib/php/modules/json.so
说明已经有了

9.修改php.ini
我的是在php.ini 中include一个文件夹 ／etc/php.d

在这个文件中添一个json.ini
vim json.ini
内容如下:
extension=json.so

10.重启服务

11.phpinfo()中您将看到

 json support

 enabled

 json version

 1.2.0