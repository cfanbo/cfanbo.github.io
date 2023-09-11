---
title: RabbitMQ中的ack介绍
author: admin
type: post
date: 2014-08-02T09:59:46+00:00
url: /archives/15285
categories:
 - 系统架构
tags:
 - RabbitMQ

---
no_ack 的用途：确保 message 被 consumer “成功”处理了。这里“成功”的意思是，（在设置了 no_ack=false 的情况下）只要 consumer 手动应答了 Basic.Ack ，就算其“成功”处理了。

对于ack简单的说就是“消费者”先从queue里读取一条数据，然后去处理，等处理完了，再给queue一个 ack 回应，表示处理完了，这时queue就将这条数据从队列中删除。如果不回应给队列ack的话，则这条消息仍然存在在queue中（这个也属于一种应用场景）。

- 在 no_ack=true 的情况下，RabbitMQ 认为 message 一旦被 deliver 出去了，就已被确认了，所以会立即将缓存中的 message 删除。所以在 consumer 异常时会导致消息丢失。

- _**no_ack=false**（此时为 **手动应答**）_

   在这种情况下，要求 consumer 在处理完接收到的 Basic.Deliver + Content-Header + Content-Body 之后才回复 Ack 。而这个 Ack 是 AMQP 协议中的 Basic.Ack 。此 Ack 的回复是和业务处理相关的，所以具体的回复时间应该要取决于业务处理的耗时。


如果Ack 未被 consumer 发回给 RabbitMQ 前出现了异常，RabbitMQ 发现与该 consumer 对应的连接被断开，之后将该 message 以轮询方式发送给其他 consumer （假设存在多个 consumer 订阅同一个 queue）。

参考： [http://www.kankanews.com/ICkengine/archives/8904.shtml](http://www.kankanews.com/ICkengine/archives/8904.shtml)

对于ack的介绍可以参考：