---
title: incorrect MySQL client library version! This gem was compiled for 6.0.0 but the client library is 5.5.20. 的解决办法
author: admin
type: post
date: 2012-08-10T06:43:41+00:00
url: /archives/13269
categories:
 - 数据库

---
从mysql官方下载 mysql-connector-c-noinstall-6.0.2-win32 解压到e:/。注意根据自己的实际情况下载相对应的版本，这里使用非安装版。

or Ruby 1.9.2:

 gem install mysql --platform=ruby -- --with-mysql-dir=e:/mysql-connector-c-noinstall-6.0.2-win32

for Ruby 1.9.3: (showing mysql2 variant)

 gem pristine mysql2 -- --with-mysql-config=e:\mysql-connector-c-noinstall-6.0.2-win32

这里我用64位的win7系统.

然后将** E:\mysql-connector-c-noinstall-6.0.2-win32\lib** 目录下的 libmysql.lib 文件复制到 **E:\RailsInstaller\Ruby1.9.3\bin** 目录下。

重新执行刚才的命令即可。