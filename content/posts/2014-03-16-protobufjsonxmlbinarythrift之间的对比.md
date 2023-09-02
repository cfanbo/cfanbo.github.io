---
title: protobuf,json,xml,binary,Thrift之间的对比
author: admin
type: post
date: 2014-03-16T13:39:22+00:00
url: /archives/14917
categories:
 - 程序开发
tags:
 - ProtoBuf

---
一条消息数据，用protobuf序列化后的大小是json的10分之一，xml格式的20分之一，是二进制序列化的10分之一，总体看来ProtoBuf的优势还是很明显的。

protobuf是google提供的一个开源序列化框架，类似于XML，JSON这样的数据表示语言，详情访问protobuf的google官方网站 [https://code.google.com/p/protobuf/](https://code.google.com/p/protobuf/)。

protobuf在google中是一个比较核心的基础库，作为分布式运算涉及到大量的不同业务消息的传递，如何高效简洁的表示、操作这些业务消息在google这样的大规模应用中是至关重要的。而protobuf这样的库正好是在效率、数据大小、易用性之间取得了很好的平衡。

**protobuf简单总结如下几点：**

1.灵活（方便接口更新）、高效（效率经过google的优化，传输效率比普通的XML等高很多）；

2.易于使用；开发人员通过按照一定的语法定义结构化的消息格式，然后送给命令行工具，工具将自动生成相关的类，可以支持java、c++、python等语言环境。通过将这些类包含在项目中，可以很轻松的调用相关方法来完成业务消息的序列化与反序列化工作。

3.语言支持；原生支持c++,java,python

**个人总结的适用protobuf的场合：**

1.需要和其它系统做消息交换的，对消息大小很敏感的。那么protobuf适合了，它语言无关，消息空间相对xml和json等节省很多。
2.小数据的场合。如果你是大数据，用它并不适合。
3.项目语言是c++,java,python的，因为它们可以使用google的源生类库，序列化和反序列化的效率非常高。其它的语言需要第三方或者自己写，序列化和反序列化的效率不保证。
4.总体而言，protobuf还是非常好用的，被很多开源系统用于数据通信的工具，在google也是核心的基础库。

此外，还有更牛叉的facebook的thrift，2007年由Facebook开发，之后在2008年加到Apache计划中。是一个跨语言的轻量级RPC消息和数据交换框架，Thrift能生成的语言有: C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, Smalltalk, and OCaml，这是它的一大优点。