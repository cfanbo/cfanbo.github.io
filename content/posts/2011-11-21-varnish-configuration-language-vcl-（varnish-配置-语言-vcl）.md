---
title: Varnish Configuration Language – VCL （varnish 配置 语言-VCL）
author: admin
type: post
date: 2011-11-21T04:17:34+00:00
url: /archives/12024
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - varnish
 - vcl

---
官方手册:

**    **Varnish 有一个很棒的配置系统，大部分其他的系统使用配置指令，让您打开或者关闭一些开关。 Varnish使用区域配置语言，这种语言叫做“VCL”（varnish configuration language），在执行vcl时，varnish 就把VCL转换成二进制代码。
**    **VCL 文件被分为多个子程序，不同的子程序在不同的时间里执行，比如一个子程序在接到请求时执行，另一个子程序在接收到后端服务器传送的文件时执行。
varnish 将在不同阶段执行它的子程序代码，因为它的代码是一行一行执行的，不存在优先级问题。随时可以调用这个子程序中的功能并且当他执行完成后就退出。


**    **如果到最后您也没有调用您的子进程中的功能，varnish 将执行一些内建的 VCL代码，这些代码就是default.vcl 中被注释的代码.

** 99%的几率您需要改变vcl\_recv 和 vcl\_fetch 这两个子进程。**

**vcl_recv**
**    **vcl\_recv（当然，我们在字符集上有点不足，应为它是unix）在请求的开始被调用，在接收、解析后，决定是否响应请求，怎么响应，使用哪个后台服务器。在vcl\_recv中，您可以修改请求，比如您可以修改cookies，添加或者删除请求的头信息。
**    **注意vcl_recv中只有请求的目标,req is available。
**vcl_fetch**
**    **vcl_fetch 在一个文件成功从后台获取后被调用，通常他的任务就是改变 response headers，触发ESI进程，在请求失败的时候轮询其他服务器。
在 vcl_fetch 中一样的包含请求的 object，req，available，他们通常是 backend response，beresp。beresp 将会包含后端服务器的HTTP 的头信息

**actions**
**    主要有以下动作**
pass \\当一个请求被pass后，这个请求将通过varnish转发到后端服务器，但是它不会被缓存。pass可以放在vcl\_recv 和 vcl\_fetch中。
lookup \\当一个请求在vcl\_recv中被lookup后，varnish将从缓存中提取数据，如果缓存中没有数据，将被设置为pass，不能在 vcl\_fetch中设置lookup。
pipe \\pipe和 pass相似，都要访问后端服务器，不过当进入pipe 模式后，在此连接未关闭前，后续的所有请求都发到后端服务器（这句是我自己理解后简化的，有能力的朋友可以看看官方文档，给我提修改建议） 。
deliver \\请求的目标被缓存，然后发送给客户端
esi \\ESI-process the fetched document（我理解的就是vcl 中包换一段 html代码）

**Requests，Responses and objects**
在VCL 中，有3 个重要的数据结构
**    **request 从客户端进来
**    **responses 从后端服务器过来
**    **object 存储在cache 中

    在VCL 中，你需要知道以下结构
**    **req \\请求目标，当 varnish 接收到一个请求，这时 req object 就被创建了，你在vcl_recv中的大部分工作，都是在 req object上展开的。
**    **beresp \\后端服务器返回的目标，它包含返回的头信息，你在vcl_fetch中的大部分工作都是在beresp object上开展的。
obj \\被 cache 的目标，只读的目标被保存于内存中，obj.ttl 的值可修改，其他的只能读。

**Operaors**
**VCL 支持以下运算符，请阅读下面的例子：**
= \\赋值运算符
== \\对比
~ \\匹配，在ACL中和正则表达式中都可以用
！ \\否定
&& \\逻辑与
 || \\逻辑或

=======================================================

**EXAMPLE 1 – manipulation headers**
我们想要取消我们服务器上/images目录下的所有缓存：

```
sub vcl_recv {
if (req.url ~ "^/images") {
unset req.http.cookie;
}
}
```

现在，当这个请求在操作后端服务器时，将不会有 cookie 头，这里有趣的行是if-statement，它匹配URL，如果匹配这个操作，那么头信息中的cookie就会被删除。

**EXAMPLE 2 – manipulation beresp**
从后端服务器返回对象的值满足一些标准，我们就修改它的TTL 值：

```
sub vcl_fetch {
    if (beresp.url ~ "\.(png|gif|jpg)$") {
      unset beresp.http.set-cookie;
      beresp.ttl = 3600;
   }
}
```

**EXAMPLE3-ACLs**
你创建一个VCL关键字的访问控制列表。你可以配置客户端的IP地址

```
# Who is allowed to purge....
    acl local {
         "localhost";
         "192.168.1.0"/24; /* and everyone on the local network */
         ! "192.168.1.23"; /* except for the dialin router */
    }

    sub vcl_recv {
       if (req.request == "PURGE") {
         if (client.ip ~ local) {
            return(lookup);
         }
       }
    }

    sub vcl_hit {
        if (req.request == "PURGE") { 12

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

摘自:Varnish权威指南-中文版: [http://docs.haohtml.com/download/cdn/Varnish%c8%a8%cd%fe%d6%b8%c4%cf-%d6%d0%ce%c4%b0%e6.pdf](http://docs.haohtml.com/download/cdn/Varnish%c8%a8%cd%fe%d6%b8%c4%cf-%d6%d0%ce%c4%b0%e6.pdf)