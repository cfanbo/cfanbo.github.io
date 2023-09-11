---
title: X-Cache 和 X-Cache-Lookup headers 的解释
author: admin
type: post
date: 2010-07-24T15:08:01+00:00
url: /archives/4783
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - 网页优化

---
X-Cache: 表示你的 http request 是由 proxy server 回的 .
MISS 表 proxy无资料,代理动作, HIT 表 proxy 直接回应

X-Pad: 這個是800 年前的 netscape  bug 的因素才用的

想象你在一个标准的[透明代理][1]80端口下，并且你正在访问一个运行了内部网络缓存（这样，又是一个代理）的站点。如果你查看HTTP headers查找某些信息，你能够找到像这样的2行，规定domain.tld 代表那个本地网站，proxy.local 代表你的内部的透明代理。

X-Cache :HIT from proxy.domain.tld, MISS from proxy.local
X-Cache-Lookup :HIT from proxy.domain.tld:3128, MISS from proxy.local:3128

这2行是什么意思？如果这是你第一次访问那个站点（MISS from proxy.local），并且它的代理的缓存中有一个有效的网页(X-Cache HIT proxy.domain.tld)

现在我们刷新了页面后(F5, Ctrl+R, you name it)将会发生什么呢？

X-Cache:MISS from proxy.domain.tld, MISS from proxy.local
X-Cache-Lookup :HIT from proxy.domain.tld:3128, HIT from proxy.local:3128

看起来好像2个代理都没有进行服务，我们在 X-Cache-Lookup状态看到2个难以理解的HITs。但这是很简单的，我们没有把另一个层次的缓存考虑进来 ，那就是浏览器缓存。因此网页现在没有经过网络传输，而是浏览器使用了它自己的缓存来显示网页。因此我们在X-Cache状态看到2个MISSes 。但是如果发出请求代理仍然会发送缓存。因而，如果你正在调试你的代理系统，出现这种情况是正常的。

现在，如果我们清空浏览器缓存。再次请求页面，将看到：

X-Cache :MISS from proxy.domain.tld, HIT from proxy.local
X-Cache-Lookup: HIT from proxy.domain.tld:3128, HIT from proxy.local:3128

我们的透明代理已获得了我们想要的网页缓存，因而他直接发送给我们(HIT from proxy.local)。另一个远程代理不用做任何事情。这是2个代理都能发送我们需要的网页。

以上为翻译:[X-Cache and X-Cache-Lookup headers explained][2]

总结：X-Cache和x-cache-lookup项常见于squid做代理服务器软件的网站的http header。依据上面的现象推测，x-cache-lookup项指专门查看代理服务器中**是否有**某个网页缓存。有就返回HIT,没有返回MISS。而x-cache项指浏览器从何处、是在哪个代理缓存载入的网页文件。服务器名后的3128指服务器端口。

 [1]: http://vbb.twftp.org/archive/index.php/t-1145.html
 [2]: http://anothersysadmin.wordpress.com/2008/04/22/x-cache-and-x-cache-lookup-headers-explained/