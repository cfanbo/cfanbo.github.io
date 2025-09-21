---
title: 理解 Solana 中的 Slot、Block、Epoch、Entry、Shred 和 PoH
date: 2025-06-10T09:32:00+08:00
type: post
url: /posts/understanding-slots-block-epoch-entry-and-shred-on-solana
toc: true
categories:
- 程序开发
- web3
tags:
- solana
- web3
---

对于刚刚接触Solana的开发者来讲，会经常遇到 Slot Block Epochs Entry 和 Shred等一些概念术语，但很验证记住它们的意思和作用。

本篇将通过 Solana 的交易原理，让大家轻记信它们并了解它们之间的联系，并加深对Solana交易的理解。由于能力有限，文章中难免有误，大家可以在评论区留言，共同学习进步！

好了，下面让我们开始彻底搞明白这些术语及它们在Solana交易中的作用吧！

在开始之前我们先看一下在Soalna链中一笔交易是如何进行的。

## 交易流程

![Visualization of where Turbine lies in the lifecycle of a Solana transaction](https://www.helius.dev/_next/image?url=%2Fapi%2Fmedia%2Ffile%2Fturbine-in-solana-transaction-lifecycle.webp&w=3840&q=90)

### **交易流程**

1. 用户客户端发起一笔交易，并使用钱包私钥进行签名
2. 通过rpc协议将交易发送到用户指定的 Solana RPC 节点。这里RPC节点在区块链网络中存在多个，起到了负载均衡的作用
3. RPC节点验证交易合法性，如进行一系列的检查 ，一般指签名验证/账户检查（账户是否存在，读取权限合法）/账户余额是否足够支付手续费等，最后进行交易模拟执行，就是我们经常说的模拟交易阶段（这一步可以手动配置 `--skip-preflight` 或 `kipPreflight: true` 跳过）。如果这一步失败，则整个交易直接失败，并返回给客户端。否则将交易转换到 Leader 节点。
4. Leader节点收到交易后，会放入自己的交易队列（Transaction Queue），按账户读写锁进行是否允许并发执行（solana交易快的原因之一就是并发执行），同时进行预检查(Pre-Check)，防止账户余额不足支付交易手续费。同时还会做更多的工作，这些工作与PoH有关，设计到slot/blocks/Epochs/Entry 和 shred，这也正是本文的重点。
5. 在区块链里一笔交易是否合法需要得到大多数节点（在solana里实际是质押量）的认可才行。因此这时 Leader 节点将交易状态设置为 **processed**。需要将这笔交易发送到一些验证节点（Validator）。如果每一次发送一笔交易的话，网络性能太差了，因此需要将多笔一起打包发送到验证节点，在Solana里一般是通过 Turbine（分层树状网络） 广播的方式将交易信息发送到 validator节点。
6. 验证节点收到交易信息后，会重放执行交易，再次对交易进行验证，如果与leader节点验证的结果一致，则表示交易是合法的，就会对LEADER节点发送一个投票交易（ **Tower BFT** ）到 leader节点。这一阶段称为投票。同时这些验证节点还需要发送到下一层的邻近 validator 节点，这些节点组织成一个类似树状网络。
7. 当Leader节点不断的收到多个验证节点的投票，并不断的计算stake数量，如果发现达到了 2/3 的，则表示交易得到了多数节点的认可，也就是表示交易是合法的，这时再将交易状态修改为 **confirmed**。
8. 后续随着新交易的不断增加，交易越来越不可能被修改，最终再将交易状态修改为 **finalized**，表示交易不可回滚动。

以上就是一笔交易的大概流程，这里涉及交易的三种状态：

- **processed**：Leader 本地执行并打包交易

- **confirmed**：交易得到网络多数 validator 的认可（>2/3 stake 投票）

- **finalized**：Slot 成为 root，交易不可回滚，真正落盘

## 术语

在学习solana底层实现原理的时候，经常会遇到Epoch/slot/block/tick/entry等概念，它们出现在上述流程中的不同步骤，下面我们介绍一下它们。



### Epoch 

在solana中，一般每隔一段时间就会重新生成一批leader，表示后续的一段时间内，这些leader用来生成区块信息。而 Epoch 就是指生成 Leader 调度表的周期，通常持续 2 天。

那么它是如何在这么多的节点中，选择哪些节点作为 leader 呢？

在每个 Epoch 开始时，Solana网络会根据**质押权重**和之前的区块**随机**选举出一个验证者(称为领导者Leader)，这个领导者序列负责在该 Epoch 内出块，领导者序列在此期间是一直保持固定不变的，直到轮到当前 Epoch 结束。

而每个领导者一般连续处理4个Sot(即**最多**出4个块 )，每个Epoch 大约持续两天(包含 86400 * 2/0.4 = 432,000个Slot)，直到下一个Epoch 重新产生领导者 Leader。

因此一个节点质押的数量越多越有机会被选择为 leader, 成为获取更多的分成。



### Slot

Slot 是验证者生成区块的最小时间单位，每个 Slot 持续约 400 毫秒。Slot 根据 Leader 调度表选择 Leader 节点来生成区块。

每一个slot 都有自己的 leader， 一般连续 四个slot 会对应同一个leader。

![image-20250911104840556](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250911104840556.png)

如图所示： `slot1~slot4` 对应的 `leader1`，而 `slot5～slot8` 对应的是 `leader2` 



另外，**slot 在 Solana 里是全局递增且连续的**，可以把它看作网络的“时钟刻度”。你可以简单的将其理解成时间里的分，有分就会有秒，而秒对应的则是 POH里的 TICK 。这一点很重要，因为它还涉及solana中的一个重要概念 POH ，要想了解它，可以参考白皮书 **[solana-whitepaper](https://solana.com/solana-whitepaper.pdf)**。

**总结：**

- 一天86400秒，而每一个slot是0.4秒，因此一个 Epoch 周期（两天）包含  86400 * 2/0.4= 432000 个slot
- slot 是全局递增且连续的，可以理解为区块链网络中的连续时钟



### PoH

PoH 是 Solana 的核心概念，它通过使用 **连续哈希链** 来提供全局的时间序列。它就像 CPU 时钟脉冲，永远在“嘀嗒嘀嗒”不停运转（计算hash）。

![image-20250911114329874](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250911114329874.png)

poh正是通过这种不停的计算hash的方式，来表示全局统一的时间序列，同时也间接的表示了交易的先后顺序。

这里直接用白皮书里的图解释，哈希 `62f51643c1` 在计数 `510144806912` 上生成，哈希 `c43d862d88` 在计数 `510146904064` 中生成。按照前面介绍的 PoH 算法的特性，我们可以相信计数 `510144806912` 和计数 `510146904064` 之间传递的实时时间。因为从头开始不停的计算hash，最后的结果肯定是 `c43d862d88`, 否则就是中间数据被篡改。

![image-20250911190417231](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250911190417231.png)

到现在为止，我们介绍了 PoH 序列机制，它一直在不停的推进，这就保证了一个顺序，就像真实世界中的时间一样。

那么当交易发生时，如何保证交易也实现类似的效果呢？其实很简单，就是将交易数据作为hash计算的参数，一并计算即可。

![image-20250911114340933](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250911114340933.png)

假设当计算 `hash335`时，有新的数据产生（如拍照交易 `photograph sha256`），则将在**下一次**进行计算 `hash336`时，会将其数据作为一个输入参数和 上一次的计算结果(hash335) 一起计算，最后得出hash336，是不是很类似于搭便车的样子呢。

> 这里值的注意的是，对于产生的额外数据（交易）一定是在下一次hash计算之前的某个时间发生的。

我们再看一个包含多个数据的例子。

![image-20250911185409685](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250911185409685.png)

这里 poH 序列里有两笔交易数据，其中 `photograph2 sha256` 是在 `hash600` 计算之前发生的，`photograph1 sha256` 则是在 `hash336` 之前发生的。如果有人修改数据的话，则会影响后续所有的值。

同样我们用白皮书里的一个图解释一下，插入交易数据后的效果。

![image-20250911191957628](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250911191957628.png)

假如我们在第一次hash后，有新的数据 `cfds0df8` 产品，则这时新计算的hash值为 `3d039eef3`,  不再是上图中的 `3d039eef3`了，同样后续计算的所有hash值也同样变化。

> 另外这里有一个 count 计数字段，你知道它是什么意思吗？建议了解一下，也是一个非常重要的知识点。提醒一下它很类似于unix时间戳的。

**总结**

- 在整个过程中，即保证了 PoH 哈希链的连续性，也实现了数据的不可篡改性

- PoH 服务一直在不停的在推进时序。有交易数据过来的话，则将数据也参与hash的计算

  

### Tick

tick 是 PoH 序列中的“心跳”，它就像物理世界中的时钟，总是在不停的滴答滴答，一直处于推进时间进度状态。

Leader 每间隔一段时间（6.25ms）就创建一个Entry，如果这个entry不包含任何交易，则为 `tick entry`，它是一个特别的 Entry。

```rust
Entry {
    hash: H1000,
    num_hashes: 1000,
    transactions: []
}
```



如果Entry 里包含交易的话，则为普通的 Entry

```rust
Entry {
    hash: H1200,
    num_hashes: 200,
    transactions: [tx1, tx2, tx3]
}
```



**总结**

- Tick 每隔 6.25ms 就心跳一次
- 每次都会生成一个entry。如果这个entry不包含交易，则为 tick Entry；否则为普通的Entry
- tick 与 tick entry 是两个完全不同的概念，但是两者存在一定的联系。在技术实现层面上，它是指 空 entry，而在我们平时描述层面上，可以将其理解为心跳



### Block

每一个block 都对应一个slot ，但反过来，并不保证每一个slot都对应一个block，因为在slot 一直推进的过程中， leader 节点可能由于网络或者宕机无法正常出块，即丢失block。

![solana slots and blocks](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/935a00_fe2b8cf87b1849959603a835b8bcecc1~mv2.jpg)

假设当前 slot = 1,000：

- 下一个一定是 1,001，再下一个是 1,002 …
- 如果这时 Leader 离线、没出块、fork 被丢弃。由于 每个节点都会每隔 400ms 产生一个slot，所以这些 slot 仍然存在且在不停的向前推进，只是当前slot没有对应的 block。

**总结**：

- block number 是全局递增且连续，它比slot 编号要小，因为slot 在一些特殊情况下有可能并不生成对应的 block
- slot在区块链所有节点都是以每400ms间隔时间推进slot，因此slot是连续不断递增的
- block 可能由于leader节点异常问题，导致无法正常出块，即只有slot，但没有对应的block

另外还有一个 `block height`，它表示当前链上确认过的block，它只有在某个block 被投票确认后才会递增，它也是全局递增，但并不是连续的。它会比slot小很多，原因仍是slot有可能并不出块。

> 另外，不要错误的理解成，没有交易就不会产生block。
>
> 在没有交易的时候，Leader 只写 tick entries（相当于空心跳），仍然会形成一个 **空块**，它仍会在区块上留下block痕迹。只有在 leader 完全离线 / 崩溃 / fork 被丢弃时，才不会产生block。

### Entry 

每一次 tick, 它就会生成一个entry， 且在tick 期间还可能会产生交易。如果 tick 时没有发现任何交易产生，则为 `空entry`。否则为 `transaction entry`，并将这些交易按顺序放在entry里。

我们看一下 Leader 出块步骤：

1. PoH服务不断生成 PoH 哈希序列。
2. 定期(每6.25ms)插入一个 **tick entry** 来表示时间前进。
3. 在合适的地方插入 **transaction entry** 来打包交易。

此时出块过程为：

```yaml
Entry 1: Tick
Entry 2: Tick
Entry 3: Transaction {tx1, tx2, tx3}
Entry 4: Tick
Entry 5: Transaction {tx4}
Entry 6: Tick
```

Entry 的数据数据定义如下

![image-20250911120150707](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250911120150707.png)

这三者之间的关系大概就是上面介绍的这个样子了，那么大家可能会好奇，这个 Entry 有什么作用呢？答案就是在多个节点之间广播交易时，其实广播的就是这个 Entry，这样一次广播就可以包含多笔交易，比起单笔广播效率要高很多。

这里提醒一下，广播时是通过 **Turbine** 协议进行的，它是以 **shred 分片** 的形式将这个 Entry 广播出去的，参考 [Turbine: Block Propagation on Solana](https://www.helius.dev/blog/turbine-block-propagation-on-solana)。

**总结：**

- 一个solt包含 64 个 entry，每个entry里面有含 `交易tick` 和 `空tick` 。每 6.25 毫秒发生一次entry，导致每个区块有 64 次entry，总区块时间为 400 毫秒。
- 每 6.25 毫秒 生成一个新的哈希值，对应一个 Tick。
- 每 64 个哈希值 组成一个 Slot，即 400 毫秒 生成一个 Slot。



###  Shred

上面介绍 Entry 的时候，讲过交易广播是通过 Turbine 协议，以 shred 的形式将 entry 广播到其它节点。

如何理解呢？一个 Entry 可能包含许多交易，数据量比较大，可能几十KB，如果直接原封不动的在网络上直接传播的话，可能存以下问题

- 包丢失率高
- 传输延时大
- 带宽问题

因此，采用了shred分片技术传播。简单的理解就是将 entry 数据分成多份传播到其它节点，如将一个 entry 切分成10小份再传播。但仅仅这样的话，又无法保证接收节点收到的数据一定是完整的。所以又采用了一种叫“**纠删码**”的技术（熟悉分布式存储的开发人员对这个应该不陌生）。

在介绍这个技术之前，我们必须知道一个前提，在 solana 里其实定义了两类shred：

- Data Shred：包含实际的交易数据（entry 的内容）
- Coding Shred: 冗余，基于 纠删码（Erasure Coding）生成，用于容错

而在 Solana 里传播交易数据时，Leader 节点会对entry 的每一批 Data Shred 额外计算一些 Coding shred（或称为 Recovery Shred）。这样可以实现即使部分 Data Shred 丢失，只要收到足够数量的 Data shred + Coding Shred，就可以恢复完整的数据Entry。这个很类似于 RAID 技术，即时部分数据丢失也不影响数据的完整性。

![](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/turbine-tree.webp)

例如：

- Leader节点将 一个 Entry 被切分成 **10 个 Data Shred**
- Leader 再用 **Reed-Solomon** 算法生成 **4 个 Coding Shred**
- 通过 **Turbine 协议分层广播**，在网络上传播这 **14** 个 shred
- 验证节点收到任意 **10** 个Shred，再就能恢复完整 Entry（在其之前会先校验 poH 哈希链，确认顺序和完整性）

这里给出一个一笔交易完整的示意图

![image-20250915113325649](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250915113325649.png)



> 注意：上图并没有体现出来 Entry 概念

以上就是使用shred 传播交易的基本实现原理，推荐参考

- https://www.helius.dev/blog/solana-executive-overview#turbine。
- https://www.helius.dev/blog/turbine-block-propagation-on-solana

**总结**

- Shred: 指将 Entry 切分成多份数据 Data Shred，并对其使用  **Reed-Solomon** 生成对应的 Coding Shred
- **Shred = Entry 的分片单元**，便于网络高效传播。
- Shred 分  **Data Shred**（entry 分片数据）和 **Coding Shred** （基于纠删码的冗余分片）两类
- 验证节点收到大部分 Shred 数据，就可以保证在丢包环境下也能恢复 Entry。
- Shred + Turbine（分层广播）是  Solana 高吞吐的关键。

对于区块共识参考 https://www.helius.dev/blog/solana-executive-overview#consensus

## 常见问题

### 在solana里为什么说至少要 2 个 Slot 才能达到确认？

（1）Leader Slot 负责交易执行（第 1 个 Slot）
在第一个 Slot（400ms）里，Leader 收集交易，执行，并通过广播 Entry的形式广播交易。

但这个 Slot 只是本地执行完毕，还需要让网络中其他验证者达成共识。

（2）下一个 Slot 的验证者需要投票确认（第 2 个 Slot）
Solana 采用 Tower BFT 共识，需要 至少 2/3 的验证者质押投票，才能确认这个 Slot 的状态。

投票通常会在下一个 Slot 里进行，即第 2 个 Slot 的 Leader 也会引用前一个 Slot，并广播投票信息。

（3）网络传播和共识时间
网络需要时间传播 Slot 数据（Entry），验证者收到数据后会进行投票。

这个过程通常需要 1~2 个 Slot，所以一般800ms 内可以得到乐观确认。

### 交易的三种状态关系

![cf1bca5eeb5d0b2c66dced250e93e21f](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/cf1bca5eeb5d0b2c66dced250e93e21f.png)

当验证者将区块提交到链上时，该区块将处于 `processed` 状态。

一旦所需数量的验证者（[66% 的验证者 ](https://docs.solanalabs.com/consensus/commitments)） 投票支持包含该区块，该区块就会被添加到链中，并且承诺级别将更改为 `confirmed`。

在此块之上再构建 31 个块后（12.8s左右），承诺级别将更改为 `finalized`。这里的 31 来源于 **Tower BFT 的 32 Slot 投票锁定期**（32 = 原始 Slot + 31 个新 Slot）, 用来保证分叉时旧区块不可被回滚。

> 投票锁定期 就是 Validator 在一段时间内被“锁定”在这个 Slot 上，不能支持分叉回滚。

## 参考文章

- https://www.helius.dev/blog/solana-executive-overview
- https://www.helius.dev/blog/how-to-land-transactions-on-solana
- https://www.helius.dev/blog/turbine-block-propagation-on-solana
- https://github.com/cfanbo/solana-reading-list/
- [solana-whitepaper](https://solana.com/solana-whitepaper.pdf)
