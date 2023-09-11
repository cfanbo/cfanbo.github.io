---
title: portupgrade和portconf配合升级软件
author: admin
type: post
date: 2009-12-11T01:46:40+00:00
url: /archives/2710
IM_contentdowned:
 - 1
categories:
 - 服务器

---
portupgrade是不带参数的(可能有带参数的办法?我不知道),带参数编译ports我习惯

cd /usr/ports/ports-mgmt/portconfBSD
make install cleanBSD

然后在 /usr/local/etc/ports.conf 加入

databases/mysql50*: WITH\_XCHARSET=all | BUILD\_OPTIMIZED=yes | WITHOUT_INNODB=yes

这样不管是直接在ports中make或者还是portupgrade都会取这里面的参数

这样管理升级ports就很方便了