---
title: centos下更新Python版本的步骤
author: admin
type: post
date: 2013-06-03T14:57:26+00:00
url: /archives/13959
categories:
 - 服务器
tags:
 - python

---
准备安装gitlab,发现系统目前的python版本为2.4.3版本.太低了, 虽然目前最高版本为3.3.0版本.但gitlab不支持这个版本.没有办法,我们这里将python升级到2.7.6版本.

更新python千万不要把老版本的删除！新老版本是可以共存的，很多基本的命令、软件包都要依赖预装的老版本python的，比如yum。


**第1步：更新gcc，因为gcc版本太老会导致新版本python包编译不成功**

[shell]yum -y install gcc[/shell]

系统会自动下载并安装或更新，等它自己结束

**第2步：下载Python 2.7.0软件包**

[shell]wget http://python.org/ftp/python/2.7/Python-2.7.tar.bz2
tar -jxvf Python-2.7.tar.bz2
cd Python-2.7
./configure
make all
make install
make clean
make distclean
/usr/local/bin/python2.7 -V
cd ../[/shell]

编译安装完毕以后，可以输入上面一行命令，查看版本显示 Python 2.7

**第4步：建立软连接指向到当前系统默认python命令的bin目录**，让系统使用新版本python
#mv /usr/bin/python /usr/bin/python2.4 //当前python的版本为2.4所以是python2.4
#ln -s /usr/local/bin/python2.7 /usr/bin/python
输入#python -V，即可查看当前默认python版本
默认的python成功指向2.7以后，yum不能正常使用，需要修改yum的配置文件

**第5步：修改yum配置文件
**
#vi /usr/bin/yum
把文件头部的#!/usr/bin/python改成#!/usr/bin/python2.4 //改为之前的老版本号
保存退出，yum即可正常使用。如若有其他命令、软件不能正常使用，仿照yum配置文件的修改方法，修改其配置文件即可。
至此，更新完毕。