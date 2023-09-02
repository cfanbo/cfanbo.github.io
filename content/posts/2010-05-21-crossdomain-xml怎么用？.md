---
title: crossdomain.xml怎么用？
author: admin
type: post
date: 2010-05-21T04:52:43+00:00
url: /archives/3670
IM_contentdowned:
 - 1
categories:
 - 程序开发

---
crossdomain.xml是adobe搞的，为了让flash跨域访问文件。

该配置文件放于服务器端的根目录下面。来设置让哪些域名下面的swf文件能够访问我服务器上的内容。

比如：我的服务器上有个图片：http://www.haohtml.com/img.gif
sina上面有个swf需要访问我的这个文件。
配置文件该这样写：

Xml代 码



1. xmlversion=“1.0”?>
2. >
3. <cross-domain-policy>
4. <allow-access-fromdomain=“*.haohtml.com”/>
5. cross-domain-policy>

```
<?xml version="1.0"?>
<!DOCTYPE cross-domain-policy SYSTEM "http://www.macromedia.com/xml/dtds/cross-domain-policy.dtd">
<cross-domain-policy>
    <allow-access-from domain="*.sina.com" />
</cross-domain-policy>
```

该文件放在http://www.haohtml.com/crossdomain.xml