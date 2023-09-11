---
title: node.js在linux下的安装教程
author: admin
type: post
date: 2011-06-28T03:19:33+00:00
url: /archives/10080
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - nodejs

---
**一．安装node.js**

> wet http://nodejs.org/dist/node-v0.4.8.tar.gz
> tar zxvf node-v0.4.8.tar.gz
> cd node-v0.4.8
> ./configure –prefix=/usr/local/node
> make
> make install

**二.测试**

> 创建test.js文件，内容如下：
> var http = require(‘http’);
> http.createServer(function (req, res) {
> res.writeHead(200, {‘Content-Type’: ‘text/plain’});
> res.end(‘Hello World\n’);
> }).listen(1337, “127.0.0.1”);
> console.log(‘Server running at http://127.0.0.1:1337/’);

执行：

> node test.js

在浏览器里输入 http://127.0.0.1:1337/，可以看到 “Hello World“字样，即表示安装成功!注意后面不能加文件名．

**注意事项：**

1.客户端只能通过端口访问，不能指定js文件名．
2.监听IP地址可以省略，这样任何地方都可以访问.如果指定了127.0.0.1，则只能在本机才可以访问！

另一篇不错的文章： [http://www.oschina.net/question/129540_21801](http://www.oschina.net/question/129540_21801)