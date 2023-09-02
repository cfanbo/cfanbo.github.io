---
title: varnish中的Health checks(健康检查)
author: admin
type: post
date: 2011-11-21T05:19:22+00:00
url: /archives/12053
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - varnish

---
让我们设置一个 director和两个后端，然后加上健康检查：

```
backend server1 {
   .host = "server1.example.com";
   .probe = {
          .url = "/";
           .interval = 5s;
          .timeout = 1 s;
          .window = 5;
          .threshold = 3;
     }
   }
backend server2 {
    .host = "server2.example.com";
    .probe = {
          .url = "/";
          .interval = 5s;
          .timeout = 1 s;
          .window = 5;
          .threshold = 3;
    }
  }
```

这些新的就是探针，varnish将检查通过探针检查每个后端服务器是否健康：


url \\哪个 url需要varnish请求。
Interval \\检查的间隔时间
Timeout \\等待多长时间探针超时
Window \\varnish将维持5个 sliding window的结果
Threshold \\至少有3 次.windows检查是成功的，就宣告 backends健康

现在我们定义 director：

```
director example_director round-robin {
       {
               .backend = server1;
       }
       # server2
       {
               .backend = server2;
       }
}
```

您的站点在您需要的时候使用这个director，varnish不会发送流量给标志为不健康的主机。如果所有的 backends 都宕掉了，varnish 可以照常服务。参照“Misbehaving servers”获得更多的信息。

官方手册:[https://www.varnish-cache.org/docs/3.0/tutorial/advanced\_backend\_servers.html#health-checks][1]

 [1]: https://www.varnish-cache.org/docs/3.0/tutorial/advanced_backend_servers.html#health-checks