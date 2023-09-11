---
title: Misbehaving servers（服务器停止运转）
author: admin
type: post
date: 2011-11-21T05:24:26+00:00
url: /archives/12046
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - varnish

---
Varnish的一个关键特色就是它有能力防御 web和应用服务器宕机。
**Grace mode**
当几个客户端请求同一个页面的时候，varnish只发送一个请求到后端服务器，然后让那个其他几个请求挂起等待返回结果，返回结果后，复制请求的结果发送给客户端。
如果您的服务每秒有数千万的点击率，那么这个队列是庞大的，没有用户喜欢等待服务器响应。为了使用过期的 cache 给用户提供服务，我们需要增加他们的 TTL，保存所有cache 中的内容在 TTL过期以后30 分钟内不删除，使用以下VCL：

```
sub vcl_fetch {
   set beresp.grace = 30m;
}
```

```
Varnish 还不会使用过期的目标给用户提供服务，所以我们需要配置以下代码，在cache过期后的15 秒内，使用旧的内容提供服务：
```

```
sub vcl_recv {
   set req.grace = 15s;
}
```

你会考虑为什么要多保存过去的内容 30 分钟？当然，如果你使用了健康检查，你可以通过健康状态设置保存的时间：

```
if (! req.backend.healthy) {
    set req.grace = 5m;
} else {
    set req.grace = 15s;
}
```

**Saint mode**
有时候，服务器很古怪，他们发出随机错误，您需要通知 varnish 使用更加优雅的方式处理它，这种方式叫神圣模式（saint mode）。Saint mode允许您抛弃一个后端服务器或
者另一个尝试的后端服务器或者 cache中服务陈旧的内容。让我们看看 VCL中如何开启这个功能的：

```
sub vcl_fetch {
   if (beresp.status == 500) {
     set beresp.saintmode = 10s;
     restart;
   }
   set beresp.grace = 5m;
}
```

当我们设置 beresp.saintmode 为 10 秒，varnish 在 10 秒内将不会访问后端服务器的这个 url。如果有一个备用列表，当重新执行此请求时您有其他的后端有能力提供此服务内容，varnish会尝试请求他们，当您没有可用的后端服务器，varnish将使用它过期的 cache提供服务内容。
它真的是一个救生员。
**God mode**
还未应用。