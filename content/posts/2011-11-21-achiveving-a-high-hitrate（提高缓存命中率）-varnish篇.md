---
title: Achiveving a high hitrate（提高缓存命中率）-varnish篇
author: admin
type: post
date: 2011-11-21T04:42:34+00:00
url: /archives/12038
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - hitrate
 - varnish

---
现在 varnish 已经正常运行了,您可以通过 varnish 访问到您的 web 应用程序。如果您的 web 程序在设计时候没有考虑到加速器的架构，那么您可能有必要修改您的应用程序或者varnish配置文件，来提高varnish的命中率。
既然这样，您就需要一个工具用来观察您和web服务器之间HTTP头信息。服务器端您可以轻松的使用varnish 的工具，比如varnishlog和 varnishtop，但是客户端的工具需要您自己去准备，下面是我经常使用的工具。
**Varnistop**
您可以使用varnishtop 确定哪些URL经常命中后端。 Varnishtop –i txurl 就是一个基本的命令。您可以通过阅读“Statistics”了解其他示例。

**Varnishlog**
当您需要鉴定哪个 URL 被频繁的发送到后端服务器，您可以通过varnishlog对请求做一个全面的分析。 varnishlog –c –o /foo/bar 这个命令将告诉您所有（-o）包含”/football/bar”字段来自客户端（-c）的请求。

**Lwp-request**
Lwp-request是 www 库的一部分，使用perl语言编写。它是一个真正的基本程序，它可以执行HTTP请求，并给您返回结果。我主要使用两个程序，GET 和HEAD。
Vg.no 是第一个使用 varnish 的站点，他们使用 varnish 相当完整，所以我们来看看他们的HTTP 头文件。我们使用 GET请求他们的主页：

```
$ GET -H 'Host: www.vg.no' -Used http://vg.no/
GET http://vg.no/
Host: www.vg.no
User-Agent: lwp-request/5.834 libwww-perl/5.834

200 OK
Cache-Control: must-revalidate
Refresh: 600
Title: VG Nett - Forsiden - VG Nett
X-Age: 463
X-Cache: HIT
X-Rick-Would-Never: Let you down
X-VG-Jobb:  http://www.finn.no/finn/job/fulltime/result?keyword=vg+multimedia
Merk:HeaderNinja
X-VG-Korken: http://www.youtube.com/watch?v=Fcj8CnD5188
X-VG-WebCache: joanie
X-VG-WebServer: leon
```

OK，我们来分析它做了什么。GET 通过发送 HTTP 0.9 的请求，它没有主机头，所以我需要添加一个主机头使用-H 选项，-U打印请求的头，-s打印返回状态，-e 答应返回状态的头，-d 丢弃当前的连接。我们正真关心的不是连接，而是头文件。
如您所见 VG 的头文件中有相当多的信息，比如 X-RICK-WOULD-NEVER 是 vg.no 定制的信息，他们有几分奇怪的幽默感。其他的内容，比如X-VG-WEBCACHE 是用来调试错误的。
核对一个站点是否使用 cookies，可以使用下面的命令：

```
GET -Used http://example.com/ |grep ^Set-Cookie
```

**Live HTTP Headers**
这是一个firefox的插件，live HTTP headers 可以查看您发送的和接收的 http头。软件在https://addons.mozilla.org/en-US/firefox/addon/3829/下载。或者google“Live HTTP headers”。

**The Role of HTTP headers**
Varnish 认为自己是真正的 web 服务器，因为它属于您控制。IETF 没有真正定义surrogate origin cache 角色的含义，（The role of surrogate origin cache is not really well defined by the IETF so RFC 2616 doesn’t always tell us what we should do.不知如何翻译）

**Cache-Control**
Cache-control指示缓存如何处理内容，varnish 关心max-age 参数，并使用这个参数计算每个对象的TTL值。
“cache-control：nocache” 这个参数已经被忽略，不过您可以很容易的使它生效。
在头信息中控制 cache-control的max-age，您可以参照下面，varnish 软件管理服务器的例子：

> $ GET -Used http://www.varnish-software.com/|grep ^Cache-Control
> Cache-Control: public, max-age=600

**Age**
Varnish添加了一个age头信息，用来指示对象已经被保存在varnish中多长时间了。
您可以在varnish中找到Age 信息：

> varnishlog -i TxHeader -I ^Age

**Overriding the time-to-live（ttl）**
有时候后端服务器会当掉，也许是您的配置问题，很容易修复。不过更简单的方法是修改您的ttl，能在某种程度上修复难处理的后端。
您需要在VCL 中使用beresp.ttl定义您需要修改的对象的 TTL：

```
sub vcl_fetch {
     if (req.url ~ "^/legacy_broken_cms/") {
         set beresp.ttl = 5d;
     }
}
```

**Cookies**
现在Varnish 接收到后端服务器返回的头信息中有Set-Cookie 信息的话，将不缓存。
所以当客户端发送一个 Cookie 头的话，varnish 将直接忽略缓存，发送到后端服务器。
这样的话有点过度的保守，很多站点使用 Google Analytics（GA）来分析他们的流量。GA 设置一个 cookie 跟踪您，这个 cookie 是客户端上的一个 java 脚本，因此他们对服务器不感兴趣。
对于一个web 站点来说，忽略一般cookies是有道理的，除非您是访问一些关键部分。这个VCL的vcl_recv片段将忽略cookies，除非您正在访问/admin/：

```
if ( !( req.url ~  “^/admin/”) ) {
   unset req.http.Cookie;
}
```

很简单，不管您需要做多么复杂的事情，比如您要删除一个 cookies，这个事情很困难，varnish 也没有相应的工具来处理，但是我们可以使用正则表达式来完成这个工作，如果您熟悉正则表达式，您将明白接下来的工作，如果您不会我建议您找找相关资料学习一下。
我们来看看 varnish 软件是怎么工作的，我们使用一些 GA 和一些相似的工具产生cookies。所有的cookies使用jsp语言。 Varnish和企业网站不需要这些cookies，而 varnish会因为这些cookies而降低命中率，我们将放弃这些多余的cookies，使用 VCL。
下面的VCL将会丢弃所有被匹配的 cookies。

> // Remove has\_js and Google Analytics \__* cookies.
> set req.http.Cookie = regsuball(req.http.Cookie, “(^|;\s\*)(\_[\_a-z]+|has_js)=[^;]\*”, “”);
> // Remove a “;” prefix, if present.
> set req.http.Cookie = regsuball(req.http.Cookie, “^;\s*”, “”);

下面的例子将删除所有名字叫 COOKIE1和 COOKIE2的cookies：

```
sub vcl_recv {
   if (req.http.Cookie) {
     set req.http.Cookie = ";" req.http.Cookie;
     set req.http.Cookie = regsuball(req.http.Cookie, "; +", ";");
     set req.http.Cookie = regsuball(req.http.Cookie, ";(COOKIE1|COOKIE2)=", "; \1=");
     set req.http.Cookie = regsuball(req.http.Cookie, ";[^ ][^;]*", "");
     set req.http.Cookie = regsuball(req.http.Cookie, "^[; ]+|[; ]+$", "");
     if (req.http.Cookie == "") {
         remove req.http.Cookie;
     }
}
```

这个例子是来自 varnish wiki的。

**Vary**
各式各样的头被发送到web server，他们让HTTP目标多样化。Accept-Encoding就有这种感觉，当一个服务器分发一个“Vary：Accept-Encoding”给 varnish。Varnish 需cache来自客户端的每个不同的Accept-Encoding。如果客户端只接收 gzip编码，varnish不
其他编码服务，那么就可以缩减编码量。
问题就是这样的，Accept-Encoding字段包含很多编码方式，下面是不同浏览器发的：
Accept-Encodign: gzip,deflate
另一个浏览器发送的：
Accept-Encoding:: deflate, gzip
Varnish可以使两个不同的accept-enconding头标准化，这样就可以尽量减少体。下面的VCL代码可以是 accept-encoding头标准化：

```
if (req.http.Accept-Encoding) {
     if (req.url ~ "\.(jpg|png|gif|gz|tgz|bz2|tbz|mp3|ogg)$")
         # No point in compressing these
         remove req.http.Accept-Encoding;
     } elsif (req.http.Accept-Encoding ~ "gzip") {
         set req.http.Accept-Encoding = "gzip";
     } elsif (req.http.Accept-Encoding ~ "deflate") {
         set req.http.Accept-Encoding = "deflate";
     } else {
         # unkown algorithm
         remove req.http.Accept-Encoding;
     }
}
```

这段代码设置客客户端发送的accept-encoding头只有gzip和default两种编码， gzip优先。
**Pitfall – Vary：User-Agent**
一些应用或者一些应用服务器发送不同 user-agent 头信息，这让 varnish 为每个单独的用户保存一个单独的信息，这样的信息很多。一个版本相同的浏览器在不同的操作系统上也会产生最少10 种不同的 user-agent头信息。如果您不打算修改 user-agent，让他们标准化，您的命中率将受到严重的打击，使用上面的代码做模板。
**Pragma**
http 1.0 服务器可能会发送“Pragma：nocache”。 Varnish忽略这个头，您可以轻松的使用VCL 来完成这个任务：

```
In vcl_fetch：
if (beresp.http.Pragma ~ "nocache") {
    pass;
}
```

**Authorization**
如果varnish收到一个认证请求的头，他将 pass这个请求，如果您不打算对这个头做任何操作的话。

**Normalizing your namespace**
有些站点访问的主机名有很多，比如 http://www.varnish-software.com，http://varnish-software.com，http://varnishsoftware.com 所有的地址都对应相同的一个站点。但是varnish 不知道，varnish会缓存每个地址的每个页面。您可以减少这种情况，

通过修改web配置文件或者通过以下 VCL：

```
if (req.http.host ~ "^(www.)?varnish-?software.com") {
   set req.http.host = "varnish-software.com";
}
```

**Purging**
增加TTL 值是提高命令率的一个好方法，如果用户访问到的内容是旧的，这样就会对您的商务照成影响。
解决方法就是当有新内容提供的时候通知 varnish。 可以通过两种机制HTTP purging和 bans。首先，我们来看一个清理的实例：

**HTTP purges**
HTTP purges和HTTP GET请求相似，除了这是用来purges的。事实上您可以在任何您喜欢的时间使用这个方法，但是大多数人使用它 purging。Squid支持相同的机制，为了让varnish支持purging，您需要在VCL 中做如下配置：

```
acl purge {
         "localhost";
         "192.168.55.0/24";
}
sub vcl_recv {
         # allow PURGE from localhost and 192.168.5
         if (req.request == "PURGE") {
                          if (!client.ip ~ purge) {
                         error 405 "Not allowed.";
                 }
                 return (lookup);
         }
}

sub vcl_hit {
         if (req.request == "PURGE") {
                 # Note that setting ttl to 0 is magical.
                  # the object is zapped from cache.
                 set obj.ttl = 0s;
                 error 200 "Purged.";
         }
}

sub vcl_miss {
         if (req.request == "PURGE") {

                 error 404 "Not in cache.";
         }
}
```

您可以看见，使用了新的VCL 子程序，vcl\_hit和vcl\_miss。当您调用 lookup时将在缓存中查找目标，结果只会是 miss或者 hit，然后对应的子程序就会被调用，如果 vcl_hit的目标存储在缓存中，并且可用，我们可以修改 TTL 值。
所以对于vg.no 的无效首页，他们使用varnish做如下处理：
PURGE / HTTP/1.0
Host: vg.no
如果 varnish 想要丢弃主页，如是很多相同 URL 的变体在 cache 中，只有匹配的变体才会被清除。清除一个相同页面的 gzip变体可以使用下面命令：

PURGE / HTTP/1.0Host: vg.no
Accept-Encoding: gzip

**Bans**
这是另外一种清空无效内容的方法，bans。您可以认为 bans 是一种过滤方法，您可以禁止某些存在 cache中存在的数据。您可以基于我们拥有的元数据来禁止。
Varnish内置的CLI接口就是支持bans的。禁止vg网站上的所有png目标代码如下：
purge req.http.host == “vg.no” && req.http.url ~ “\.png$”
是不是很强大？
在没有被 bans 命中之前的 cache，是能够提供服务的。一个目标只被最新的bans检查。如果您有很多长TTL 的目标在缓存中，您需要知道执行很多的Bans对性能照成的影响。
您也可以在varnish 中添加bans，这样做需要一点 VCL：

```
sub vcl_recv {
         if (req.request == "BAN") {
                 # Same ACL check as above:
                 if (!client.ip ~ purge) {
                         error 405 "Not allowed.";
                 }
                 purge("req.http.host == " req.http.host
                       "&& req.url == " req.url);
                 # Throw a synthetic page so the
                 # request wont go to the backend.
                 error 200 "Ban added"
         }
}
```

这是一个实用varnish的VCL 处理ban的方法。添加一个 ban在 URL 上，包含它的主机部分。