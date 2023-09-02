---
title: '隐藏 Apache & PHP 的版本号'
author: admin
type: post
date: 2011-07-22T17:05:58+00:00
url: /archives/10605
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache

---
有朋友问起，如何隐藏 HTTP header 中发送包含在 Server 信息里面的 Apache 和 PHP 版本号（譬如我们可以到 Firefox 的附加工具里面找 Live HTTP Headers;还可以用curl -I IPaddress|http://域名 ) 下面是做法：
**Apache:**
打开 httpd.conf，在文件最后加入以下代码：

1. #Hidden I can with apache version number

2. ServerTokens ProductOnly

3. ServerSignature Off


**PHP:**
隐藏 PHP 版本就是隐藏类似于 “X-Powered-By: PHP/5.1.2-1+b1” 这个，开启 php.ini，加入:

1. expose_php = Off


设置了expose_php=Off后,用phpinfo查看的时候,页面上原来正常显示的图片会消失隐藏的.

_相关:_