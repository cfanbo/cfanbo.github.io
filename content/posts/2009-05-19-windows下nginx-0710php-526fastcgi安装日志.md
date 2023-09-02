---
title: windows下nginx-0.7.10+php-5.2.6+fastcgi安装日志
author: admin
type: post
date: 2009-05-19T01:26:05+00:00
excerpt: |
 最近心里有点痒，打算怀下旧，搞个php玩玩。找了几台服务器想装个php，虽然是举手之劳，但是总觉得有点不方便。另外家里的宽带总被占线，所以在服务器上做测试那也比较痛苦。

 所以就想在本机弄个，记得以前有个apache php mysql的整合安装版，这可是个好东西，如果有人问我怎么在windows装php啊，我顺口就告诉他找这个，的确可以省不少力气。

 不过今天我就不想用这古老的玩意装机了，虽然这东西装得是快，不过我已经不怎么记得起apache的配置怎么写，甚至有点厌恶写那配置。
url: /archives/1409
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx

---
最近心里有点痒，打算怀下旧，搞个php玩玩。找了几台服务器想装个php，虽然是举手之劳，但是总觉得有点不方便。另外家里的宽带总被占线，所以在服务器上做测试那也比较痛苦。

所以就想在本机弄个，记得以前有个apache php mysql的整合安装版，这可是个好东西，如果有人问我怎么在windows装php啊，我顺口就告诉他找这个，的确可以省不少力气。

不过今天我就不想用这古老的玩意装机了，虽然这东西装得是快，不过我已经不怎么记得起apache的配置怎么写，甚至有点厌恶写那配置。

于是我下了个nginx的windows版，然后再找个php的windows版，在http://www.kevinworthington.com/category/computers/nginx/，下完后先装nginx，没什么复杂，启动，在浏览器输入http://127.0.0.1，没有反应，看一下netstat -an，发现貌似80端口存在，估计这个端口不是nginx占用的，于是关掉所有开启的程序，再启动，再刷新，就可以看到有一个测试页出现了。

这样算是弄完了一个东西，接下来装php吧，先把它解压到d盘，放在d:\php-5.2.6-win32这个目录，有点冗长，不过算了，现在老了也不想对这种太过纠缠。接下来思维就有点短路了，因为windows上哪去找个什么spawn-fcgi来用啊，没办法，google一下。

打开了十几个网页，还是在一篇介绍lighttpd的文章里找到了办法，lighttpd对nginx还是大有帮助的，nginx安装php的办法，linux和windows下都是lighttpd先有，nginx在后面照着抄就可以了，他们所使用的思路都是一样的。那篇lighttpd的文章看来是提供的一份完整解决方案，我从里面揪出了一句话：

php-cgi -b 127.0.0.1:521

用这行小命令就可以启动php-cgi进程和端口了。不过运行时这个dos窗口不会关闭，也挺碍眼的，所以根据该文所述，用一个RunHiddenConsole.exe来启动，就可以了，不知道怎么关闭？很笨的说，ctrl alt del，杀死php-cgi进程就可以了。

最后贴一份nginx的php配置文件，小改一下端口，写一个phpinfo页，运行正常。

—————————————————————–

nginx conf:

server {
listen 127.0.0.1:80;
server_name localhost;

location / {
root d:/php/ ;
}

location ~ .php$ {
fastcgi_pass   127.0.0.1:521;
fastcgi_index  index.php;
fastcgi\_param  SCRIPT\_FILENAME  d:/php$fastcgi\_script\_name;
include        fastcgi_params;
}
}

—————————————————————–

我的php解压在d:\php-5.2.6-win32，在里面多放了几个东东：

d:\php-5.2.6-win32\RunHiddenConsole.exe

这个exe在http://www.box.net/shared/vfvqyjhday这里找得到

d:\php-5.2.6-win32\start_fastcgi

这是个快捷方式，先造一个RunHiddenConsole.exe的快捷方式，然后进入属性修改目标为

D:\php-5.2.6-Win32\RunHiddenConsole.exe php-cgi -b 127.0.0.1:521

当然，亦可写个bat，目的都是一样的

—————————————————————–

使用的时候，开启nginx，然后执行一下d:\php-5.2.6-win32\start_fastcgi就可以了，php和nginx的其他配置，自己打理去咯。

附说-nginx php的工作原理：

nginx以一种类似代理的模式，去连接fastcgi的端口，php需要开启cgi引擎，然后监听相应的端口即可，fastcgi下nginx和php的耦合度比较小，所以相互影响会减到最低限度。spawn-fcgi和RunHiddenConsole.exe分别是linux和windows下用来管理php-cgi的工具，spawn-fcgi可以制造多个php-cgi进程监听同样端口比较强大，RunHiddenConsole仅仅是隐藏掉cmd窗口，就算不用这两个工具，php-cgi也能启动。