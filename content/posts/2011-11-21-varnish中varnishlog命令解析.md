---
title: varnish中Varnishlog命令解析
author: admin
type: post
date: 2011-11-21T04:03:54+00:00
url: /archives/12014
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - varnish
 - Varnishlog

---
Varnish一个真正的特点就是它如何记录数据的。使用内存段代替普通的日志文件，当内存段使用完以后，又从头开始，覆盖最旧的记录。这样就可以非常快的记录数据，并且不需要磁盘空间。缺点就是您没有把数据写到磁盘上，可能会消失。 （varnish也支持将数据写到硬盘的文件上，看您如何选择）
Varnishlog 这个程序可以查看 varnish 记录了哪些数据。Varnishlog 给您生成原始的日志，包括所有的事件。我一会给您演示。
在运行了varnish的终端窗口上，运行varnishlog这个命令。
您可以看见如下显示

> #./varnishlog
> 0 CLI – Rd ping
> 0 CLI – Wr 200 PONG 1277172542 1.0

这是检查varnish的主进程是否正常，如果看见这就说明一切OK.


现在您去浏览器通过 varnish 重新访问您的 web程序，您将看到如下信息：

```
11 SessionOpen  c 127.0.0.1 58912 0.0.0.0:8080
11 ReqStart     c 127.0.0.1 58912 595005213
11 RxRequest    c GET
11 RxURL        c /
11 RxProtocol   c HTTP/1.1
11 RxHeader     c Host: localhost:8080
11 RxHeader     c Connection: keep-alive
```

第一列是任意的数，它用来定义请求，相同的号码代表相同的 HTTP传输。
第二列是信息标记，所有的日志都带有一个标记（tag） ，标记对应相对的操作。Rx
表示varnish收到数据，Tx表示varnish发送数据。
第三列代表数据是从哪里传出或者传入的，是从 c（client）还是 b（backend）。
第四列是被记录的数据。

现在，您可以过滤掉相当多的varnishlog，这些基本的选项，您需要知道：

> **varnishlog命令选项:**
>
> -b \\只显示varnish和backend server 之间的日志，当您想要优化命中率的时候可以使用这个参数。
> -c \\和-b差不多，不过它代表的是 varnish和 client端的通信。
> -i tag \\只显示某个 tag，比如“varnishlog –i SessionOpen”将只显示新会话，注意，这个地方的tag名字是不区分大小写的。
> -I \\通过正则表达式过滤数据，比如“varnishlog -c -i RxHeader -I Cookie”将显示所有接到到来自客户端的包含 Cookie 单词的头信息。
> -o \\聚合日志请求 ID

如果varnish一切运行 OK，我们就可以把它调整到80端口上。