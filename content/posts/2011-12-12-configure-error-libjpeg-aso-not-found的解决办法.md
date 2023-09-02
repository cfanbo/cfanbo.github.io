---
title: 'Centos64位系统下”configure: error: libjpeg.(a|so) not found”的解决办法'
author: admin
type: post
date: 2011-12-12T13:23:10+00:00
url: /archives/12272
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - lnmp
 - php

---
刚刚发布了Centos6.1新版本.就下载了64位的版本进行测试.

按照原来的lnmp安装教程.在安装php的过程中.执行到./configure 这一步的时候.竟然提示”configure: error: libjpeg.(a|so) not found”这项错误.明明已经安装过了libjpeg 和libjpeg-devel了.可这里仍然提示找不到库文件.很明显是路径的问题.默认会在/usr/lib/目录里查找相应的文件.但用whereis libjpeg发现.libjpeg被安装在了/usr/lib64/目录里.

```
[root@bogon php-5.2.17]# whereis libjpeg
libjpeg: /usr/lib/libjpeg.so /usr/lib64/libjpeg.so
```

1.如果提示”configure: error: libjpeg.(a|so) not found”错误

所以这里我们需要复制一份libjpeg.so到/usr/lib/目录里才可以.再次执行./configure命令即可.

```
cp -frp /usr/lib64/libjpeg.* /usr/lib/
```

2.注意过程中还会提示” Configure: error: libpng.(also) not found.“错误,解决办法和上面的一样.

```
cp -frp /usr/lib64/libpng* /usr/lib/
```

3.如果提示”configure: error: Cannot find ldap libraries in /usr/lib.”的话.

cp -frp /usr/lib64/libldap* /usr/lib/

**说明:**

> 通过上面的搜索其实就知道一些原因了,configure一般的搜索编译路径为/usr/lib/下,因为php默认就在/usr/lib/下找相关库文件,而x64机器上是在:/usr/lib64.这时你就可以直接把需要的库文件从/usr/lib64中拷贝到/usr/lib/中去就可以了.

常见错误参考: