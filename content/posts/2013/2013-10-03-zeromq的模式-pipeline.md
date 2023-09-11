---
title: 'ZeroMQ的模式-Pipeline[转]'
author: admin
type: post
date: 2013-10-03T14:02:23+00:00
url: /archives/14512
categories:
 - 程序开发
tags:
 - zeromq

---

**Pipeline pattern** 管道模式。

这种模式描述的场景是数据被散布到以管道方式组织的各个节点上。管道的每一步都连接一个或多个节点，连接多个节点时数据以RR方式往下流。

注意是**流**，意味着数据跟发布模式一样是单向的。这个模式对应的socket是ZMQ\_PUSH和ZMQ\_PULL.

# ZMQ_PUSH

用来向下游节点发消息。下游多个节点时采取RoundRobin分发，_zmq_recv()_对于这个socket也是无效的。

与Pub不同的是,当下游节点达到高水位(HWM)或者根本没有下游节点时，_zmq_send()_就阻塞了，消息并不丢失。

 Summary of ZMQ_PUSH characteristics

 Compatible peer sockets
 _ZMQ_PULL_
 Direction

 Unidirectional

 Send/receive pattern

 Send only

 Incoming routing strategy

 N/A

 Outgoing routing strategy

 Round-robin

 ZMQ_HWM option action

 Block


# ZMQ_PULL

下游节点在这个socket上进行_zmq_recv()_，来收取上游发来的消息。_zmq_send()_在此socket上是没有意义的。

 Summary of ZMQ_PULL characteristics

 Compatible peer sockets
 _ZMQ_PUSH_
 Direction

 Unidirectional

 Send/receive pattern

 Receive only

 Incoming routing strategy

 Fair-queued

 Outgoing routing strategy

 N/A

 ZMQ_HWM option action

 N/A


# 总结

流行的map-reduce可以说就是这样的模式。数据从头开始，map到许多节点进行计算，计算结果最终reduce到一处。单向，没有回头。

事实上，这种模式也多见于并行计算、分布式计算这些场景中。

这个模式跟pub-sub一样容易理解，因此也没必要再赘述了。

转自： [http://www.kongch.com/2012/01/zeromq-pattern-pipeline/](http://www.kongch.com/2012/01/zeromq-pattern-pipeline/)