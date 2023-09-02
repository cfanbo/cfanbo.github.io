---
title: 增加Apache2和Nginx的header长度限制
author: admin
type: post
date: 2012-09-05T15:08:06+00:00
url: /archives/13377
categories:
 - 服务器
tags:
 - apache
 - nginx

---
nginx默认的header长度上限是4k，如果超过了这个值
nginx会直接返回400错误

> [error] 16613#0: *105 upstream sent too big header while reading response header from upstream

可以通过以下2个参数来调整header上限

> client\_header\_buffer_size 16k;
> large\_client\_header_buffers 4 16k;

看起来是，nginx默认会用client\_header\_buffer\_size这个buffer来读取header值，如果header过大，它会使用large\_client\_header\_buffers来读取

client_header_buffer_size

syntax: client_header_buffer_size size

default: 1k

context: http, server

Directive sets the headerbuffer size for the request header from client.

For the overwhelming majority of requests it is completely sufficient a buffer size of 1K.

However if a big cookie is in the request-header or the request has come from a wap-client the header can not be placed in 1K, therefore, the request-header or a line of request-header is not located completely in this buffer nginx allocate a bigger buffer, the size of the bigger buffer can be set with the instruction large_client_header_buffers.


large_client_header_buffers

syntax: large_client_header_buffers number size

default: large_client_header_buffers 4 4k/8k

context: http, server

Directive assigns the maximum number and size of buffers for large headers to read from client request.

The request line can not be bigger than the size of one buffer, if the client send a bigger header nginx returns error “Request URI too large” (414).

The longest header line of request also must be not more than the size of one buffer, otherwise the client get the error “Bad request” (400).

Buffers are separated only as needed.

By default the size of one buffer is equal to the size of page, depending on platform this either 4K or 8K, if at the end of working request connection converts to state keep-alive, then these buffers are freed.


**参考：** [http://www.pagefault.info/?p=220](http://www.pagefault.info/?p=220)

—————————————————————————————


对于apache2来说，它默认值是8k
可以调整以下2个参数

> LimitRequestLine 16k
> LimitRequestFieldSize 16k
> LimitRequestLine 指令设置的是每一个header长度的上线

LimitRequestLine 指令

说明 限制接受客户端发送的HTTP请求行的字节数

语法 LimitRequestLine bytes

默认值 LimitRequestLine 8190

作用域 server config

状态 核心(C)

模块 core

bytes将设置HTTP请求行的字节数限制。

LimitRequestLine指令允许服务器管理员增加或减少客户端HTTP请求行允许大小的限制。因为请求行包括HTTP方法、URI、协议版本，所以LimitRequestLine指令会限制请求URI的长度。服务器会需要这个值足够大以装载它所有的资源名，包括可能在GET请求中所传递的查询部分的所有信息。

这个指令给了服务器管理员更大的可控性以控制客户端不正常的请求行为。这有助于避免某些形式的拒绝服务攻击。


LimitRequestFieldSize指令设置的是所有header总长度的上限值

LimitRequestFieldSize 指令

说明 限制客户端发送的请求头的字节数

语法 LimitRequestFieldsize bytes

默认值 LimitRequestFieldsize 8190

作用域 server config

状态 核心(C)

模块 core

bytes指定了HTTP请求头允许的字节大小。

LimitRequestFieldSize指令允许服务器管理员增加或减少HTTP请求头域大小的限制。一般来说，服务器需要此值足够大，以适应普通客户端的任何请求的头域大小。一个普通头域的大小对于不同的客户端来说是有很大差别的，一般与用户配置他们的浏览器以支持更多的内容协议密切相关。SPNEGO的认证头最大可能达到12392字节。

这个指令给了服务器管理员更大的可控性以控制客户端不正常的请求行为。这有助于避免某些形式的拒绝服务攻击。


**参考：** [http://lamp.linux.gov.cn/Apache/ApacheMenu/mod/core.html](http://lamp.linux.gov.cn/Apache/ApacheMenu/mod/core.html)