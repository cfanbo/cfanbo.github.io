---
title: 什么是Layer 2网络
date: 2024-12-06T14:02:20+08:00
type: post
url: /posts/what-is-Layer 2-network-in-blockchain
categories:
- 程序开发
- web3
tags:
- solana
- web3
---



我们平时提到的比特币、以太坊、Solana，它们都属于Layer 1网络，而Layer 2(L2) 网络是指基于Layer1网络之上构建的一层网络，它很类似于Layer 1 网络，也是一个独立的区块链。但它的主要目的并不是为了代替 Layer 2 ，而是为了通过扩展 Layer1 层网络从而解决一些在 Layer 1 网络中存在的一些问题，它同时继承了 Layer 1 网络的安全性和去中心化性。

# 以太坊存在的问题

这里以以太坊为例，在 L1 网络上随着交易量越来越大，交易频率也越来越频繁，导致Gas（网络交易费）越来越高，一笔交易可能在网络繁忙的时候高达十几美元，导致一些交易可能需要花费好久才可以真正成交到区块网络。

这两个问题大大提高了用户使用门槛，那有没有好的解决办法呢？

# 解决方案

区块链的三个核心特性是 `去中心化`、`安全性`和`可扩展性`，一般简单的区块链架构只能实现其中两个特性（这一点很类似于分布式中的CAP理论）。想要一个安全且去中心化的区块链的话，只能牺牲可扩展性，而这也正是第二层网络发挥作用的地方。

**以太坊生态系统坚定认为，第 2 层扩展是解决可扩展性三难问题的唯一途径，同时保持去中心化和安全性。**

# Layer 2网络工作原理

Layer 2 也是一个独立的区块链，它扩展了以太坊。这在很大程度上减轻了第一层的交易负担，使其变得不那么拥堵，从而使整体更具扩展性。

目前存在多种第二层解决方案，每种都有其自身的权衡和安全模型。如 **Rollups**、**State Channels**、**[Sidechains](https://ethereum.org/zh/developers/docs/scaling/sidechains/)**、**[Validium](https://ethereum.org/zh/developers/docs/scaling/validium/)**、**混合解决方案** 等。

其中**Rollups** 是一种目前比较流行的 **Layer 2（L2）扩展技术方案**，它在以太坊生态系统中得到了广泛应用，并被许多主流项目采用。它不仅提高了区块链(以太坊生态)的性能和可扩展性，同时保持 Layer 1（以太坊）的安全性和去中心化特性。

Rollups 的核心思想是将大量交易打包成一个**批次（Rollup）**在**链下**处理，仅在 Layer 1 上存储**交易数据**和**验证证明**，从而降低交易手续费。

> 在区块链的中，**链下** 是指不在 Layer 1（主链）上直接处理交易或计算，而是将这些任务转移到 Layer 2 网络或其他链下环境中执行

![img](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/layer2-network.webp)

针对 Rollups 分两种类型，`乐观汇总 `和 `零知识证明`， ——它们的主要区别在于如何将此交易数据提交到 Layer 1网络。

## [Optimistic rollups](https://ethereum.org/developers/docs/scaling/optimistic-rollups/)

它假如所有交易都是有效的，但如果怀疑存在无效交易，则提出挑战(质疑)，此时将运行错误证明以查看是否发生这种情况。

- **原理**：

  - 用户将交易发送到 Optimistic rollups 网络，并将这些交易合并成一个批次，并计算新的状态。
  - 将所有交易数据进行压缩，并和新的状态根提交给 layer1 ，并在layer1进行存储
  - 假设此时所有交易都是有效的，而如果有人质疑批次中的交易，则可以提出挑战(在挑战期内)。
  - 如果发现无效交易，挑战者可以提交欺诈证明（Fraud Proof），触发状态回滚。

- **优点**：

  - 实现简单，兼容性高。

- **缺点**：

  - 提现需要等待挑战期（通常为 7 天）。

- **使用场景**
  - **DeFi**：Optimistic Rollups 支持复杂的智能合约，适合 DeFi 应用。
  - **NFT**：提供低成本、高吞吐量的交易环境，适合 NFT 市场。
  
- **示例**：

  - **Optimism**：https://optimism.io/ 由 Optimism 团队开发，专注于以太坊 Layer 2 扩展。 被多个主流 DeFi 项目采用，如 Uniswap 和 Synthetix
  - **Arbitrum**：https://arbitrum.io/  由 Offchain Labs 开发，支持以太坊兼容的智能合约。 广泛应用于 DeFi 和 NFT 项目。

## [Zero knowledge rollups](https://ethereum.org/developers/docs/scaling/zk-rollups/)

简称 **ZK-Rollups（零知识证明 Rollups）**。

零知识汇总使用有效性证明，其中交易计算在链下进行，然后将这些 **数据** 以及其 **有效性验证证明** 一起提供给以太坊主网进行存储。

- **原理**：
  - 使用零知识证明（ZKP）验证交易的有效性。
  - 每个批次(Rollup 节点将大量交易打包成一个批次)包含一个有效性证明（Validity Proof），确保交易的正确性。
  
- **优点**：
  - 无需挑战期，提现速度快。
  - 更高的安全性和隐私性。
  
- **缺点**：
  - 实现复杂，计算成本高。
  
- **使用场景**
  - **支付**：ZK-Rollups 提供快速、低成本的支付解决方案。
  - **隐私保护**：零知识证明提供更高的隐私性，适合隐私敏感的应用
  
- **示例**：
  - **zkSync**：https://zksync.io/ 由 Matter Labs 开发，支持以太坊兼容的智能合约。
  - **StarkWare**：https://starkware.co/ 专注于高性能的 ZK-Rollups。

## 总结

上面提到的两种类型各有优缺点，不同场景需要选择不同的方法，目前使用 Optimistic Rollups 比较多。

Rollups 一句话描述就是通过 **链下计算** 和 **链上存储** 提高区块链的性能和可扩展性。

# 其它

**既然 Layer 2 网络有这些优化，那官方为何不自行实现一个官方 Layer 2网络呢？** 

以太坊生态系统坚定认为，第 2 层扩展是解决可扩展性三难问题的唯一途径，同时保持去中心化和安全性，通过各种方法设计二层网络，都会对以太坊是有益处的。而其它类似于 layer1 链为了实现更高的每秒交易量和更低的费用，不得不在安全性或去中心化方面做出牺牲。

另外 **侧链** 和 **有效性证明链** 是允许从一个区块链桥接资产并在另一个区块链上使用的区块链。侧链和有效性证明链与主链并行运行，并通过桥接与主链交互，但它们的安全性或数据可用性并不依赖于主链。它们与第二层扩展方案类似，但信任假设不同。它们提供更低的交易费用和更高的交易吞吐量。更多关于[侧链](https://ethereum.org/zh/developers/docs/scaling/sidechains/)和[有效性证明链](https://ethereum.org/zh/developers/docs/scaling/validium/)的信息。

# 参考资料

- https://ethereum.org/zh/layer-2/
- https://ethereum.org/zh/layer-2/learn/
- https://ethereum.org/zh/developers/docs/scaling/optimistic-rollups/
- https://ethereum.org/zh/developers/docs/scaling/zk-rollups/
