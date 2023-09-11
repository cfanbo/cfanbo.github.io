---
title: meta http-equiv=”Cache-Control” content=”max-age=0″
author: admin
type: post
date: 2010-05-29T16:49:42+00:00
url: /archives/3716
categories:
 - 前端设计
tags:
 - 网页优化

---
**Cache-Control头域**
Cache-Control指定请求和响应遵循的缓存机制。在请求消息或响应消息中设置Cache- Control并不会修改另一个消息处理过程中的缓存处理过程。请求时的缓存指令包括no-cache、no-store、max-age、max- stale、min-fresh、only-if-cached，响应消息中的指令包括public、private、no-cache、 no-store、no-transform、must-revalidate、proxy-revalidate、max-age。各个消息中的指令含 义如下

Public指示响应可被任何缓存区缓存

Private指示对于单个用户的整个或部分响应消息，不能被共享缓存处理。这允许服务器 仅仅描述当用户的部分响应消息，此响应消息对于其他用户的请求无效

no-cache指示请求或响应消息不能缓存

no-store用于防止 重要的信息被无意的发布。在请求消息中发送将使得请求和响应消息都不使用缓存。

max-age指示客户机可以接收生存期不大于指定时间（以秒为单 位）的响应

min-fresh指示客户机可以接收响应时间小于当前时间加上指定时间的响应

max-stale指示客户机可以接收超出超时 期间的响应消息。如果指定max-stale消息的值，那么客户机可以接收超出超时期指定值之内的响应消息。