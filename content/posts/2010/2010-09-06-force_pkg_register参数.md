---
title: FORCE_PKG_REGISTER参数
author: admin
type: post
date: 2010-09-06T07:03:23+00:00
url: /archives/5598
IM_contentdowned:
 - 1
categories:
 - 服务器

---
更新ports到最新，然后直接重新 (make install) 编辑安装PHP时提示出错。升级之前就想到这个问题，因为没有卸载旧版本的PHP，新版本的可能没有办法正常安装。但是卸载的话相关的几个包也都要重新安装，很麻烦也很浪费时间。google了一圈也没有结果就只能自己试了。

提示是这样的：

```
===>   php5-5.2.6 is already installed
      You may wish to ``make deinstall'' and install this port again
      by ``make reinstall'' to upgrade it properly.
      If you really wish to overwrite the old port of lang/php5
      without deleting it first, set the variable "FORCE_PKG_REGISTER"
      in your environment or the "make install" command line.
*** Error code 1

Stop in /usr/ports/lang/php5.
*** Error code 1

Stop in /usr/ports/lang/php5.
```

这时候可以有几种选择：

解除以前安装，然后安装新的版本：

```
make deinstall
make install
```

重新安装：

```
make reinstall
```

上面两种方法不起作用的时候可以用下面的命令强制安装注册：

```
make install FORCE_PKG_REGISTER="yes"
```