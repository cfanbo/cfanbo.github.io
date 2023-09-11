---
title: TBCompressor – JS和CSS压缩工具
author: admin
type: post
date: 2010-04-26T07:50:48+00:00
url: /archives/3497
IM_contentdowned:
 - 1
categories:
 - 前端设计

---

有朋友问到淘宝是怎么压缩js和css的，这里分享下。

我们使用的是 [YUI Compressor](http://www.julienlecomte.net/yuicompressor/):


> The YUI Compressor is a JavaScript compressor which, in addition to removing comments and white-spaces, obfuscates local variables using the smallest possible variable name. This obfuscation is safe, even when using constructs such as ‘eval’ or ‘with’ (although the compression is not optimal is those cases) Compared to jsmin, the average savings is around 20%.
>
>
> The YUI Compressor is also able to safely compress CSS files. The decision on which compressor is being used is made on the file extension (js or css)

淘宝前端的开发环境以Windows居多。为了方便使用，对YUICompressor做了层简单的封装，称之为TBCompressor. 安装和使用方法如下：

#### 安装说明

1. 安装请点击install.cmd

2. 卸载请点击uninstall.cmd

3. 如果以前安装过2.3.5之前的版本, 请点击update.cmd升级


#### 测试使用

1. 在test.source.js上右键，执行菜单“压缩JavaScript”，会生成test.js文件。如果再对test.js文件执行一次 压缩，会生成test-min.js文件

2. CSS同1


#### 下载试用

[TBCompressor_v2.4.zip](http://lifesinger.org/blog/wp-content/uploads/2008/10/TBCompressor_v2.4.zip) (1.53 MB)

[tbcompressor-2.4.2.zip](http://code.google.com/p/ourtools/downloads/list) (808.3 KB)