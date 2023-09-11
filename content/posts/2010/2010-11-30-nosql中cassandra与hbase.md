---
title: NOSQL中Cassandra与HBase的比较，及我们迁移系统的原因
author: admin
type: post
date: 2010-11-30T03:01:36+00:00
url: /archives/6792
IM_contentdowned:
 - 1
categories:
 - nosql
tags:
 - Cassandra
 - hbase

---
原文地址：http://ria101.wordpress.com/2010/02/24/hbase-vs-cassandra-why-we-moved

HBase vs Cassandra: why we moved

下文中将讨论为何选择Cassandra作为我们的NOSQL方案。

**是否****Cassandra****的血统预言了未来？******

我发现在软件问题上，我们先去考虑上层问题而不是直接深入到细节，可以节约大量时间。在选择HBase还是Cassandra上我也遵循了这一信条。HBase还是Cassandra具有完全不同的血统和基因，这决定了他们在我们应用的可行性。

HBase及其支持系统源自Google的GFS和BigTable设计；而最初由Facebook开源出来的Cassandra采用了BigTable的数据模型，确实是用类似Amozon的Dynamo的存储系统（实际上Cassandra最初的开发工作都是由两个原Dynamo工程师开发的）。

在我看来，他们的根源决定了HBase更适用于数据仓库和大规模数据处理分析（比如对Web建索引），而Cassandra更适用于实时事务处理和处理交互性数据。（HBase的committers被MS Bing收购，Cassandra的committers则为Rackspace工作，后者旨在提供Google，Yahoo和Amazon之外的通用NOSQL方案）

**哪个****NOSQL****数据库势头更好？******

另一个考虑因素是因为Cassandra在社区中目前具有极好的发展势头。软件平台往往是越大就容易更大，人们都喜欢使用有更好支持的系统。

当开始关注HBase，我的感觉是它的背后有很好的社区支持（主要因为StumpleUpon and Streamy的CTO的报告和“HBase vs Cassandra: NoSQL Battle!”），但是我现在相信Cassandra将会比它更加强大。

为了证明这一点，可以参考IRC上开发者的活动：当连接到freenode.org

比较#hbase和#cassandra的开发者频道，你会发现Cassandra的开发者是前者的两倍。

并且，Twitter也打算大规模使用Cassandra。

**CAP****：****CA vs AP**

根据Eric Brewer教授的CAP理论，在大型分布式系统设计中，C（一致性）A（可用性）P（网络分区容忍性，即让集群被分成多个孤立的分区时系统仍然可用）不能同时满足。普遍认为HBase选择了CP而Cassandra选择了AP。

但是我必须提醒一下这种管理是基于一个不合逻辑的推论。虽然CAP不能同时满足，但是在一个系统中却可以让每个操作去指定它也选择哪两个放弃哪一个，或者对于CAP分别有什么程度的关注、在中间获得一个自己需要的平衡。这就是Cassandra所做的。

我要反复重申Cassandra的这一优点：你可以为每一个操作去选择trade-off。例如，当需要读操作具有高一致性时，就使用“ALL”这个一致性级别（译者注：实际上，因为临时故障的存在，写时可能写到了Hint点，即使使用ALL也不一定能读到期望的）；当我对一致性没有高要求而要求性能，就使用“ONE”这个一致性结果。并且，你还可以选择介于这两者之间的一致性级别，比如“QUORUM”（表决，即多数）。

并且，当一些节点失效时，或者网络抖动时，使用Cassandra仍然能保证除部分要求极高一致性的请求失败外，大部分操作可用。HBase则做不到这样的灵活性。

**什么时候****monolithic****优于****modular**

一个重要的差别是每个Cassandra节点是单个Java进程；而完整的HBase方案则由多个部分组成：运行在多个模式的数据库进程，Hadoop HDFS和ZooKeeper系统。

对于小公司来说，HBase的方案的配置过于复杂。如果是个数据库管理员想学习NOSQL系统，HBase则是个不错的选择。

**Gossip!**

Cassandra是完全的对称系统，系统中没有像HBase那样有管理节点存在，系统中的所有节点承担完全相同的作用。系统中协调的作用完全有集群中节点相互按照纯粹P2P协议Gossip来完成。Cassandra依靠这种协议来检测节点故障,或者路由请求到合适节点处理,所花费的时间相当小。

这种基于Gossip的架构给用户带来如下好处：首先，系统的管理极其简单。例如要添加一个新节点，该节点会自己和seed节点通信完成引导（bootstrapping）过程，做好数据和路由信息的准备。并且，这种P2P架构带来了好的性能和可用性。负载可以很好的在系统内均衡，对于网络出现分区故障或者节点故障可以无缝的解决，这种完全对称性也避免了HBase再加入/移除节点时会出现的那种临时性能不稳定。

**第三方报告******

Yahoo对NOSQL系统进行了较为详细的比较，研究结果表明Cassandra更有优势。HBase仅在Range scan上比较有优势。但是我认为实际上应该在Cassandra的基础上再实现你自己的索引，而不是直接用Range scan。如果你对Cassandra的区间查询和存储索引感兴趣，参考我另一篇http://ria101.wordpress.com/2010/02/22/cassandra-randompartitioner-vs-orderpreservingpartitioner/。

下面是相关的报告：

http://nosql.mypopescu.com/post/407159447/cassandra-twitter-an-interview-with-ryan-king

**锁和模块性******

你可能听HBase阵营说过他们的复杂架构可以提供Cassandra的P2P架构所不能提供的好处，比如行锁。但是我要说的是模块性。Cassandra实现了BigTable的数据模型但使用了所有点对称的分布式模型。这是很灵活很高效的一个模型。如果你需要锁、事务或者其他功能，你可以通过添加自己的模块来实现——比如我们就在Cassandra中配合使用Zookeeper来实现scalable locking。

需要锁，自己用Zookeeper；需要索引，自己用Lucandra…Cassandra没有强加可能用不到的复杂性，而是提供了灵活性让你可以自己添加自己需要的模块来完成功能。

**MapReduce**

Cassandra的一个突出弱点是在于MapReduce。而HBase因为使用的是Hadoop HDFS存储数据，天生就为MapReduce这种分析处理设计。如果你需要这种数据分析，HBase目前确实是最好的选择。

虽然我在这里大肆吹捧Cassandra，我必须指出HBase和Cassandra并不是切地的竞争者，实质上他们又更适合的场景。据我所知，StumbleUpon使用HBase极其Hadoop MapReduce来处理庞大的post。我们的系统更多的是交互应用，因此我们选择Cassandra。

Cassandra从0.6开始支持hadoop，相信其MapReduce支持会越来越好。

**丢数据？******

通过CAP的争论，容易产生这样的印象：HBase比Cassandra更安全。实际上在Cassandra中，当你写入新数据他会立即写到commit log并复制到其他节点，这使得及时你的集群系统断电，也只是损失小量数据。并且，Cassandra还利用Merkle树来发现副本间数据不一致问题，进一步提升数据安全