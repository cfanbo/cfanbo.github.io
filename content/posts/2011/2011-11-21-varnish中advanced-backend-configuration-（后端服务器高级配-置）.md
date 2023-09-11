---
title: varnish中advanced backend configuration （后端服务高级配置）
author: admin
type: post
date: 2011-11-21T05:12:37+00:00
url: /archives/12044
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - varnish

---
在某些时刻您需要 varnish 从多台服务器上缓存数据。您可能想要 varnish 映射所有的URL 到一个单独的主机或者不到这个主机。这里很多选项。
我们需要引进一个 java程序进出php的web站点。假如我们的java程序使用的 URL开始于/JAVA/

我们让它运行在8000端口，现在让我们看看默认的default.vcl：

```
backend default {
     .host = "127.0.0.1";
     .port = "8080";
}
```

我们添加一个新的 backend：

```
backend java {
     .host = "127.0.0.1";
     .port = "8000";
}
```

现在我们需要告诉特殊的URL 被发送到哪里：

```
sub vcl_recv {
     if (req.url ~ "^/java/") {
         set req.backend = java;
     } else {
         set req.backend = default.
     }
}
```

这真的很简单，让我们停下来并思考一下。正如您所见，可以通过任意的后端来选择您要的数据。您想发送移动设备的请求到不同的后端？没问题

> if (req.User-agent ~ /mobile/) …. \\这样做应该就可以成功。