---
title: php5.3不支持ZendOptimizer的解决办法(Zend Guard Loader)
author: admin
type: post
date: 2011-11-06T08:35:32+00:00
url: /archives/11919
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - php

---
[ **2013-04-04]好像ZendGuard-5_5_0版本找不到这个dll文件的**

PHP 5.3 下，Zend Optimizer 已经被全新的 **Zend Guard Loader** 取代

已经Zend Optimer的代替品为 Opcache，请参考： [http://blog.haohtml.com/archives/14071](http://blog.haohtml.com/archives/14071)

————————————————

1. 下载 Zend Guard Loader 压缩包。（官方下载地址： [http://www.zend.com/en/products/guard/downloads](http://www.zend.com/en/products/guard/downloads)）

2. 解压并提取 ZendGuardLoader.so（Linux）或 ZendLoader.dll（Windows），对应你的PHP版本。

3. 在你的 php.ini 文件添加下面一行，用来加载 Zend Guard Loader：

**Linux 和 Mac OS X:　**zend_extension = 完整路径/ZendGuardLoader.so
**Windows（非线程安全）:** 　zend_extension = 完整路径/ZendLoader.dll

4. 在 php.ini 额外新增一行，启用 Zend Guard Loader：

>  zend_loader.enable = 1

5. 可选：可以在 php.ini 文件添加以下行到 Zend Guard Loader 配置位置：

;禁用许可证检查（为了性能的原因）
zend\_loader.disable\_licensing = 0

;让 Zend Guard Loader 支持混淆级别。级别在 Zend Guard 的 [官方详细文档](http://www.zend.com/topics/Zend-Guard-User-Guidev5x.pdf)。 0 – 不启用混淆
zend\_loader.obfuscation\_level_support = 3

;从这个路径寻找Zend产品授权的产品许可证。欲了解更多有关如何创建一个许可证文件的信息，请参阅 Zend Guard 用户指南.
zend\_loader.license\_path =

6. 如果您使用 Zend debugger，请确保加载 Zend guard Loader。

7. 如果您使用 ioncube loader，请务必在它之前加载 Zend guard Loader。

8. 重新启动Web服务器。

**推荐文章：**
[使用 Zend Opcache 加速 PHP](http://blog.haohtml.com/archives/14071)