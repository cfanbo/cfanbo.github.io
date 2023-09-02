---
title: windows下node.js之 express框架+jade模板搭建
author: admin
type: post
date: 2012-11-01T13:30:35+00:00
url: /archives/13472
categories:
 - 程序开发
tags:
 - nodejs

---
1、node.js安装

在Windows平台部署Node.js比较容易，从0.6.1开始，Node.js在Windows平台上可直接通过.mis文件安装。

下载地址 [http://nodejs.org/#download](http://nodejs.org/#download) 目前最新版本是 node-v0.8.3-x86.msi

文件在安装过程中已经指定了默认安装路径。

验证node.js 安装是否成功

打开cmd，直接输入**node -v**

2.npm安装

node安装成功后npm已经默认安装，npm可以直接安装相关扩展

验证npm是否安装成功

打开cmd，直接输入**npm**** -v**

3.express安装

打开cmd，直接输入**npm install -g express**

**-g：在当前目录下安装express框架**

[![](http://blog.haohtml.com/wp-content/uploads/2012/11/install_express_on_windows_for_nodejs.jpg)][1]

验证express是否安装成功

安装完成后，关闭cmd，在重新打开

进入cmd，直接输入**express -V**

**注意我这里用的大写V**

4.用express创建项目

1).cmd进入要创建项目的目录，直接输入 express testapp（项目名称）

2)cd testapp   //进入刚新建的站点目录

**3)cmd app  //再次进入刚创建的项目 输入npm install**

[![](http://blog.haohtml.com/wp-content/uploads/2012/11/express_create_project.jpg)][2]

完后你会发现你站点目录(这里为c:\documetns and settings\Administrator)下多了 node_modules，这个目录就是扩展库文件

express本来默认提供的引擎是jade模板引擎，它颠覆了传统的模板引擎，制定了一套完整的语法用来生产HTML的每个标签，功能强大但是不易学习，所以使用ejs模板，语法与asp、jsp和php一样，易于学习。如果要换模板引擎的话，可以通过编辑 **package.json** 来实现

**现在cmd到项目目录下运行**node app.js****

打开浏览器 http://127.0.0.1:3000/就可以访问了，到此环境搭建完成，开始新的旅程吧。

下一步我们选择开发IDE工具,这里选用webstorm.教程见：

======================================================

想看文档教程神马的，请到官网：

[http://expressjs.com](http://expressjs.com)

Express的源码托管在这里：

[https://github.com/visionmedia/express](https://github.com/visionmedia/express)

Express使用了Connect中间件，Connect的文档在这里：

[http://senchalabs.github.com/connect/](http://senchalabs.github.com/connect/)

如果你想做单元测试，可以看看TDD框架Expresso：

[http://visionmedia.github.com/expresso/](http://visionmedia.github.com/expresso/)

Express可选的Jade模板引擎也挺有趣的，它和haml是亲戚，像是个HTML预编译器：

[http://jade-lang.com/](http://jade-lang.com/)

说到haml，你也可以了解下sass，它像一个CSS预编译器：

[https://github.com/visionmedia/sass.js](https://github.com/visionmedia/sass.js)

不过同类产品中，我更喜欢不那么激进的less：

[https://github.com/cloudhead/less.js](https://github.com/cloudhead/less.js)

汗～我严重跑题了，本来只想简单介绍下Express的。

最后，node.js是以上各种玩意儿跑起来的前提：

[http://nodejs.org/](http://nodejs.org/)



 [1]: http://blog.haohtml.com/wp-content/uploads/2012/11/install_express_on_windows_for_nodejs.jpg
 [2]: http://blog.haohtml.com/wp-content/uploads/2012/11/express_create_project.jpg