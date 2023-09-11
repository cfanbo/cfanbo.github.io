---
title: windows下安装rabbitmq的php扩展amqp（原创)
author: admin
type: post
date: 2015-03-28T14:06:34+00:00
url: /archives/15484
categories:
 - 系统架构
tags:
 - amqp
 - RabbitMQ

---
从php官方下载相应的版本 [http://pecl.php.net/package/amqp](http://pecl.php.net/package/amqp)，我这里使用的是1.4.0版本（ [http://pecl.php.net/package/amqp/1.4.0/windows](http://pecl.php.net/package/amqp/1.4.0/windows)）
根据当前使用的php版本选择相应的扩展dll，下载后是一个压缩包，里面有两个dll扩展(php_amqp.dll和rabbitmq.1.dll)。

[![php_amqp](https://blogstatic.haohtml.com//uploads/2023/09/php_amqp1.jpg)][1]

我的环境是64位的,php5.5.12.所以使用的是 [http://windows.php.net/downloads/pecl/releases/amqp/1.4.0/php_amqp-1.4.0-5.5-ts-vc11-x64.zip](http://windows.php.net/downloads/pecl/releases/amqp/1.4.0/php_amqp-1.4.0-5.5-ts-vc11-x64.zip)

1.将php_amqp.dll放在php的ext目录里，然后修改php.ini文件，在文件的最后面添加两行

```
[amqp\]
extension=php_amqp.dll
```



2.将rabbitmq.1.dll文件放在php的根目录里(也就是ext目录的父级目录)，然后修改apache的httpd.con文件，文件尾部添加一行

```
LoadFile "d:/wamp/bin/php/php5.5.12/rabbitmq.1.dll"
```



这里的路径根据情况修改，我这里使用的wampserver软件。

3.重启apache，并查看phpinfo信息。只要看到amqp 字样即可。

[![amqp](https://blogstatic.haohtml.com//uploads/2023/09/amqp.jpg)][2]

下面我们看一下rabbitmq扩展amqp的用法: [用PHP尝试RabbitMQ（amqp扩展）实现消息的发送和接收](http://blog.haohtml.com/archives/15491)

[1]: http://blog.haohtml.com/wp-content/uploads/2015/03/php_amqp1.jpg
[2]: http://blog.haohtml.com/wp-content/uploads/2015/03/amqp.jpg