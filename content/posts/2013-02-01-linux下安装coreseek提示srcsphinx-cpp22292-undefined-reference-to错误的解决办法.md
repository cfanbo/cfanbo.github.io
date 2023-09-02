---
title: 'linux下安装coreseek提示”/src/sphinx.cpp:22292: undefined reference to”错误的解决办法'
author: admin
type: post
date: 2013-02-01T02:17:11+00:00
url: /archives/13641
categories:
 - 服务器
tags:
 - coreseek.sphinx

---
今天在64位的Centos5.8系统下安装coreseek的时候，发现编辑的的时候总是出错

/root/coreseek-4.1-beta/csft-4.1/src/sphinx.cpp:22292: undefined reference to \`libiconv_open’
/root/coreseek-4.1-beta/csft-4.1/src/sphinx.cpp:22310: undefined reference to \`libiconv’
/root/coreseek-4.1-beta/csft-4.1/src/sphinx.cpp:22316: undefined reference to \`libiconv_close’
collect2: ld returned 1 exit status
make[2]: \*** [indexer] Error 1
make[2]: Leaving directory \`/root/coreseek-4.1-beta/csft-4.1/src’
make[1]: \*** [all] Error 2
make[1]: Leaving directory \`/root/coreseek-4.1-beta/csft-4.1/src’
make: \*** [all-recursive] Error 1

在其它机器上未发现此错误.

一开始以为libiconv的问题，又重装了几次还是一样，最后终于找着办法了
编辑：
./src/MakeFile文件
将
LIBS = -lm -lexpat -L/usr/local/lib
改成
LIBS = -lm -lexpat -liconv -L/usr/local/lib

就可以了。

在[http://www.coreseek.cn/products-install/install\_on\_bsd_linux/][1] 的最下方也有这个问题的解决办法.

 [1]: http://www.coreseek.cn/products-install/install_on_bsd_linux/