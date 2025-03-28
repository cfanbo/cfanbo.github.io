---
title: solana中MEV监听交易的几种方法
date: 2025-03-28T15:02:20+08:00
type: post
url: /posts/mev-in-solana
toc: true
categories:
- 程序开发
- web3
tags:
- solana
- web3
---



# 策略分类

在 Solana 上 **MEV（最大可提取价值）**的几种策略主要有：

- 套利（Arbitrage）
- 清算（Liquidation）
- 抢跑（Front-Running）
- 三明治攻击（Sandwich Attack）
- 后置插队（Back-Running）
- Jito MEV 竞标

## 套利（Arbitrage）：

价格差异猎手 想象你是一个精明的商人，在不同的市场上发现同一件商品价格不同。比如，在A市场一个苹果卖1元，在B市场卖1.2元。你会怎么做？立即在A市场买入，然后在B市场卖出，赚取0.2元的差价。

在加密世界中，这就是套利。交易员快速在不同交易所或流动性池之间捕捉加密货币价格差异，瞬间完成买卖，赚取微小但稳定的利润。

## 清算（Liquidation）：

充当风险管理者角色。想象银行放贷时，借款人需要提供抵押物品，如果抵押物品价值下跌到无法覆盖贷款，银行会立即收回资产。

在加密借贷平台，当用户的抵押品价值跌破安全线，MEV机器人（**清算人Liquidator）** 通常可以以折扣价买入资产。这是因为在借贷平台（如 **Solend**、**Mango Markets**）中，如果用户的抵押资产价值不足以覆盖其贷款，借贷平台会进行 **清算**，并出售抵押资产来弥补贷款人的损失。MEV机器人（清算人）通过买入这些资产并偿还借款人欠的平台，可以获得折扣或者奖励。

## 抢跑（Front-Running）

想象在股市，有人获得内部消息，得知一家上市公司发生重大利好消息，他要在股市正式开市前抢先买入大量股票（盘前交易），这被称为"抢跑"。

MEV机器人监听交易池，当发现即将发生大额交易时，快速插入自己的交易，在原交易之前完成，从中获利。这就像是站在股市交易大厅的最前排，抢先一步。

## 三明治攻击（Sandwich Attack）

假如你要在一个拥挤的市场买入大量的苹果。一些精明的商贩知道这个消息后，他会在你购买之前大量的收购市场里的苹果，然后再出售给你，从而获利。

由于在一笔大额交易的前后各插入一笔交易，通过制造价格滑点来获利，就像把目标交易"夹"在中间，形成一个"三明治"。

```
     攻击者的前置交易
            ↓
    目标用户的大额交易
            ↓
     攻击者的后置交易
```

在加密交易中，**MEV机器人** 会在大额交易前后插入自己的交易，通过制造价格波动来获利，就像把目标交易"夹"在中间。

## 后置插队（Back-Running）

可以想象在一个拍卖会上，一位艺术家的艺术品被以高价成交后，紧接着他的其它类似艺术品的价值大概率也会上涨。

**MEV机器人** 会紧随大额交易之后立即交易，利用这个交易带来的市场变动获利，就像是紧跟在大浪后的冲浪者。

## Jito MEV 竞标

**区块空间拍卖** 你可以将其想象成一个竞标会，不同的商人为获得 **最佳展位** 而竞争。谁出价最高，谁就能获得最好的位置。

在Solana区块链上，MEV机器人通过Jito平台向验证者竞标区块空间。出价最高的机器人可以将自己的交易打包到区块中，从MEV中获得收益。

## 总结

总的来说，MEV策略就像是金融世界的高频交易算法，需要极快的计算能力、复杂的算法和对市场的敏锐洞察。它们在捕捉微小利润的同时，也为市场提供了一定的流动性和价格发现机制。

不过，这些策略也存在争议。它们可能被视为不道德，因为它们在某种程度上"操纵"了市场。



对于以上这些MEV策略，在开发时，一般通过以下几种监听交易的方式实现

------

# **监听实时交易日志（onLogs）**

**适用场景**：抢跑（Front-Running）、三明治攻击（Sandwich Attack）

📌 **思路**：

- 监听目标 DEX（如 Raydium、Orca、Jupiter）的交易日志
- 解析 **交易内容**（Swap / 清算 / 预言机更新）
- 立即提交 **更高 Gas 费的交易** 抢跑

## 示例代码

```typescript
import { Connection, PublicKey } from "@solana/web3.js";

const connection = new Connection("https://api.mainnet-beta.solana.com", "confirmed");
const RAYDIUM_PROGRAM_ID = new PublicKey("675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8");

connection.onLogs(
  RAYDIUM_PROGRAM_ID,
  async ({ logs, signature }) => {
    console.log(`⚡ 检测到新交易: ${signature}`);

    // 获取交易详情
    const tx = await connection.getTransaction(signature, {
      maxSupportedTransactionVersion: 0,
      commitment: "confirmed",
    });

    if (tx && tx.meta) {
      console.log(`🛠 解析交易详情:`, tx.meta.logMessages);
      
      // 检测是否是套利交易
      if (tx.meta.logMessages.some(log => log.includes("swap"))) {
        console.log(`🚀 发现 DEX 交易，尝试抢跑！`);
        sendHigherFeeTransaction(tx);
      }
    }
  },
  "confirmed"
);
```

✅ **优点**：

- 能在交易执行前**尽早捕获交易**，抢跑更有效
- 适用于 DEX 交易，如 Swap、借贷清算

❌ **缺点**：

- 只能监听**特定智能合约**（比如 Raydium），无法监听所有交易
- 需要**解析交易内容**，才能确定是否值得抢跑

------

# 监听 Slot 更新（onSlotUpdate）

**适用策略**：抢跑、三明治攻击

## onSlotUpdate

触发时机为 Slot **构造中**，要早于 **onSlotChange** 。

🔹 触发时机：当 Leader 正在构造新区块时，就会不断触发。
 🔹 数据更新：提供 **新区块的实时状态**（未确认状态），**可以提前洞察交易趋势**。
 🔹 适用场景：

- **监控区块构造过程**（更快发现交易）。
- **分析即将进入区块的交易**。
- **适合抢跑、三明治攻击**（比 `onSlotChange` 更快）。

📌 **思路**：

- 监听新区块正在**构造**的过程
- 监测即将**进入新区块**的交易
- 发现套利 / 清算机会时，直接提交更高 Gas 费交易

## 示例代码

```typescript
connection.onSlotUpdate((slotUpdate) => {
  console.log(`⚡ Slot ${slotUpdate.slot} 更新:`, slotUpdate);
  
  if (slotUpdate.type === "optimisticConfirmation") {
    console.log(`🚀 Leader 正在构造区块 Slot ${slotUpdate.slot}，可以尝试抢跑！`);
  }
});
```

对于 [slotUpdate.type](https://solana-labs.github.io/solana-web3.js/types/SlotUpdate.html) 有以下几种类型

Slot updates which can be used for tracking the live progress of a cluster.

- `"firstShredReceived"`: connected node received the first shred of a block. Indicates that a new block that is being produced.
- `"completed"`: connected node has received all shreds of a block. Indicates a block was recently produced.
- `"optimisticConfirmation"`: block was optimistically confirmed by the cluster. It is not guaranteed that an optimistic confirmation notification will be sent for every finalized blocks.
- `"root"`: the connected node rooted this block.
- `"createdBank"`: the connected node has started validating this block.
- `"frozen"`: the connected node has validated this block.
- `"dead"`: the connected node failed to validate this block.

✅ **优点**：

- **比 `onSlotChange` 更快**，可以**提前获取区块里的交易**
- 可以用来分析即将进入新区块的**套利 / 清算**交易

❌ **缺点**：

- 监听的 Slot **可能不会最终确认**，数据可能不可靠

------

# 监听新区块（onSlotChange + getBlock）

**适用策略**：套利（Arbitrage）、清算（Liquidation）、后置插队（Back-Running）

## onSlotChange

触发时机为  **Slot确认后**，比晚于 **onSlotUpdate** 。

 🔹 触发时机：当新区块被确认（confirmed）时触发。
 🔹 数据更新：提供 **已确认区块的 slot 号**，但不包含交易数据。
 🔹 适用场景：

- **监控新区块**（获取已确认交易）。
- **获取完整区块数据**（用 `getBlock()`）。
- **适合套利、清算策略**（但比 `onSlotUpdate` 慢）。

📌 思路：

- 监听 **新区块创建**
- 获取新区块的**完整交易列表**
- 计算套利 / 清算机会，提交后续交易

## 示例代码

```typescript
connection.onSlotChange(async ({ slot }) => {
  console.log(`✅ Slot ${slot} 确认成功！`);

  try {
    const block = await connection.getBlock(slot, {
      transactionDetails: "full",
      maxSupportedTransactionVersion: 0,
    });

    if (block && block.transactions.length > 0) {
      console.log(`📦 Slot ${slot} 交易数: ${block.transactions.length}`);

      block.transactions.forEach((tx) => {
        console.log(`📝 交易哈希: ${tx.transaction.signatures[0]}`);
      });
    }
  } catch (error) {
    console.log(`❌ Slot ${slot} 交易获取失败:`, error);
  }
});
```

✅ **优点**：

- 获取 **完整区块的交易**，数据**最准确**
- 适用于 **套利、清算**、**后置插队** 等策略

❌ **缺点**：

- **获取数据较慢**（等区块确认后）
- 不能用于 **抢跑**，因为交易已经打包



## onSlotChange vs onSlotUpdate

| 监听方式       | 触发时机        | 适合策略                 | 延迟                   |
| -------------- | --------------- | ------------------------ | ---------------------- |
| `onSlotChange` | Slot **确认后** | **套利、清算、后置插队** | **较慢**（等区块确认） |
| `onSlotUpdate` | Slot **构造中** | **抢跑、三明治攻击**     | **最快**（实时监听）   |

------

# 订阅 Jito MEV 交易流

**适用场景**：使用 [Jito](https://www.jito.wtf/) 进行 MEV 竞标

📌 **思路**：

- 通过 **Jito Block Engine** 订阅交易流
- 提交**更高优先级**的交易，确保被优先打包

**操作步骤**

1. 访问 [Jito Labs](https://jito.wtf/)
2. 订阅 Jito 的 **MEV 交易流**
3. **提高 Gas 费**，提交交易到 **Jito Block Engine**

✅ **优点**：

- 适用于 **高频 MEV 竞标**
- 可以**直接向 Leader 发送交易**，提高打包概率

❌ **缺点**：

- 需要 Jito 账户权限
- 交易手续费更高

------

# Solana Private RPC（获取未确认交易）

**适用场景**：监听 Leader 交易队列

📌 **思路**：

- 通过 **私有 RPC** 获取 **Leader 正在处理的交易**
- 先行分析套利机会，抢跑更快

✅ **优点**：

- 比 `onSlotUpdate` 更快，适用于 **专业 MEV 交易**

❌ **缺点**：

- 需要 **私有 RPC 访问权限**

------

## **🎯 场景对比**

| 方式                                                         | 适用场景           | 监听速度           | 是否适合抢跑 |
| ------------------------------------------------------------ | ------------------ | ------------------ | ------------ |
| [onLogs](https://solana-labs.github.io/solana-web3.js/classes/Connection.html#onlogs) | **监听 DEX 交易**  | **快**             | ✅            |
| [onSlotUpdate](https://solana-labs.github.io/solana-web3.js/classes/Connection.html#onslotupdate) | **新区块构造中**   | **最快**           | ✅            |
| [onSlotChange](https://solana-labs.github.io/solana-web3.js/classes/Connection.html#onslotchange) + [getBlock](https://solana-labs.github.io/solana-web3.js/classes/Connection.html#getblock) | **套利、清算**     | **较慢（确认后）** | ❌            |
| `Jito MEV`                                                   | **高级 MEV 竞标**  | **极快**           | ✅            |
| **私有 RPC**                                                 | **获取未确认交易** | **最快**           | ✅            |

**🚀 最佳策略**：

1. **抢跑**（Front-Running）：`onLogs()` + `onSlotUpdate()`
2. **套利 / 清算**：`onSlotChange()` + `getBlock()`
3. **Jito MEV 竞标**：Jito Labs API

------

# 总结

- **如果你想抢跑**，必须 **监听交易日志** (`onLogs()`) 或 **区块构造过程** (`onSlotUpdate()`)
- **如果你想套利 / 清算**，可以 **监听新区块** (`onSlotChange()` + `getBlock()`)
- **如果你想确保交易被优先打包**，可以使用 **Jito MEV 竞标**

