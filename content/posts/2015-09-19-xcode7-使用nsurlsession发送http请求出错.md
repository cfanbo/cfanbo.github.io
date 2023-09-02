---
title: Xcode7 使用NSURLSession发送HTTP请求出错
author: admin
type: post
date: 2015-09-19T11:20:48+00:00
url: /archives/15988
categories:
 - 程序开发

---
控制台打印：

> Application Transport Security has blocked a cleartext HTTP (http://) resource load since it is insecure. Temporary exceptions can be configured via your app’s Info.plist file.

在iOS9 中，苹果将原http协议改成了https协议，使用 TLS1.2 SSL加密请求数据。

**解决方法：**

在info.plist中添加
NSAppTransportSecurityNSAllowsArbitraryLoads
<true/>

[![20150629175005991](http://blog.haohtml.com/wp-content/uploads/2015/09/20150629175005991.png)][1]

官方文档：

[https://developer.apple.com/library/prerelease/ios/releasenotes/General/WhatsNewIniOS/Articles/iOS9.html#//apple_ref/doc/uid/TP40016198-DontLinkElementID_13](https://developer.apple.com/library/prerelease/ios/releasenotes/General/WhatsNewIniOS/Articles/iOS9.html#//apple_ref/doc/uid/TP40016198-DontLinkElementID_13)

 [1]: http://blog.haohtml.com/wp-content/uploads/2015/09/20150629175005991.png