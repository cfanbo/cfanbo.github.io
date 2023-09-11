---
title: “Xdebug MUST be loaded as a Zend extension in Unknown on line 0 “的解决办法
author: admin
type: post
date: 2011-02-22T06:09:28+00:00
url: /archives/7774
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - xdebug

---
**解决方法:**

找到 php.ini 中的并修改如下:

写道

;extension=php_xdebug-2.1.0-5.2-vc6.dll

zend_extension_ts=”d:/AppServ\php5\ext\php_xdebug-2.1.0-5.2-vc6.dll” //如果有其它提示,将”_ts”去掉就可以了

xdebug 必须使用 zend\_extension\_ts 或者 zend_extension 来标明它是zend的扩展

写道

另：根据 PHP 版本，zend_extension 指令可以是以下之一：


zend_extension (non ZTS, non debug build)

zend_extension_ts ( ZTS, non debug build)

zend_extension_debug (non ZTS, debug build)

zend_extension_debug_ts ( ZTS, debug build)


ZTS：ZEND Thread Safety


可通过phpinfo()查看ZTS是否启用，从而决定用zend_extension还是zend_extension_ts。


extension意为基于php引擎的扩展


zend_extension意为基于zend引擎的扩展