---
title: 从零开始，安装运行Pushlets所需的Servlet相关环境
author: admin
type: post
date: 2010-09-28T06:57:41+00:00
url: /archives/5853
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - pushlets

---
最近在研究基于WEB的聊天程序，了解了长连接相关技术，从网上看了很多文章，但多数是英文的，这让我很头疼。我准备自己动手测试一下网上的开源comet框架，最后我选择的是pushlet，先将我的一些操作步骤写下来和大家一起分享。

我先去 [pushlet官方网站](http://www.pushlets.com/ "pushlets") 下载了相关代码，我选择的是5-feb-2010: v2.0.4 released版本的（下载地址：）。下载了后我解压打开相关文件，里面只有几个文件夹和文件，找了一下，DOC文件夹里面有说明文档，打开index.html，是英文的。按照左侧导航大概浏览一下，最后跳到install上面，硬着头皮打开浏览一下，大概意思是需要Servlet引擎环境。于是开始在Google上面找相关的文章。看了一下大概意思明白，就是需要安装Tomcat。

去Tomcat官方网站下载，我下载的是Tomcat 7.0.2 Released版本的（下载地址：）。下载解压出来是一个没有扩展名的文件：apache-tomcat-7.0.2-windows-x86，想到Windows下面的文件应该是EXE的，我将其扩展名改为exe运行，结果闪了一个命令窗口就没有了，在文件上面点右键，发现有解压的选项，于是解压到当前目录。解压出来是一个同名文件夹，里面有个RUNNING.txt的文件，我又一次阅读了相关英文说明，大概意思是需要先安装J2SE的环境。里面有相关下载J2SE的链接，按照里面说的步骤去下载（这里有两个文件都可以下载JRE和JDK，我两个都下载了，看他的说明文档意思是JDK要好些，我就直接安装的是JDK，下载地址：），下载并按照提示进行安装，我是按默认设置安装的。安装完毕后，我以为就是所谓的JAVA环境了，去运行Tomcat下面的bin文件夹下shutdown.bat，结果命令窗口又是一闪而过，实际上他的说明文档有句话我没有读懂：

原文：
_In this
case set your JAVA_HOME environment variable to the pathname of
the directory into which you installed the JDK, e.g. c:\j2sdk6.0
or /usr/local/java/j2sdk6.0._

实际上这里是比较关键的，具体操作：

桌面上 “我的电脑” 点击右键 “属性”>“高级”>“环境变量”>“系统变量”>新建一个

变量名：JAVA_HOME

变量值：C:\Program Files\Java\jdk1.6.0_21（你刚才安装JDK的设置目录）

确定提交就OK了。

现在再去运行Tomcat下面的bin文件夹下shutdown.bat，弹出一个名为Tomcat命令窗口说明OK。在浏览器里面输入http://localhost:8080/ 回车，打开页面有下面的文字就说明你的安装成功了：

If you’re seeing this page via a web browser, it means you’ve setup Tomcat successfully. Congratulations!

现在我的Servlet环境搭建好了，就开始继续着手研究Pushlets了，再去读Pushlets相关说明文档，里面说了有相关的例子，我将刚才的pushlet-2.0.4文件夹全部复制到Tomcat下面的webapps\ROOT\下面。在浏览器里面输入：http://localhost:8080/pushlet-2.0.4/webapps/pushlet/打开的页面就用Examples的链接，点击basics，打开页面有一个表格，再点击Run，弹出的就是他给的简单例子。点击Source下面的链接就可以看到刚才演示例子的源文件了。

到此，我已完成了初步对Pushlets的简单例子测试，下来后我会再根据他的这些例子进行简单修改和研究，争取自己能够写一些东西出来。

**相关文章及资源：**
1. Comet：基于 HTTP 长连接的“服务器推”技术 [http://www.ibm.com/developerworks/cn/web/wa-lo-comet/](http://www.ibm.com/developerworks/cn/web/wa-lo-comet/)
2. “pushlet”：开源 comet 框架
3. Apache Tomcat
4. Java [http://www.oracle.com/technetwork/java/javase/overview/index.html](http://www.oracle.com/technetwork/java/javase/overview/index.html)

来源: