---
title: SPL-Token CLI 使用入门教程
date: 2025-03-27T10:42:20+08:00
type: post
url: /posts/spl-token-cli-tutorials-in-solana
toc: true
categories:
- 程序开发
- web3
tags:
- solana
- web3
---

本篇文章将对官方教程的基本上进行整理和完善，同时对一些官方未提到的一些知识点和实践进行一些介绍，方便一些刚入门solana开发的同学能够对solana开发有更完整的了解。

本篇文章使用 `spl-token` 命令行进行演示操作，分别介绍 Mint Account 和 token Account 的创建、转账、关闭账户并回收租金SOL等操作。

这里假如用户本机已经安装好solana命令行，并通过 `solana-test-validator` 命令在本地启动开发集群。



# 环境准备

1. 打开一个终端，在本地创建一个新集群

```shell
(base) ➜  ~ solana-test-validator --ledger test-ledger -r
Ledger location: test-ledger
Log: test-ledger/validator.log
⠠ Initializing...                                                                                                                                                                                                     Waiting for fees to stabilize 1...
Identity: 7U4BYYhgGtjXyujPxX7BrqwYJmLA5BGijUhEv6BACphi
Genesis Hash: ATwFvEZBVuQjAYbp2Y8FyrbLL8AQxN3sPxci2uKkNxFe
Version: 2.2.3
Shred Version: 45925
Gossip Address: 127.0.0.1:1024
TPU Address: 127.0.0.1:1027
JSON RPC URL: http://127.0.0.1:8899
WebSocket PubSub URL: ws://127.0.0.1:8900
⠚ 00:00:10 | Processed Slot: 22 | Confirmed Slot: 22 | Finalized Slot: 0 | Full Snapshot Slot: - | Incremental Snapshot Slot: - | Transactions: 21 | ◎499.999895000
```

这进成功在本地启动了一个solana集群环境。

**参数介绍：**

`--ledger` 表示账本数据存储目录

 `-r` 参数表示删除上次生成的账户数据，这样每次执行这个命令时都将创建一个全新的集群环境

2. 修改本地配置，使用本地集群

   ```shell
   (base) ➜  ~ solana config set -ul
   Config File: /Users/sxf/.config/solana/cli/config.yml
   RPC URL: http://localhost:8899
   WebSocket URL: ws://localhost:8900/ (computed)
   Keypair Path: /Users/sxf/.config/solana/id.json
   Commitment: confirmed
   ```

   后面的 `-ul` 表示使用 localhost 本地集群，除此之外还有 `-ud` 表示 devnet 在线开发集群， `-um` 表示主网 mainnet-beta， `-ut` 表示测试集群 testnet。

3. 确认本地账户

   ```shell
   (base) ➜  ~ solana address
   MuXontwAkV9BoWAx8WuDqbWNNsynoTgCHWUF7M443zi
   ```

   这里我的钱包账户地址为 `MuXontwAkV9BoWAx8WuDqbWNNsynoTgCHWUF7M443zi`，在后面的交易中需要使用这个地址进行交易签名。

# 创建Token(Mint  Account)

在 Solana里创建一种代币，其实就是创建一个 Mint Account 账户，因为在 Solana 有一个 `一切皆账户`的概念，这很像 Linux 里一切皆文件一样。

创建命令

```shell
spl-token create-token --program-id TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb --enable-metadata --enable-close
```

重点：在Solana中分 [Token Program](https://spl.solana.com/token)  和 [Token Extensions program](https://spl.solana.com/token-2022) 两类程序，其中后者为最新的程序，简称 `Token 2022`，它支持一些官方提供的[扩展](https://solana.com/zh/solutions/token-extensions)功能。本文为了方便理解，直接使用了 [Token Program](https://spl.solana.com/token)  。

这里执行结果

```shell
(base) ➜  ~ spl-token create-token
Creating token DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR under program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA

Address:  DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
Decimals:  9

Signature: 3KKBcqy3EKHv1NFBTUqQ7e92pun2ypg33tXYxwPKqc5xDyusyzXTw5JcwEbMa1xTszGBmu1se6pKs4xbw96XFcNk
```

表示通过官方Program (`TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` )创建了一个 token (`DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR`)，这个token 其实就是一个 `Mint Account` 地址，其中

 `Address` 字段表示生成的 `Mint Account` 地址，也就是 Token 地址。

`Decimals` 字段表示当前 Token 默认小数位数，这里表示 `9` 位，也可以手动指定小数位置，如 `--decimals 6`。

`Signature` 表示本次交易签名，表示可以通过访问 https://solscan.io/tx/3KKBcqy3EKHv1NFBTUqQ7e92pun2ypg33tXYxwPKqc5xDyusyzXTw5JcwEbMa1xTszGBmu1se6pKs4xbw96XFcNk?cluster=custom&customUrl=http://127.0.0.1:8899/ 查看本次交易详情。

由于我们使用的是本地环境，因此在区块链浏览器查看时，需要切换为本地环境，一般为 `127.0.0.1:8899`.

![image-20250327110519330](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250327110519330.png)

现在我们已经创建了一种新的代币，它的地址为 `DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR`，后面创建代币账户(ATA)来持有这些代币。

这里先通过 `spl-token supply` 命令查看一下当前 mint Account 存储的代币供应量。由于这是一个刚刚创建的代币，此时供应量为 0 。

```shell
(base) ➜  ~ spl-token supply DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
0
```

查看 Mint Account 信息

```shell
(base) ➜  ~ spl-token display DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR

SPL Token Mint
  Address: DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
  Program: TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA
  Supply: 0
  Decimals: 9
  Mint authority: MuXontwAkV9BoWAx8WuDqbWNNsynoTgCHWUF7M443zi
  Freeze authority: (not set)
```

字段解释：

Address：当前 Mint Account 地址，即该 SPL 代币的唯一标识符

Program：创建账户的程序地址，`TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` 是官方提供的一个Program

Supply：总供应量，账户刚创建，所以为0

Decimals： 小数位数

Mint authority：铸造权限，只有这个地址可以铸造更多的该 Token，这里使用的是本地账户

Freeze authority： 表示哪个账户有冻结当前 Token Account 的权限，这里为空表示Token Account 无法被冻结

> 如果使用的是 Token 2022 程序，则可能输出
>
> ```
> SPL Token Mint
>   Address: HGqmRXedsKMXAB1Yi3CjKwEFoS7QAr7NDiQv1MpoY6HB
>   Program: TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb
>   Supply: 0
>   Decimals: 9
>   Mint authority: MuXontwAkV9BoWAx8WuDqbWNNsynoTgCHWUF7M443zi
>   Freeze authority: (not set)
> Extensions
>   Close authority: MuXontwAkV9BoWAx8WuDqbWNNsynoTgCHWUF7M443zi
>   Metadata Pointer:
>     Authority: MuXontwAkV9BoWAx8WuDqbWNNsynoTgCHWUF7M443zi
>     Metadata address: HGqmRXedsKMXAB1Yi3CjKwEFoS7QAr7NDiQv1MpoY6H
> ```
>
> 这里的 Program 是不一样的，同时显示  Extensions 信息



如果此时，你是通过 `spl-token account-info` 命令查看Mint Account

```shell
(base) ➜  ~ spl-token account-info DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
Error: "Account 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa not found"
```

会遇到这个错误，这是由于每个Mint Account 必须关联一个默认的 Token Account 账户，而当前缺少一个对应的 ATA 账户。后面如果有对应 ATA 账户的话，则可以正常显示。

> 如果要实现设置token的 metadata，则需要使用命令
>
> ```shell
> spl-token create-token --program-id TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb --enable-metadata --enable-close
> ```
> 
>这里 TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb 是官方提供的 Token 2022 扩展程序，而后面的参数 `--enable-metadata` 则表示启用 metadata 扩展，--enable-close 是否启用关闭mint Account权限.
> 
>如果不通过 --program-id 参数指定token2022 的话，则默认使用官方提供的另一个程序 `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` ，它不支持扩展功能，因此现在新功能将无法提到支持。

# 创建 token Account

我们已经创建了一种 Token，现在为此代币创建一个代币账户（Token Account），在 Solana 里一般称为 `关联代币账户`，简称ATA。

命令格式为：

```shell
spl-token create-account [OPTIONS] <TOKEN_ADDRESS>
```

这里执行命令

```shell
(base) ➜  ~ spl-token create-account DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
Creating account 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa

Signature: 2vh7PDPHqVvo1mwrZvfdV7ajtic8Zz1dn35iXaTmpZuwz6WWaHoX8iYWiH6BzNqc9EL5ux7m5VR2UPExo9w5GcWj
```

新创建的ata 账户地址为 `2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa`，它的 Owner 是本地账户 `MuXontwAkV9BoWAx8WuDqbWNNsynoTgCHWUF7M443zi`，如果不清楚它们之间关系的话，可以参考文章 https://blog.haohtml.com/posts/pda-ata-account-in-solana/。

现在我们再看一下 mint account 的信息。

```shell
(base) ➜  ~ spl-token account-info DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR

SPL Token Account
  Address: 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa
  Program: TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA
  Balance: 0
  Decimals: 9
  Mint: DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
  Owner: MuXontwAkV9BoWAx8WuDqbWNNsynoTgCHWUF7M443zi
  State: Initialized
  Delegation: (not set)
  Close authority: (not set)
```

这次可以完整的显示出来所有代币信息了。

字段解释：

Address 当前ATA 地址

Program 创建当前账户的程序，这里的 `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` 是官方提供的程序**TokenProgram**，除了这一个还有一个  **Token 2022 Program(**Token Extensions program**)** 程序，它的address 为 `TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb`。

Balance 当前账户的代币供应量，由于它是一个Mint Account地址，因此等于所有Token Account的总务

Decimals 该 Token 采用 9 位小数（类似于 Solana 的 SOL 也是 9 位小数），实际的可读余额需要除以 `10^9`

Mint 当前账户Mint 地址，就是它自己的地址

Owner 我本地的账户

State 账户状态，目前是 `Initialized`，表示已初始化，可以正常使用。

Delegation: 该字段表示该 Token 账户是否被委托给其他地址进行代币管理

Close authority: 该字段表示谁有权限关闭该账户并取回租金（仅当余额为 0 时）。`(not set)` 表示没有设置关闭权限，即只有 `Owner` 本身可以关闭账户，也就是我本地的账户可以关闭这个账户。



默认情况下， `create-account` 命令会创建一个与您的钱包地址($ solana address)关联的代币账户 [associated token account](https://solana.com/zh/docs/core/tokens#associated-token-account) ，作为代币账户的所有者。

如果想手动指定代币账户的Owner，命令格式为：

```shell
spl-token create-account --owner <OWNER_ADDRESS> <TOKEN_ADDRESS>
```

这里演示一下完整的操作步骤：

1. 利用 solana-keygen new 命令创建一个账户，账户将包含私钥和公钥

   ```shell
   (base) ➜  ~ solana-keygen new --outfile player1.json --force
   Generating a new keypair
   
   For added security, enter a BIP39 passphrase
   
   NOTE! This passphrase improves security of the recovery seed phrase NOT the
   keypair file itself, which is stored as insecure plain text
   
   BIP39 Passphrase (empty for none):
   
   Wrote new keypair to player1.json
   ==============================================================================
   pubkey: GeL7fSEwd2PCp2NYCB1vpvz7VB3eJk8nr2qvoci4xjRL
   ==============================================================================
   Save this seed phrase and your BIP39 passphrase to recover your new keypair:
   able hammer stage rough sample ahead identify sound artefact equip frown issue
   ==============================================================================
   ```

   新创建的账户地址为 `GeL7fSEwd2PCp2NYCB1vpvz7VB3eJk8nr2qvoci4xjRL`, 私钥文件为 `player1.json`

2. 创建一个ata账户，并手动指定Owner

   ```shell
   (base) ➜  ~ spl-token create-account --owner GeL7fSEwd2PCp2NYCB1vpvz7VB3eJk8nr2qvoci4xjRL DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR --fee-payer ./player1.json
   Creating account 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv
   Error: Client(Error { request: Some(SendTransaction), kind: RpcError(RpcResponseError { code: -32002, message: "Transaction simulation failed: Attempt to debit an account but found no record of a prior credit.", data: SendTransactionPreflightFailure(RpcSimulateTransactionResult { err: Some(AccountNotFound), logs: Some([]), accounts: None, units_consumed: Some(0), return_data: None, inner_instructions: None, replacement_blockhash: None }) }) })
   ```

   这里提示账户不存在，这是由于账户刚刚账户，账户并没有代币，无法支付手续费。由于我们使用的本地环境，因此可以空投一些代币给这个账户。

   ```shell
   (base) ➜  ~  solana airdrop 100 GeL7fSEwd2PCp2NYCB1vpvz7VB3eJk8nr2qvoci4xjRL
   Requesting airdrop of 100 SOL
   
   Signature: 3A1xPmmC8EexJt2QEgdbnGjts2TW97QxiCFEcPE65R8bkNhvYi2MF4n8kRF1zU5Kv12L1dtTvF8vuvHsZETU7F56
   
   100 SOL
   ```

   再次执行上面的命令
   ```shell
   (base) ➜  ~ spl-token create-account --owner GeL7fSEwd2PCp2NYCB1vpvz7VB3eJk8nr2qvoci4xjRL DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR --fee-payer ./player1.json
   Creating account 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv
   
   Signature: 4yqRSqcpQfsZiUQQhQ5dpcwa6gVB3R3sB8BRkP1NRFRPsDLgNEEVPqsNvewmuPVAVdvLLmWZccJuBfPutAHAKcgb
   ```

​	现在我们创建了第二个 Token Account账户，地址为 `3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv`



到目前为止，我们一共有三个账户，一个 Mint Account 账户，两个 Token Account  账户，对于 Token Account账户的创建又分别介绍了两种方法。

这里使用了两种方法创建现在用另一个命令 `spl-token display` 查看这些账户信息详细信息。

Mint Account

```
(base) ➜  ~ spl-token display DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR

SPL Token Mint
  Address: DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
  Program: TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA
  Supply: 0
  Decimals: 9
  Mint authority: MuXontwAkV9BoWAx8WuDqbWNNsynoTgCHWUF7M443zi
  Freeze authority: (not set)
```

第一个 Token Account

```
(base) ➜  ~ spl-token display 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa

SPL Token Account
  Address: 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa
  Program: TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA
  Balance: 0
  Decimals: 9
  Mint: DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
  Owner: MuXontwAkV9BoWAx8WuDqbWNNsynoTgCHWUF7M443zi
  State: Initialized
  Delegation: (not set)
  Close authority: (not set)
```

第二个 Token Account

```
(base) ➜  ~ spl-token display 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv

SPL Token Account
  Address: 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv
  Program: TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA
  Balance: 0
  Decimals: 9
  Mint: DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
  Owner: GeL7fSEwd2PCp2NYCB1vpvz7VB3eJk8nr2qvoci4xjRL
  State: Initialized
  Delegation: (not set)
  Close authority: (not set)
```




下面我们这两个Token Account 铸造一些代币，并观察其 Supply 的变化。



# 铸币 Mint Token

经过上面两步，我们有了 `Mint Account` 账户和 `Token Account` 账户，现在我们为每个 Token Account 铸造一些代币。第一个账户铸造 100 个代币，第二个账户铸造 20 个代币，并查看 Mint Account 账户的供应量 Supply 变化。

命令格式：

```shell
spl-token mint [OPTIONS] <TOKEN_ADDRESS> <TOKEN_AMOUNT> [--] [RECIPIENT_TOKEN_ACCOUNT_ADDRESS]
```

这里 `<TOKEN_ADDRESS>` 表示Mint Account Address，而 `[RECIPIENT_TOKEN_ACCOUNT_ADDRESS]` 表示 Token Account Address。

第一个账户，省略接收账户地址 `RECIPIENT_TOKEN_ACCOUNT_ADDRESS`，则会使用本地的默认账户（`/Users/sxf/.config/solana/id.json`）作为目标地址。

```shell
(base) ➜  ~ spl-token mint DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR 100

Minting 100 tokens
  Token: DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
  Recipient: 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa

Signature: 5nDdtJXi7S5FXkEKtS7w37JnrEXRYB4KxKGBLCVVbaczByCi1KMxoXTefYzDmacyd7WSGA9H6S8EfJv69QdMZMjw
```

查看 Mint Account 供应量

```shell
(base) ➜  ~ spl-token display DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR

SPL Token Mint
  Address: DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
  Program: TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA
  Supply: 100000000000
  Decimals: 9
  Mint authority: MuXontwAkV9BoWAx8WuDqbWNNsynoTgCHWUF7M443zi
  Freeze authority: (not set)
```

看到 Supply 值变为 `100 * 10**9`， 这里的 `9` 就是 `Decimals` 的值。



第二个账户，指定接收账户地址 `RECIPIENT_TOKEN_ACCOUNT_ADDRESS`

```shell
(base) ➜  ~ spl-token mint DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR 20 -- 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv

Minting 20 tokens
  Token: DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
  Recipient: 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv

Signature: 27skMNxq1VW5Wasu9cSMsPqJHe2auYyaUM7LJvyLoNjvczDYaMTw3gCk8odqYQhifi6LrPTxGsrSEXyYgqsKMdm5
```

查看供应量，这次我们使用 spl-token supply 代替 spl-token display 命令，注意它们的区别

```shell
(base) ➜  ~ spl-token supply DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
120
```

这时会发现通过 `spl-token supply` 命令查看的结果是只有一个总供应量 `120`，这比 `spl-token display` 命令显示信息的少很多。



# 转账

命令格式：

```shell
spl-token transfer [OPTIONS] <TOKEN_ADDRESS> <TOKEN_AMOUNT> <RECIPIENT_ADDRESS
or RECIPIENT_TOKEN_ACCOUNT_ADDRESS> --owner <SENDER_KEYPAIR_PATH>
```



当前两个账户余额为

```shell
(base) ➜  ~ spl-token balance --address 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa
100

(base) ➜  ~ spl-token balance --address 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv
20
```

## 默认支付账户转账

第一个账户 -> 10 -> 第二个账户

```shell
(base) ➜  ~ spl-token transfer DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR 10 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv

Transfer 10 tokens
  Sender: 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa
  Recipient: 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv

Signature: xJroywwG1cmc6R86VmxKXjSj8daDLb4rQyi2iczsFdnAguC6VfGFkbVuFhptAWbq4hLpyJ1WJGJxnhKSxPvibGU
```

注意，这里省略了 `--owner` 参数，则默认使用本地配置的文件 `/Users/sxf/.config/solana/id.json` 账户生成的ATA作为支付账户 。

确认两个账户余额

```shell
(base) ➜  ~ spl-token balance --address 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa
90

(base) ➜  ~ spl-token balance --address 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv
30
```

那么有个问题，如果接收地址是它自己的地址又会怎样？交易能否正常执行呢？

```shell
(base) ➜  ~ spl-token transfer DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR 10 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa

Transfer 10 tokens
  Sender: 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa
  Recipient: 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa

Signature: 2xKDhD6WZyR8EeNLSwZgNgMDHUEGZnPmqMYV5k96Dm49V9LAp8BzXchamSLdppWMPd4fKm3gSC9t3z3BCHKg7yzZ
```

可以看到 Sender 和 Recipient 都是同一个地址，即交易正常执行，这时账户余额不变。

```shell
(base) ➜  ~ spl-token balance --address 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa
90
```

## 指定支付账户转账

第二个账户 -> 10 -> 第一个账户

通过 `--owner` 参数指定 sender 账户的密钥文件。

转账命令

```
(base) ➜  ~ spl-token transfer DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR 10 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa --owner ./player1.json

Transfer 10 tokens
  Sender: 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv
  Recipient: 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa

Signature: 4UyouAJEHJtBLSFE6CeP1jfFVuVFdBqyj3PriHjTQ8qYWMDnfDg1oeAvsq2myniV3hoXUB7RqwWwT5a5jNwWAqzh
```

这时发现 sender 是第二个用户了，这里它是通过 `--owner ./player1.json` 来实现的。

现在我们再看一下两个账户的余额

```shell
(base) ➜  ~ spl-token balance --address 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa
100

(base) ➜  ~ spl-token balance --address 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv
20
```



注意：
对于账户之间的转账，并不会影响 `Mint Account` 的总供应量大小。



# 关闭账户

由于在 Solana 生态里，每个账户都需要支付一定的租金，以保证在区块链上只存在有效的用户，减少维护成本。如果有不需要使用的账户的话，则可以通过将其关闭并回收账户租金。

在Solana里要关闭账户有一个前提，就是不允许  Account 账户里存在代币。如果账户为 Token Account 的话，必须将持有的甩有代币 burn 掉,此操作将影响 Mint Account 的供应量。

## 关闭 Token Account

首先，我们先试图关闭一个账户有代币的账户，这里以第一个账户为例。

```shell
(base) ➜  ~ spl-token close --address 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa
Error: "Account 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa still has 100000000000 tokens; empty the account in order to close it."
```

提示账户仍存在 100 个代币，现在我们burn 掉 40 个代币

```shell
(base) ➜  ~ spl-token burn 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa 40
Burn 40 tokens
  Source: 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa

Signature: 2yw5of15Xub2RX55zXbNkW36n79RJkH1c4d8tsSdHvFnzZc18XioxzUMjvY1Lw1AZyrqr36ALYuGMAJMsX1P1Bjx
```

确认代币账户余额

```shell
(base) ➜  ~ spl-token display 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa

SPL Token Account
  Address: 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa
  Program: TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA
  Balance: 60
  Decimals: 9
  Mint: DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
  Owner: MuXontwAkV9BoWAx8WuDqbWNNsynoTgCHWUF7M443zi
  State: Initialized
  Delegation: (not set)
  Close authority: (not set)
```

代币由 100 变成了 60 个，验证了我们上面提到的 burn 代币将影响总供应量。

我们再看一下 Mint Account 的供应量

```shell
(base) ➜  ~ spl-token supply DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
80
```

由于第二个Token Account 仍有20个代币，因此供应量变为80。

现在我们将剩余的 60 个代币burn掉，并关闭账户。

```shell
(base) ➜  ~ spl-token burn 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa 60
Burn 60 tokens
  Source: 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa

Signature: 218VKfnj3PatFZ4BS9uBSGiUc2Ff83jfuy3DZwxNrsJruztSaZvZojmvmH1UUrrfZdfedG9WHhykMpNrzUKKjSdn
```

关闭账户

```
(base) ➜  ~ spl-token close --address 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa

Signature: 3a9VUhkYqaFVq41XhATe3zhLcZzAL7DMnveMwTkpsD8YSiRbE8NfxM885ufBJvnpDyFa64xqEpQXJYRhjVWC2D2Y
```

在浏览器查看详细

![image-20250327134722368](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250327134722368.png)

可以看到关闭账户后，赎回 `0.00203928 SOL` ，将其返还给原来创建账户时支付手续费的账户。

如果后续再次使用这个已关闭账户的话，将提示账户不存在错误，如

```shell
(base) ➜  ~ spl-token display 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa
Error: "Account 2Vwv3ngvh8srRvSvd2VhTiYcHU6dyh9EkMAXMkTaSTaa not found"
```

这里查看一下 Mint Account 供应量将发现由 80 变为了  20。

接着我们要关闭第二个账户，在此之前我们先看一下这个账户的租金情况

![image-20250327140341478](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250327140341478.png)

账户租金金额为 `0.002039 SOL`，这里 burn 掉这个账户的代币。

```shell
(base) ➜  ~ spl-token burn 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv 20
Burn 20 tokens
  Source: 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv
  
Error: Client(Error { request: Some(SendTransaction), kind: RpcError(RpcResponseError { code: -32002, message: "Transaction simulation failed: Error processing Instruction 0: custom program error: 0x4", data: SendTransactionPreflightFailure(RpcSimulateTransactionResult { err: Some(InstructionError(0, Custom(4))), logs: Some(["Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA invoke [1]", "Program log: Instruction: BurnChecked", "Program log: Error: owner does not match", "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA consumed 4394 of 4394 compute units", "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA failed: custom program error: 0x4"]), accounts: None, units_consumed: Some(4394), return_data: None, inner_instructions: None, replacement_blockhash: None }) }) })
```

出现错误“**owner does not match**”，这是因为这个账户是使用 `player1.json` 用户生成的，而非本地默认账户(`/Users/sxf/.config/solana/id.json`)，因此需要通过 `--owner` 指定账户私钥。

```shel
(base) ➜  ~ spl-token burn 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv 20 --owner ./player1.json
Burn 20 tokens
  Source: 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv

Signature: 5FEYNn5Uf2XnZckg5SJkDVJDPtHgMf3BjrD4H6b9wwtwrJimvGo6mRz11ynmqBRxJ8Q4tYf2A7HJEFXtTPrsrdcb
```

浏览器交易详情

![image-20250327140030227](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250327140030227.png)

同样，对于关闭账户也需要指定 owner

```shell
(base) ➜  ~ spl-token close --address 3x73dZu54AGwe6NmUyASSw3DwrDKFmShoxoZjVV79rxv --owner ./player1.json

Signature: 3nJCba7LPCPuY8QgoRZ4NVa55UtSzLiRNbnMyvdZ7HXYoogFfFXCAj6cpsb9L1pLFJGVNK342QjbMyJcJ3wsHNfK
```

交易详情

![image-20250327140255445](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250327140255445.png)

可以看到，账户租金全部返还给原来支付手续费的账户。

## 关闭 Mint Account

已经关闭了两个 `Token Account` 账户，现在我们不需要关闭 `Mint Account` 账户。

```shell
(base) ➜  ~ spl-token close-mint DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR
Error: "Mint DpEUS1j36nekBw7Sm11MabEXTXPHE1zrhv9xwe8itMLR does not support close authority"
```

提示不支持 `close authority` 功能，表示当前 `Mint Account` 账户并不支持关闭账户操作。

如果想实现关闭 Mint  Account 功能，需要在 create-token 时指定使用 **Token 2022 扩展程序**，同时启用 `--enable-close` 参数，如

```shell
spl-token create-token --program-id TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb --enable-close
```



# 总结

不要试图认为学会了命令行工具如何使用，就等于学会了Solana中的 Program 开发，命令行工具只不过是官方为了方便大家更好的理解solana中的一些概念而推出的一个工具而已，要想真正了解内部实现原理，还得动手写实现代码才可以。

通过本文介绍的一些使用方法，你需要明白 Mint Account 和 Token Account 之间的区别与关系。

知识点：

1. 在solana中，一切皆账户（类似Linux中一切皆文件的概念）
2. 创建一种Token代币就是创建一个 Mint Account账户
3. 一个 mint account 可以创建多个 Token Account ，这些 Token Account 是通过PDA生成的，简称为 ATA，它是没有私钥的
4. 每个账户在区块链上要想存在，必须支付一定的租金（存在两年免租的概念）
5. 账户不用的话，需要close掉，以回收SOL租金，同时减少区块维护负担
6. close账户前要保证账户没有代币，如果有代币的话，可以burn掉或者transfer给其它Token Account
7. 官方提供了两个程序，分别为 **Token Program** 和 **Token Extensions Program**，后者简称为 Token 2022，它是官方新推荐的程序，官方为其提供了一系列的扩展，这在使用 Anchor 框架开发时，尤其的方便，强烈推荐使用。
8. 对于ATA 账户它是不可变账户，这里的不可变是指 owner 无法修改，主要出于安全考虑
9. 对于NFT开发方法与上面类似，只不过指定了 Mint Account的 decimlas 为0, 同时supply 为1，并禁止 mint Account铸造token, 同时还会有一个关联的额外元数据账户（[nft](https://solana.com/zh/developers/courses/tokens-and-nfts/nfts-with-metaplex#nfts-on-solana)）



# 参考资料

- https://solana.com/zh/docs/core/tokens#create-token-account
- https://www.anchor-lang.com/docs/tokens/basics
- https://www.anchor-lang.com/docs/tokens/extensions
