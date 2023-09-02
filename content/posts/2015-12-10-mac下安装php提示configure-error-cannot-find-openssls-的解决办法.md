---
title: 'mac下安装PHP提示configure: error: Cannot find OpenSSL’s 的解决办法'
author: admin
type: post
date: 2015-12-09T19:33:19+00:00
url: /archives/16286
categories:
 - 程序开发

---
在mac 10.11.2 下安装PHP7的时候，在./configure的时候，提示

> checking for strftime… (cached) yes
> checking whether to enable LIBXML support… yes
> checking libxml2 install dir… /usr
> checking for xml2-config path… /usr/bin/xml2-config
> checking whether libxml build works… yes
> checking for OpenSSL support… yes
> checking for Kerberos support… no
> checking whether to use system default cipher list instead of hardcoded value… no
> checking for RAND_egd… no
> checking for pkg-config… no
> configure: error: Cannot find OpenSSL’s

错误，解决办法

```
brew install openssl
```

安装xcode命令行

```
xcode-select --install
```

重启，再次安装即可。

**如果还是不行的话，建议从opensll官方网站下载源码，手动安装一下。**

如果还是不行的话，手动指定一下 openssl 的路径

```
--with-openssl=/usr/local/opt/ssl
```