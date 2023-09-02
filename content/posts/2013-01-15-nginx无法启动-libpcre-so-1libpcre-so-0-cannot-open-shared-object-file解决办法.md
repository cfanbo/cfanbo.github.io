---
title: 'nginx无法启动: libpcre.so.1/libpcre.so.0: cannot open shared object file解决办法'
author: admin
type: post
date: 2013-01-14T16:30:23+00:00
url: /archives/13571
categories:
 - 服务器
tags:
 - nginx

---
NGINX启动时提示错误：

> /usr/local/nginx/sbin/nginx -t
> /usr/local/nginx/sbin/nginx: error while loading shared libraries: libpcre.so.1: cannot open shared object file: No such file or directory

ldd $(which /usr/local/nginx/sbin/nginx)

> linux-vdso.so.1 => (0x00007fff48ff0000)
> libcrypt.so.1 => /lib64/libcrypt.so.1 (0x0000003065800000)
> libpcre.so.1 => not found
> libssl.so.6 => /lib64/libssl.so.6 (0x0000003067000000)
> libcrypto.so.6 => /lib64/libcrypto.so.6 (0x0000003066400000)
> libdl.so.2 => /lib64/libdl.so.2 (0x0000003063000000)
> libz.so.1 => /lib64/libz.so.1 (0x0000003063c00000)
> libc.so.6 => /lib64/libc.so.6 (0x0000003062c00000)
> libgssapi\_krb5.so.2 => /usr/lib64/libgssapi\_krb5.so.2 (0x0000003066c00000)
> libkrb5.so.3 => /usr/lib64/libkrb5.so.3 (0x0000003069c00000)
> libcom\_err.so.2 => /lib64/libcom\_err.so.2 (0x0000003068800000)
> libk5crypto.so.3 => /usr/lib64/libk5crypto.so.3 (0x0000003069000000)
> /lib64/ld-linux-x86-64.so.2 (0x0000003062800000)
> libkrb5support.so.0 => /usr/lib64/libkrb5support.so.0 (0x000000306a800000)
> libkeyutils.so.1 => /lib64/libkeyutils.so.1 (0x0000003067c00000)
> libresolv.so.2 => /lib64/libresolv.so.2 (0x0000003068400000)
> libselinux.so.1 => /lib64/libselinux.so.1 (0x0000003064400000)
> libsepol.so.1 => /lib64/libsepol.so.1 (0x0000003064000000)

解决方法：

> ln -s /usr/local/lib/libpcre.so.1 /lib64

32位系统则：

> ln -s /usr/local/lib/libpcre.so.1 /lib

注：
/usr/local/lib/libpcre.so.1 为prce安装后的文件地址
低版本prce对应的libpcre.so.1 为libpcre.so.0