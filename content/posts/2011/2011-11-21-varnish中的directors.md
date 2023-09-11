---
title: varnish中的Directors
author: admin
type: post
date: 2011-11-21T05:16:15+00:00
url: /archives/12049
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - Directors
 - varnish

---
您可以把多台 backends 聚合成一个组，这些组被叫做 directors。这样可以增强性能和弹力。您可以定义多个 backends和多个 group在同一个directors。

```
backend server1 {
     .host = "192.168.0.10";
}
backend server2{
     .host = "192.168.0.10";
}
```

现在我们创建一个 director：

```
director example_director round-robin {
{
         .backend = server1;
}
# server2
{
         .backend = server2;
}
# foo
}
```

这个 director 是一个循环的 director。它的含义就是 director 使用循环的方式把backends分给请求。
但是如果您的一个服务器宕了？varnish 能否指导所有的请求到健康的后端？当然可以，这就是健康检查在起作用了。