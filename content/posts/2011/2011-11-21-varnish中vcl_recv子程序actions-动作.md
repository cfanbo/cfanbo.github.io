---
title: varnish中vcl_recv子程序actions 动作
author: admin
type: post
date: 2011-11-21T08:01:56+00:00
url: /archives/12079
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - deliver
 - esi
 - lookup
 - pass
 - pipe

---
**主要有以下动作**
pass \\当一个请求被pass后，这个请求将通过varnish转发到后端服务器，但是它不会被缓存。pass可以放在vcl\_recv 和 vcl\_fetch中。
lookup \\当一个请求在vcl\_recv中被lookup后，varnish将从缓存中提取数据，如果缓存中没有数据，将被设置为pass，不能在 vcl\_fetch中设置lookup。
pipe \\pipe和 pass相似，都要访问后端服务器，不过当进入pipe 模式后，在此连接未关闭前，后续的所有请求都发到后端服务器（这句是我自己理解后简化的，有能力的朋友可以看看官方文档，给我提修改建议） 。
deliver \\请求的目标被缓存，然后发送给客户端
esi \\ESI-process the fetched document（我理解的就是vcl 中包换一段 html代码）