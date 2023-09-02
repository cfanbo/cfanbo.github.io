---
title: 'fatal error: ‘php.h’ file not found 错误的解决办法'
author: admin
type: post
date: 2015-06-08T21:01:06+00:00
url: /archives/15718
categories:
 - 服务器
tags:
 - swoole

---
今天在MAC下安装Swoole扩展的时候( [https://github.com/swoole/swoole-src](https://github.com/swoole/swoole-src))，提示

> In file included from /Users/sxf/Downloads/swoole-src/swoole.c:19:
> ./php_swoole.h:22:10: fatal error: ‘php.h’ file not found
> #include “php.h”
> ^
> 1 error generated.
> make: \*** [swoole.lo] Error 1

错误。解决办法如下：

[shell]sudo ln -s /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.10.sdk/usr/include /usr/include[/shell]

这里注意一下，你使用的系统版本号。

 *注意MacOSX10.10.sdk修改为自己系统的版本号*

另外也有可能遇到提示 pcre.h 找不到，直接安装一下即可 brew install pcre，参考: [http://blog.csdn.net/rsp19801226/article/details/44590803](http://blog.csdn.net/rsp19801226/article/details/44590803)