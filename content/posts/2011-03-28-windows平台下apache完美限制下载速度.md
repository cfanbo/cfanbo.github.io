---
title: windows平台下apache限制下载速度
author: admin
type: post
date: 2011-03-28T08:23:24+00:00
url: /archives/8187
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache

---
首先说说完美限制的意思：防盗链、限制客户端下载线程数，限制下载带宽。下面一一介绍怎么在Apache里面实现这些功能。

**防盗链**

传统的防盗链都是通过Referer来判断用户来路的，不过这样的方法对于下载工具来说形同虚设，因为现在的下载工具早就能伪造Referer了。

现在一些流行的防盗链的方式都是用在浏览页面的时候产生一个随机验证码，在用户点击连接的时候服务器会验证这个验证码是否有效从而决定是否允许下载。或者就是用某些方法把文件实际地址进行伪装。不过我觉得这些都不怎么好用，我用了一个简单有效的方式来实现防盗链。

其实就是用Apache的URL Rewrite模块就能很简单的就能实现防盗链下载。

在Apache的httpd.conf文件里面搜索：

> #LoadModule rewrite\_module modules/mod\_rewrite.so

把它前面的#去掉，再找到块，在里面加入类似如下代码：
RewriteEngine on
RewriteCond %{HTTP_REFERER} !^ [http://lply.net/.](http://lply.net/)*$ [NC]
RewriteCond %{HTTP_REFERER} !^ [http://lply.com](http://lply.com/) $ [NC]
RewriteCond %{HTTP_REFERER} !^ [http://www.lply.com.](http://www.lply.com./)*$ [NC]
RewriteCond %{HTTP_REFERER} !^ [http://www.lply.com](http://www.lply.com/) $ [NC]
RewriteRule .*\.(gif|jpb|png|css|js|swf])$ [http://disk.lply.net](http://disk.lply.net/) [R,NC]

其中有色的地方都是要改为你的：


红色：就是改为你提供下载页面的地址，也就是只有通过这个地址才可以下载你所提供的东东。
蓝色：就是要保护文件的扩展名(以|分开)，也就是说以这些为扩展名的文件只有通过红色的地址才可以访问。
绿色：如果不是通过红色的地址访问蓝色这些为扩展名的文件时就回重定向到绿色地址上。

这样如果一个盗链而来的请求将会被重定向到错误页面，就算实际地址暴露也不怕。从而防止了服务器资源被盗链的危险。

**限制客户端多线程下载**

限制多线程现在需要用到一个Apache的扩展模块mod_limitipconn，这里是作者的官方网站 [http://dominia.org/djao/limitipconn2.html](http://dominia.org/djao/limitipconn2.html)，先下载适合自己版本的模块文件到Apache安装目录下的modules目录下面(Win32平台 [mod_limitipconn.dll](/wp-content/uploads/2011/03/mod_limitipconn.rar))，然后在httpd.conf文件中搜索：

> #LoadModule status\_module modules/mod\_status.so

把它前面的#去掉，再加入：

> #开启ExtendedStatus，默认在extra/httpd-info.conf文件里，不能用在虚拟主机设置区域
> ExtendedStatus On
>
> \# 如果你下载的不是Win版，请把后面的文件名改为你所下载的文件名，多次使用只需要加载一次就可以了
> LoadModule limitipconn\_module modules/mod\_limitipconn.dll
>
> \# 这里表示限制根目录，即全部限制，可以根据需要修改
>
> \# 这里表示最多同时两个线程
> MaxConnPerIP 2
> \# 这里表示html目录下不受限制
> NoLimit html/*
>
>

这样来自同一客户端的超过2个的线程请求将被拒绝，从而限制了客户端的多线程下载。

**限制下载带宽**

这个同样需要扩展模块支持，模块是**mod_bw**，注意启用该模块，必须先行启用status模块

> LoadModule status_module modules/mod_status.so
> ExtendedStatus On

否则限速模块无效。

作者的官方网站 [http://ivn.cl/apache/](http://ivn.cl/apache/)(Win32: [http://ivn.cl/files/dlls/mod_bw-0.91-2.2.14/mod_bw.dll](http://ivn.cl/files/dlls/mod_bw-0.91-2.2.14/mod_bw.dll) 可以下载到。同样也是放入modules目录下面，然后在httpd.conf文件中加入：

> LoadModule bw\_module modules/mod\_bw.dll

再找到块，加入：

>
> BandwidthModule On # 启动带宽限制
> ForceBandWidthModule On # 启动带宽限制
> Bandwidth all 0
> MinBandwidth all 0   # 0表示除下面限制文件外的文件不限速，当然你也可以改成比如400000限制为400K/S
> LargeFileLimit *.rar 10240 250000 # 大于10MB的rar后缀文件限速为250K/S，以下类推
> LargeFileLimit *.zip 10240 250000
> LargeFileLimit *.exe 10240 250000
>

到此，我们的完美限制的HTTP下载服务器就配置完成了，重新启动你的Apache这些功能便能生效了。因为Apache和这些模块都是开源免费的，我们不需要为此掏一分钱，不用去购买那些第三方的软件，只是需要多去了解一下这些软件的使用说明。不要一切都祈祷有现成美好的东西，自己动手做一次会有不一样的收获。

官方提供了一种查看下载信息的方法:

Suppose the vhost for 127.0.0.1 :

> `<location /modbw` `>`
>
> `SetHandler modbw-handler`
>
> `</location>`

Now you can get information of the mod by visiting [http://127.0.0.1/modbw](http://127.0.0.1/modbw)
You can get the same information in csv format at [http://127.0.0.1/modbw?csv](http://127.0.0.1/modbw?csv) [本地下载mod_bw](/wp-content/uploads/2011/03/mod_bw.rar)

**相关文章:**

[使用apache的rewrite功能来防迅雷](http://blog.haohtml.com/archives/8196) [Apache带宽流量控制模块安装 mod_bw 配置说明](http://blog.haohtml.com/archives/8205)