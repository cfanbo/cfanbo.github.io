---
title: linux下安装zeromq消息队列软件
author: admin
type: post
date: 2013-05-05T10:00:22+00:00
url: /archives/13798
categories:
 - 系统架构
tags:
 - zeromq
 - 消息队列

---
在上一节　[消息中间件的技术选型心得－RabbitMQ、ActiveMQ和ZeroMQ](http://blog.haohtml.com/archives/13790)　我们介绍了一些相关的消息队列软件．这里我们对安装zeromqq这款软件的安装及php使用方法介绍一下．

centos下安装zeromq消息队列软件．

**一．安装服务端**

```shell
cd ~
wget http://download.zeromq.org/zeromq-3.2.3.tar.gz
tar zxvf zeromq-3.2.3.tar.gz
cd zeromq-3.2.3
./configure # –prefix=/usr/local/zeromq
make && make install
```



**二．安装php扩展　**

```shell
git clone git://github.com/mkoppanen/php-zmq.git
cd php-zmq
phpize
./configure –with-php-config=/usr/local/php/bin/php-config
make && make install
```



执行完以后，会提示：

Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/

表示生成了动态链接库文件zmq.so.这个时候可以查看一下目录里有没有zmq.so 这个文件．

如果在make的时候，提示

checking for pkg-config… /usr/bin/pkg-configchecking libzmq installation… configure: error: Unable to find libzmq installation

则只需要将安装zeromq的 -prefix 项取消即可．

zeromq扩展安装： [http://zeromq.org/bindings:php](http://zeromq.org/bindings:php)
windows下zmq扩展： [http://valokuva.org/builds/](http://valokuva.org/builds/) 和
[http://stackoverflow.com/questions/6742773/zeromq-php-extension-for-windows](http://stackoverflow.com/questions/6742773/zeromq-php-extension-for-windows)

[https://github.com/Polycademy/php\_zmq\_binaries/tree/master/php-zmq-20130203/php-zmq][1]

[http://windows.php.net/downloads/pecl/releases/zmq/](http://windows.php.net/downloads/pecl/releases/zmq/) [http://pecl.php.net/package/zmq/1.1.2/windows](http://pecl.php.net/package/zmq/1.1.2/windows)

**三．配置PHP.INI**

修改/usr/loal/php/etc/php.ini文件，在extension 区块添加一行

```
extension=zmq.so
```



，然后重启php-fpm．通过phpinfo()　查看zmq的php扩展是否已经安装成功．

[![zmq-php-extension](https://blogstatic.haohtml.com//uploads/2023/09/zmq-php-extension.jpg)][2]

—————————————–

对于window下安装zmq扩展的话，先下载 [http://pecl.php.net/package/zmq/1.1.2/windows](http://pecl.php.net/package/zmq/1.1.2/windows) 与本地一致的压缩包，压缩包里会有 **libzmq.dll** 和 **php_zmq.dll** 两个dll文件，先将php_zmq.dll 文件放在php的ext目录里（D:\wamp\bin\php\php5.5.12\ext)，再将 libzmq.dll 文件放在 php 的根目录里 D:\wamp\bin\php，然后分别修改php.ini文件，添加 extension=php_zmq.dll 一行，然后财修改apache的httpd.conf文件，在最后面添加一行 LoadFile “D:/wamp/bin/php/php5.5.12/libzmq.dll” 然后重启apache即可。(现在许多扩展都会提供丙个dll文件，基本上全部使用这个方面就可以了，如rabbitmq的php扩展 [amqp](http://pecl.php.net/package/amqp))

四．实例

```
cd php-zmq
mv examples/ /usr/local/nginx/html/
```



访问http://localhost/examples/client.php,会看到 string(7) “Got it!” 字样，表示写入队列成功．

现在可以查看是否监听了 5555 端口

[shell]netstat -anp | grep 5555[/shell]

php使用手册可参考： [http://zguide.zeromq.org/php:all](http://zguide.zeromq.org/php:all)

另一篇相关zeroMQ的介绍文章：

[1]: https://github.com/Polycademy/php_zmq_binaries/tree/master/php-zmq-20130203/php-zmq
[2]: http://blog.haohtml.com/wp-content/uploads/2013/05/zmq-php-extension.jpg