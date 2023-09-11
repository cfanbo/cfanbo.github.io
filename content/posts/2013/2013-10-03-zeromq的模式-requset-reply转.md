---
title: 'ZeroMQ的模式-Requset-Reply[转]'
author: admin
type: post
date: 2013-10-03T14:08:59+00:00
url: /archives/14518
categories:
 - 程序开发
tags:
 - zeromq

---
我们先来看看第一种模式:**Request-Reply Pattern**。 请求应答模式。

Request-Reply这个名字很直白，口语点说就是**一问一答**。可以使同步的遵循请求序的一问一答，也可以是异步的不按请求序的一问一答；其中也可以包含各种不同的路由策略——让谁来回答。zeromq定义的为这个模式服务的socket有：ZMQ\_REQ, ZMQ\_REP, ZMQ\_ROUTER以及ZMQ\_DEALER. 用他们进行合理的组合，就可以实现现实世界中各种不同的请求应答模式。

分别来看：

# ZMQ_REQ

ZMQ_REQ做的事情就是**发问**，然后**收答**。发、收必须是严格按序进行。请求时对对端进行Round Robin，遇到异常则阻塞。官方对这个socket的总结如下：

 Summary of ZMQ_REQ characteristics

 Compatible peer sockets
 _ZMQ_REP_
 Direction

 Bidirectional

 Send/receive pattern

 Send, Receive, Send, Receive, …

 Outgoing routing strategy

 Round-robin

 Incoming routing strategy

 Last peer

 ZMQ_HWM option action

 Block


# ZMQ_REP

ZMQ_REP做的事情是**收问题**然后**回答**。收、发严格按序调用。对收到的问题公平排队,逐一作答。回答发出时遇到异常则直接丢弃，不会阻塞。

 Summary of ZMQ_REP characteristics

 Compatible peer sockets
 _ZMQ_REQ_
 Direction

 Bidirectional

 Send/receive pattern

 Receive, Send, Receive, Send, …

 Incoming routing strategy

 Fair-queued

 Outgoing routing strategy

 Last peer

 ZMQ_HWM option action

 Drop


**可以发现，上述两种socket是纯同步的，连用在它们身上的api的调用顺序都有严格定义。而且没有办法动态决定请求的去向。如果要实现更复杂的请求应答模式，就要借助于下面两种socket了。**

# ZMQ_DEALER

其实ZMQ\_DEALER也是同步的，ZMQ\_DEALER也叫作ZMQ_XREQ，概念上是request/reply socket的扩展(实现上刚好相反哦)。从名字上可知它是一个代理层。它对收到的消息公平排队，并以RR方式发送消息，在遇到异常时发送阻塞。它的主要作用是将come in的请求load balance地发送给到对端们。

 Summary of ZMQ_DEALER characteristics

 Compatible peer sockets
 _ZMQ_ROUTER_, _ZMQ_REQ_, _ZMQ_REP_
 Direction

 Bidirectional

 Send/receive pattern

 Unrestricted

 Outgoing routing strategy

 Round-robin

 Incoming routing strategy

 Fair-queued

 ZMQ_HWM option action

 Block


#### {#toc7}

# ZMQ_ROUTER

ZMQ\_ROUTER是真正的异步。ZMQ\_ROUTER socket收到消息时会在消息栈上加一层包含消息来源地址的消息；发送消息时，会将这一层消息取出，将其作为发送的目的地。如果发送时遇到异常，则丢弃消息。ZMQ_ROUTER通过这种方式做到了不需要保存任何状态便可异步地转发消息，而这一切应用层是看不到的。

 Summary of ZMQ_ROUTER characteristics

 Compatible peer sockets
 _ZMQ_DEALER_, _ZMQ_REQ_, _ZMQ_REP_
 Direction

 Bidirectional

 Send/receive pattern

 Unrestricted

 Outgoing routing strategy

 See text

 Incoming routing strategy

 Fair-queued

 ZMQ_HWM option action

 Drop


# 总结

归纳总结一下，0mq的Request-Reply模式下有四种socket类型：

 * DEALER： 给连接的对端RR地分发消息，对收到的消息公平排队。
 * REQ：在应用层外发的消息上加一层空消息再发送；在收到消息后去掉分隔空消息再返回应用层。它实质上是在DEALER上构建的，只是在此基础上强制加入了发、收循环。
 * ROUTER：针对每一个收到的消息：加一层来源地址段，然后再交给应用层；针对每一个要发出的消息：去掉最上层的地址段，并以该地址段为目的地进行发送。
 * REP：对收到的消息：储存所有的消息内容直到第一个分隔空消息段，把剩余的消息体传给应用层；对发出的消息：把之前储存的消息体加回来，并像ROUTER一样发送出去。它实质上实在ROUTER上构建的，只是在此基础上强制加入了收、发循环。

有了这几种socket，便可以组合成各种Request-Reply模式了：

> [REQ] <--> [REP]
> [REQ] <--> [ROUTER--DEALER] <--> [REP]
> [REQ] <--> [ROUTER--DEALER] <--> [ROUTER--DEALER] <--> [REP]
> ...

举个例子，N个client要给M个worker提交任务，worker完成任务后返回给client；worker需要平衡负载以避免某一worker过于劳累，client发出的请求也有先后之说，不能让后发的请求先于之前的被交给worker。这番描述的socket组合便可如下图(from the guide)方式构建：

[![zeromq-request-reply](http://blog.haohtml.com/wp-content/uploads/2013/10/zeromq-request-reply.png)][1]

更多查看： [http://www.kongch.com/tag/zeromq/](http://www.kongch.com/tag/zeromq/)

 [1]: http://blog.haohtml.com/wp-content/uploads/2013/10/zeromq-request-reply.png