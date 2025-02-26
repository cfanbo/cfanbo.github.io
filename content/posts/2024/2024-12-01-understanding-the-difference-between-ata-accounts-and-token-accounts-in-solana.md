---
title: 了解 Solana 中ATA账户与普通账户的关系
date: 2024-12-01T11:22:20+08:00
type: post
url: /posts/understanding-the-difference-between-ata-accounts-and-token-accounts-in-solana
toc: true
categories:
- 程序开发
- web3
tags:
- solana
- web3
---

本文主要通过示例让大家理解在 Solana 中 `ATA` 账户与普通账户的关系。


# 目的

主篇内容目的是为了让开发者加深对 solana 中 Account 这一概念的理解，同时搞清楚 `关联代币账户(ATA) ` 在 Solana 开发中的使用场景和用法，以及在多个账户之间的交易和手续费扣除情况。

本篇实现源码会在 [github.com/cfanbo/solana-repos/](https://github.com/cfanbo/solana-repos/tree/main/tokenaccount-and-ata) 中找到。

这里用到的一些api 函数可以在以下地址找到：

- [@solana/web3.js](https://solana-labs.github.io/solana-web3.js/index.html)  用户实现通过 Solana JSON RPC API 与 Solana 网络上的帐户和程序进行交互。
- [@solana/spl-token](https://solana-labs.github.io/solana-program-library/token/js/index.html) 用于实现与 SPL Token 和 Token-2022 程序交互。


本文通过脚本实现 SPL Token 标准功能，并不需要调用已创建好的智能合约，因此不需要 programId.

# 设置网络环境

```shell
➜  my-solana-program git:(master) ✗ solana config get
Config File: /Users/sxf/.config/solana/cli/config.yml
RPC URL: https://api.devnet.solana.com
WebSocket URL: wss://api.devnet.solana.com/ (computed)
Keypair Path: /Users/sxf/.config/solana/id.json
Commitment: confirmed
```

默认情况下，当根据教程 https://solana.com/zh/docs/intro/installation 安装好后，默认环境就是开发环境。不过这里为了方便，我使用了本地作为开发环境。

```shell
➜  my-solana-program git:(master) ✗ solana config set --url localhost
Config File: /Users/sxf/.config/solana/cli/config.yml
RPC URL: http://localhost:8899
WebSocket URL: ws://localhost:8900/ (computed)
Keypair Path: /Users/sxf/.config/solana/id.json
Commitment: confirmed

➜  my-solana-program git:(master) ✗ solana config get
Config File: /Users/sxf/.config/solana/cli/config.yml
RPC URL: http://localhost:8899
WebSocket URL: ws://localhost:8900/ (computed)
Keypair Path: /Users/sxf/.config/solana/id.json
Commitment: confirmed
```

在本地启动模拟器服务

```shell
➜  solana-test-validator --ledger test-ledger
--faucet-sol argument ignored, ledger already exists
Ledger location: test-ledger
Log: test-ledger/validator.log
⠓ Initializing...                                                                                                                                                                                           Waiting for fees to stabilize 1...
Identity: 9LXK1AD1zGx37VYt5X8GADrJgni2MiF7Gykp3X66vkCP
Genesis Hash: 5yFQTxGSnmFsD1AXSH75wi3JpwDio64msX2fo2FT5xZQ
Version: 2.0.21
Shred Version: 56690
Gossip Address: 127.0.0.1:1024
TPU Address: 127.0.0.1:1027
JSON RPC URL: http://127.0.0.1:8899
WebSocket PubSub URL: ws://127.0.0.1:8900
  RPC connection failure: error sending request for url (http://127.0.0.1:8899/): error trying to connect: tcp connect error: Operation timed out (os error 60)
Identity: 9LXK1AD1zGx37VYt5X8GADrJgni2MiF7Gykp3X66vkCP
Genesis Hash: 5yFQTxGSnmFsD1AXSH75wi3JpwDio64msX2fo2FT5xZQ
Version: 2.0.21
Shred Version: 56690
Gossip Address: 127.0.0.1:1024
TPU Address: 127.0.0.1:1027
JSON RPC URL: http://127.0.0.1:8899
WebSocket PubSub URL: ws://127.0.0.1:8900
```

> 这里启用服务使用了 --ledger  参数，以将数据可以持久化，否则交易信息在浏览器查看时，只能保留几分钟

这里前端脚本以 typescript 为主。

配置使用 localhost 作为 RPC 端点

```type
  // 1 设置localhost:8899 rpc
  // Connect to a solana cluster. Either to your local test validator or to devnet
  //const connection = new Connection("https://api.devnet.solana.com", "confirmed");
  const connection = new Connection("http://localhost:8899", "confirmed");
```

# 创建账号

在 solana 中有两个很重要的概念就是[账户模型](https://solana.com/zh/docs/core/accounts) 和 [PDA](https://solana.com/zh/docs/core/pda)，官方文章对此作为详细的介绍，包括它的生成过程。在系统中为了简化查找特定铸造和所有者的代币账户地址的过程，我们通常使用 [关联代币账户](https://solana.com/zh/docs/core/tokens#%E5%85%B3%E8%81%94%E4%BB%A3%E5%B8%81%E8%B4%A6%E6%88%B7) ，简称 `ATA`, 它与 PDA 有着紧密的关系。

![关联代币账户](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/associated-token-account.svg)
PDA 提供了一种使用一些预定义输入生成地址的确定性方法。这使我们能够在以后轻松 找到账户的地址。

可以看到 ATA 账户的生成，依赖于 Wallet Account 和 Mint Account 这两个地址，下面的代码也可以验证这一点。

下面我们创建三个账户。

这里一个账户是从本地文件 `~/.config/solana/id.json` 中读取中，另两个账户是随机生成的。

 ```types
   // 包括从本地读取一个账户
   const payer = await getKeypairFromFile("~/.config/solana/id.json");
 
   // 随机创建两个账户
   const user1 = Keypair.generate(); // 第一个用户的密钥
   const user2 = Keypair.generate(); // 第二个用户的密钥
 ```



# 创建转账交易

## 创建 Mint 账户

```ts
  // 3. 创建 Mint
  const mint = await createMintAccount(connection, payer);
```

函数实现为

```ts
async function createMintAccount(connection: Connection, payer: Keypair) {
  const mint = await createMint(
    connection,
    payer,
    payer.publicKey, // mint的所有者
    null, // mint没有冻结权限
    9, // 精度，通常是9
  );
  console.log("Mint address:", mint.toBase58(), "\n");
  return mint;
}
```

这里指定了代币小数点为 9 位，这个也是多数选择的精度位数。

## 创建ATA 账户

这里我们分别为每个账户创建对应的ATA账户，并利用 Mint 账户给这些ATA账户充值一些代币

```ts
  // 4. 为每个账号创建对应的 ATA 账号，并添加一些代币
  await mintTokens(connection, mint, payer, 500 * 10 ** 9, payer);
  await mintTokens(connection, mint, user1, 100 * 10 ** 9, payer);
  await mintTokens(connection, mint, user2, 20 * 10 ** 9, payer);
```

在代币里小数点一般默认为 `9` 位，因此这里 `500 * 10 ** 9` 表示转 200个代币

这里 mintTokens 实现

```ts
// 创建ATA账号，并添加一些代币
async function mintTokens(
  connection: Connection,
  mint: PublicKey,
  user: Keypair,
  amount: number,
  payer: Keypair,
) {
  // 1. 获取或创建用户的 ATA
  const userATA = await getOrCreateAssociatedTokenAccount(
    connection,
    payer,
    mint,
    user.publicKey,
  );
  console.log("User ATA address:", userATA.address.toBase58());

  // 2. 添加代币
  await mintTo(connection, payer, mint, userATA.address, payer, amount);
  console.log(`Minted ${amount} tokens to ${userATA.address.toBase58()}`);
}
```

对于创建ATA 账户是通过api函数  [getOrCreateAssociatedTokenAccount()](https://solana-labs.github.io/solana-program-library/token/js/functions/getOrCreateAssociatedTokenAccount.html) 实现的，需要 Mint 账户 和 用户账户(也就是上图中的 Wallet Address)，函数参数解决

![image-20250107130234499](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250107130234499.png)

为了后面测试账户之间的转账功能，通过  [mintTo](https://solana-labs.github.io/solana-program-library/token/js/functions/mintTo.html) 为 ATA 充值代币。函数签名

```
mintTo(connection, payer, mint, destination, authority, amount, multiSigners?, confirmOptions?, programId?): Promise<TransactionSignature>
```

这两个函数的调用都需要支付一定的sol手续费，这里仍是从 payer 账户里扣除。

注意：在 Solana 中手续费是以原生币 sol 扣除的，由于这个账户是我本地已存在的账户，账户里已经通过空投得到了一些sol 币，因此这里可以支付手续费，保证交易的完成。

> 注意原生币与代币的区别，手续费是以原生币 sol 为准的。

这里我们共用到两个api 函数，一个是获取或创建ATA函数 getOrCreateAssociatedTokenAccount()，另一个是 充值代币函数 minTo()。

## 账户转账

由于在这个合约里，我们主要演示对代币的操作（当然也可以实现原生币的操作），所在我们需要在多个ATA账户之间实现转账功能。

在转账之前 ，我们需要获取每个账户对应的 ATA 地址，这个通过调用api函数 [getAssociatedTokenAddress()](https://solana-labs.github.io/solana-program-library/token/js/functions/getAssociatedTokenAddress.html) 实现。函数签名

```ts
getAssociatedTokenAddress(mint, owner, allowOwnerOffCurve?, programId?, associatedTokenProgramId?): Promise<PublicKey>
```

分别获取所有ATA账户地址(其地址在上面已经创建过)

```TS
// 5. 转账，这里在ATA账号之间实现转账功能
  let payerATA = await getAssociatedTokenAddress(mint, payer.publicKey);
  let user1ATA = await getAssociatedTokenAddress(mint, user1.publicKey);
  let user2ATA = await getAssociatedTokenAddress(mint, user2.publicKey);
```

我们先将 payerATA 账户的代币转给 user1ATA 一些, 再将 user1ATA 转代币给 user2ATA 一些。

```ts
  // 5.1 payer => user1ATA
  // 这里 payer 账户里已存在一些原生币，不需要空投
  const tx1 = await transfer(
    connection,
    payer,
    payerATA,
    user1ATA,
    payer,
    100 * 10 ** 9, // 转账 500 个代币
  );
	console.log("tx1", tx1);

	// 5.2 user1ATA => user2ATA
  // 由于 user1 是刚刚创建的，账户没有原生币sol,因此需要空投一些 SOL 用于支付交易手续费
  const airdropSignature2 = await connection.requestAirdrop(
    user1.publicKey,
    1 * 10 ** 9, // 1 SOL 作为转账手续费
  );
  await connection.confirmTransaction(airdropSignature2);
  const tx2 = await transfer(connection, user1, user1ATA, user2ATA, user1, 50 * 10 ** 9);
  console.log("tx2", tx2);
```

这里我们看一下api函数  [transfer](https://solana-labs.github.io/solana-program-library/token/js/functions/transfer.html) 的签名

```ts
transfer(connection, payer, source, destination, owner, amount, multiSigners?, confirmOptions?, programId?): Promise<TransactionSignature>
```

同上面的参数基本一样， payer 表示支付手续费的账户，一般是代币转出人支付这个手续费，当然也可以选择其它账户，如 payer、user2 或 其它账户，只要账户有sol原生币都没有问题的； source  表示转出代币账户; destination 表示代币接收账户； owner 表示source 的 ower账户（第二个transfer里 user1ATA 的 owner 为  user1）; amount 代币数量。

这一步我们用到了三个api函数， 用来获取ATA地址的 getAssociatedTokenAddress() 和 实现转账功能的 transfer() ，还有一个空投函数 [requestAirdrop()](https://solana-labs.github.io/solana-web3.js/classes/Connection.html#requestairdrop)。

在程序开始时，我们为三个账户分别充值了500、100、 20代币，经过两笔交易完成后， 代币账户金额发生变化

```shell
➜  my-solana-program git:(master) ✗ npm start

> my-solana-program@1.0.0 start
> tsx client.ts

(node:55860) [DEP0040] DeprecationWarning: The `punycode` module is deprecated. Please use a userland alternative instead.
(Use `node --trace-deprecation ...` to show where the warning was created)
PAYER: MuXontwAkV9BoWAx8WuDqbWNNsynoTgCHWUF7M443zi
USER1: 57RQvKFL6rozNJZDMM4B7GGNuo1TBrqMgPqFohKSxuVq
UESR2: F2y3HuXHdYz2p88mHF4tJmfWiAxmEhRbwLbsvNYCSYMm
Mint address: Cndihn8MTvGdHhPJvJtrEknQHbZXWTw4EGUenEoyjYmM

User ATA address: GpbxDBXKfdkSCsveExFANyFtPAonBD9xJbGCLp7az1KV
Minted 500000000000 tokens to GpbxDBXKfdkSCsveExFANyFtPAonBD9xJbGCLp7az1KV
User ATA address: 4WGgzpBd1E618VP88w3ihbzHfKuQKSy9pvfJfjNpWci1
Minted 100000000000 tokens to 4WGgzpBd1E618VP88w3ihbzHfKuQKSy9pvfJfjNpWci1
User ATA address: 7Ue8JtXa8kW91P3Zo5pw8VeKTzcDN51SNKcrDCHCpZY
Minted 20000000000 tokens to 7Ue8JtXa8kW91P3Zo5pw8VeKTzcDN51SNKcrDCHCpZY
payer:  GpbxDBXKfdkSCsveExFANyFtPAonBD9xJbGCLp7az1KV

转账前:
payer代币余额: 500
支付者代币余额: 100
接收者代币余额: 20
tx1 d13q4KCfvTjCZrW9qKJVn3JfrAeH4CGR6oPbvpnVzzowxbTUnqnXWv14sEQYn3XjedjkFbdVLejkJC4AeaacRWs
user1 初始余额： { lamports: 1000000000, sol: 1, formatted: '1.000000000 SOL' }
tx2 5QRKXbzZ3udfQF7doGHqJvubNKvosoKeaa5b2rRy4rBbDBoJjgZUUzfKwqC36AXmsKeVcnd8HRu2UefcjM9JRwW7
user1 转账后： { lamports: 999995000, sol: 0.999995, formatted: '0.999995000 SOL' }

转账后:
payer代币余额: 400
支付者代币余额: 150
接收者代币余额: 70
```

可以看到在第二笔交易完成后，我们的 user1 原生代码也减少了，这个被用来支付手续费消耗掉了。

### 账户与ATA账户的对应关系

| 用户  | 普通账户                                     | ATA账户                                      |
| ----- | -------------------------------------------- | -------------------------------------------- |
| payer | MuXontwAkV9BoWAx8WuDqbWNNsynoTgCHWUF7M443zi  | GpbxDBXKfdkSCsveExFANyFtPAonBD9xJbGCLp7az1KV |
| user1 | 57RQvKFL6rozNJZDMM4B7GGNuo1TBrqMgPqFohKSxuVq | 4WGgzpBd1E618VP88w3ihbzHfKuQKSy9pvfJfjNpWci1 |
| user2 | F2y3HuXHdYz2p88mHF4tJmfWiAxmEhRbwLbsvNYCSYMm | 7Ue8JtXa8kW91P3Zo5pw8VeKTzcDN51SNKcrDCHCpZY  |

### ATA 账户详情

第一笔转账 （payerATa => user1ATA, 100）


![image-20250107142446122](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250107142446122.png)

上面的 Token Program 是合约ID，它是 Solana 官方的代币程序地址.

![image-20250107183026034](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250107183026034.png)

> 凡是使用 spl-token program 这种方式开发的程序，这个合约ID就永远不会改变。
>
> 如果你是自己的开发合约，在客户端里调用合约的话，则这个合约ID就是部署的合约ID。··

第二笔转账 （user1ATA => user2ATA, 50),

![image-20250107142619368](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250107142619368.png)

**payerATA**

![image-20250107143640932](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250107143640932.png)

**user1ATA**

![image-20250107143501210](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250107143501210.png)

**user2ATA**

![image-20250107143311163](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250107143311163.png)

以上三个账户都是ATA账户，因此在浏览器查看的时候都显示 `isOnCurve: FALSE`, 如果查看它们的 owner 用户的话，会发现 `isOnCurve: TRUE`

### 普通账户详情

**账户 payer**

```shell
➜  my-solana-program git:(master) ✗ solana balance
499999999.855950356 SOL
```



![image-20250107144152525](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250107144152525.png)

**账户 user1**

```shell
➜  my-solana-program git:(master)✗ solana balance 57RQvKFL6rozNJZDMM4B7GGNuo1TBrqMgPqFohKSxuVq
0.999995 SOL
```



![image-20250107144557450](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250107144557450.png)

**账户 user2**

```shell
➜  my-solana-program git:(master) ✗ solana balance F2y3HuXHdYz2p88mHF4tJmfWiAxmEhRbwLbsvNYCSYMm
0 SOL
```

![image-20250107145307758](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250107145307758.png)

仔细观察的话，会发现这个账户没有 `Owner` 属性，其它两个账户都正常的，这是怎么回事呢？具体原因见：https://blog.haohtml.com/posts/why-token-account-have-no-owner-property-in-solana/ 

# 总结

1. 了解普通账户与ATA账户之间的关系，ATA账户是一个普通账户持有一种代币的账户，如果账户有多种代币的话，则会有多个ATA账户
2. 生成ATA 账户所需要的条件
3. 了解常见的API函数，如获取ATA账户地址和创建ATA账户、创建代币、给用户账户铸造币、获取账户原生币和代币数量
4. 了解如何创建一个交易，并清楚手续费支付由谁来控制，以及扣除的是sol原生币而非代币。

# 参考资源

- https://solana.com/zh/docs/intro/installation 
- https://solana.com/zh/docs/core/accounts
- https://solana-labs.github.io/solana-program-library/token/js/index.html
- https://solana-labs.github.io/solana-web3.js/index.html

