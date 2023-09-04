---
title: 'windows下配置jdk环境[原创]'
author: admin
type: post
date: 2010-05-05T06:00:14+00:00
url: /archives/3558
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - jdk

---
我个人安装的为jdk1.6.0_14版本，安装在d:盘的program fiiles目录里了,直接在安装的时候把c:修改成d:就可以了，呵呵

**第一步：**下载JDK6，从官网下载(),如果下载的版不是1.6.0.14版本的,请根据情况修改下面的路径,这里提供另一个版本的下载地址: [http://www.newhua.com/softdown/61003_2.htm](http://www.newhua.com/softdown/61003_2.htm)

[![](https://blogstatic.haohtml.com//uploads/2023/09/jdk_install.jpg)][1]

**第二步：**安装JDK6,修改默认盘符C：为D:，具体按步骤进行就OK了

**第三步：**配置环境变量，主要是三个：JAVA_HOME、PATH、CLASSPATH。

以下几个变量为系统变量，默认的PATH变量已经存在，所以只需要新建JAVA_HOME和CLASSPATH两个变量就可以了
JAVA_HOME: **d:\Program Files\Java\jdk1.6.0_14
** CLASSPATH变量值为 **.;%JAVA\_HOME%\lib;%JAVA\_HOME%\lib\tools.jar;
** 修改PATH:变量值为 **;%JAVA_HOME%\bin;** //此变量已经存在，直接编辑在最后面添加上这行就可以了

以上环境变量没有的则新建添加，已经有的则编辑增加相应的值。

这里一定要注意是”系统变量”，不是”用户变量”，有些变量值后面有;而有的没有;号

好了，环境变量已经添加好了，现在我们检测是否配置成功的，在“命令”里输入”command“,在命令提示符下输入“java -version“命令，如果输出jdk的版本信息的话，则说明我们配置成功了，否则请根据上面的步骤检查操作是否有误.

默认情况下需要重启电脑,才可以使系统变量生效
