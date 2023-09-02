---
title: CentOS下配置Java环境JDK
author: admin
type: post
date: 2011-06-13T03:20:30+00:00
url: /archives/9765
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - jdk

---
**第一步：**查看Linux自带的JDK是否已安装 （卸载centOS已安装的1.4）

> <1># rpm -qa|grep jdk                ← 查看jdk的信息或直接执行
> 或
> \# rpm -q jdk
>
> 或
> \# java -version
> \# rpm -qa | grep gcj                ← 确认gcj的版本号
> \# yum -y remove java-1.4.2-gcj-compat        ← 卸载gcj

**第二步：安装JDK**
_<1>从SUN下载jdk-1_5_0_14-linux-i586-rpm.bin或jdk-1_5_0_14-linux-i586.bin_

jdk1.6的下载地址:
在/usr下新建java文件夹，将安装包放在/usr/java目录下

> \# mkdir /usr/java

_<2>安装JDK_
\# cd /usr/java

> ①jdk-1\_5\_0_14-linux-i586-rpm.bin文件安装
> \# chmod 777 jdk-1\_5\_0_14-linux-i586-rpm.bin    ← 修改为可执行
> \# ./jdk-1\_5\_0_14-linux-i586-rpm.bin        ← 选择yes同意上面的协议
> \# rpm -ivh jdk-1\_5\_0_14-linux-i586.rpm        ← 选择yes直到安装完毕
>
> ②jdk-1\_5\_0_14-linux-i586.bin文件安装
> \# chmod a+x jdk-1\_5\_0_14-linux-i586.bin         ← 使当前用户拥有执行权限
> \# ./jdk-1\_5\_0_14-linux-i586.bin            ← 选择yes直到安装完毕

**第三步：配置环境变量**
<1># vi /etc/profile
<2>在最后加入以下几行：

> export JAVA\_HOME=/usr/java/jdk-1\_5\_0\_14
> export CLASSPATH=.:$JAVA\_HOME/jre/lib/rt.jar:$JAVA\_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
> export PATH=$PATH:$JAVA_HOME/bin

<3>执行如下命令使环境变量生效

> source /etc/profile

我参考上面的教程用的是jdk1.6,最后安装成功!

[![](http://blog.haohtml.com/wp-content/uploads/2011/06/centos-jdk1.6.png)][1]

 [1]: http://blog.haohtml.com/wp-content/uploads/2011/06/centos-jdk1.6.png