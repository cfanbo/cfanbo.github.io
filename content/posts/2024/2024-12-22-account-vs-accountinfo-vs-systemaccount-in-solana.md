---
title: Solana中账户类型 Account、AccountInfo与 SystemAccount 的区别
date: 2024-22-06T17:02:20+08:00
type: post
url: /posts/account-vs-accountinfo-vs-systemaccount-in-solana.
categories:
- 程序开发
- web3
tags:
- solana
- web3
---

在Solana中 Account 的角色很重要，它就像Linux中`一切皆文件`的概念一样，无处不在。了解它也是开发Solana的基础，本节主要介绍我们最经常使用的 `Account` 、`AccountInfo` 和  `SystemAccount` 这三种账户类型的区别与使用场景。

当然除此之外还有一些账户类型也很重要，如 `UncheckedAccount`、`Signer`、`TokenAccount`、`Mint`、`CpiAccount`、`Loader`、`Program`、`AssociatedToken` 等，我们这里就不再一一讲解，有兴趣的话可以参考官方相关文档。

> 由于多数情况下都是使用anchor框架开发Solana合约，因此本文主要是根据 anchor-lang 文档里介绍账户来讲解

# 账户类型

以下我们分别对这三种账户类型做一些简单的介绍。

## AccountInfo

在 Solana 中 `AccountInfo` 是最基础的账户类型。

![AccountInfo](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/accountinfo.svg)

其它几种账户类型都是对它的封装，它的[定义](https://docs.rs/anchor-lang/0.30.1/anchor_lang/prelude/struct.AccountInfo.html)

```rust
#[repr(C)]
pub struct AccountInfo<'a> {
    pub key: &'a Pubkey,
    pub lamports: Rc<RefCell<&'a mut u64>>,
    pub data: Rc<RefCell<&'a mut [u8]>>,
    pub owner: &'a Pubkey,
    pub rent_epoch: u64,
    pub is_signer: bool,
    pub is_writable: bool,
    pub executable: bool,
}
```

**字段解释**

key 公钥地址，当前账户的address

lamports 账户中的 lamports。可由程序修改。

data 此账户中保存的数据。

owner 当前账户的所有者（下面会有一个图解释owner关系），它也是一个公钥地址

rent_epoch 租约纪元，此账户下次需支付租金的时间点（每个账户要想在在网络中存在，必须支付一定的存储空间费用，如果达到租金两倍则可以免除）

is_signer 该交易是否由本账户的公钥签名

is_writable 是否可写, 用 account(mut) 声明

executable 是否为可执行文件。如果为合约可执行程序账户的话，则表示为可执行；否则就是普通的账户。

这里讲一下最经常用到的几个字段。

executable 字段如果值为 true 则表示当前账户是一个合约，因为只有合约才可以执行，否则就是一般的账户；lamports 就是当前账户里的代币金额；对于 data 就是我们平时开发中用到的自定义数据。如 `pub account: Account<'info, NewData>` 写法，是表示 `NewData` 为我们的自定义数据；还有 owner 字段也很重要，后面会讲到。

**这种账户类型只是映射网络上的账户，并不创建新的账户。**

## Account

Account 账户类型是我们使用最广泛的账户类型之一，它的主要功能就是实现自定义数据存储，[定义](https://docs.rs/anchor-lang/0.30.1/src/anchor_lang/accounts/account.rs.html#226-229)

```rust
#[derive(Clone)]
pub struct Account<'info, T: AccountSerialize + AccountDeserialize + Clone> {
    account: T,
    info: &'info AccountInfo<'info>,
}
```

这里一共两个字段，其中一下 info 字段就是我们上面介绍过的 AdminInfo 结构体。而 account 就是我们的自定义数据，它是一个 T 泛型参数，实现了三个trait。

这三个 trait 用来实现在 `info.data` 和 T 之间进行序列化和反序列化，当然这一切操作完全是由anchor框架来完成的。否则的话，只能由开发人员来自行完成，这就显的太过于繁琐了。

它的基本用法如下，这里是一个官方提供的用示例

```rust
#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(init, payer = signer, space = 8 + 8)]
    pub new_account: Account<'info, NewAccount>,
    #[account(mut)]
    pub signer: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[account]
pub struct NewAccount {
    data: u64
}
```

这里 NewAccount 是我们业务中需要用到的处理数据。

对于它的序列化操作(数据存储)，可以在 [exit_with_expected_owner](https://docs.rs/anchor-lang/0.30.1/src/anchor_lang/accounts/account.rs.html#253-267) 函数中看到一些套路，这里不再介绍

```rust
    pub(crate) fn exit_with_expected_owner(
        &self,
        expected_owner: &Pubkey,
        program_id: &Pubkey,
    ) -> Result<()> {
        // Only persist if the owner is the current program and the account is not closed.
        if expected_owner == program_id && !crate::common::is_closed(self.info) {
            let info = self.to_account_info();
            let mut data = info.try_borrow_mut_data()?;
            let dst: &mut [u8] = &mut data;
            let mut writer = BpfWriter::new(dst);
            self.account.try_serialize(&mut writer)?;
        }
        Ok(())
    }
```

一句话总结， **Account 是一种用来实现携带自定义数据的账户类型**。它通过实现 trait 来将数据存储到 AccountInfo.data 字段或从这个字段读取并解析为我们定义的数据结构体（NewAccount）。

**这种类型将在网络创建新的账户，用来存储数据。**

## SystemAccount

对于 SystemAccount 同 Account 类型一样，也是对 AccountInfo 的封装。[定义](https://docs.rs/anchor-lang/0.30.1/src/anchor_lang/accounts/system_account.rs.html#8-16)

```rust
/// Type validating that the account is owned by the system program
///
/// Checks:
///
/// - `SystemAccount.info.owner == SystemProgram`
#[derive(Debug, Clone)]
pub struct SystemAccount<'info> {
    info: &'info AccountInfo<'info>,
}
```

这里的定义要简洁很好，只有一个字段，由此看到相比 Account 账户类型它是没有自定义数据功能的。

那为什么还单独用一个新的struct 对  AccountInfo 呢，直接使用 AccountInfo 不一样么？

答案就在注释内容的描述， 它主要用来验证 `SystemAccount.info.owner == SystemProgram`。注意这时用到了 owner 字段哟，这个很重要，它在用来处理权限判断的，一个账户必须有一个所属者，不同的所属者权限也不一样，这个在 Solana中很重要，下面示例中会发现它们的区别。

如这里有一个账户 `9ox8Dd9CSmBuoGqm94aMfvNiPdHZWn71PDEPQCiYhCec`，它的 Owner 是 `E4G3M284c1e5egPntBSYb39BsDwm14qzR7NEPfat7kcp`。

![image-20250120145324073](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250120145324073.png)

而它是一个 Program 类型，也就是合约程序，因此 `executable` 为 TRUE，这一点我们在上面介绍过。

![image-20250120145633128](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250120145633128.png)

> 说明： 
>
> 这里介绍一下 Program Account(https://solana.com/docs/core/accounts#program-account) 。当一个新程序被部署在Solana上后，将创建三个独立的账户，分别为 **Program Account**、**Program Executable Data Account** 和 **Buffer Account**。
>
> 
>
> ![Program and Executable Data Accounts](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/program-account-expanded.svg)
>
> 因此上图中的 Executable Data 也是一个公钥地址 `2nV7Nv...oMAvCr`，它是一个 Program Executable Data Account，只不过它主要存储的是合约程序代码，因此其 data 字段也比较大。
>
> ![image-20250120151030456](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250120151030456.png)
>
> 对于 Buffer Account 它是一个临时账户，在部署期间或升级期间会自动创建。一旦该过程完成，数据将被转移至Program Executable Data Account，且缓冲账户随即关闭，它所占用的存储空间会被 **释放**。

我们接着再看下 Owner 关系

![image-20250120145751154](/Users/sxf/Library/Application Support/typora-user-images/image-20250120145751154.png)

![image-20250120145818318](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250120145818318.png)

可以看到直到 NativeLoader 账户没有 Owner  也，也就是说它是最上面的账户了。

上面的 Owner 关系大概为（图左侧部分为 owner 关系）

![image-20250120152220439](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250120152220439.png)



这种账户映射网络同名账户，不创建新账户。

下面为了更好一理解这三种账户，我们通过一个示例来看一下它的执行结果

# 示例讲解

为了详细了解这三种账户的区别，这里给出一个示例源码 https://beta.solpg.io/678e3346cffcf4b13384d566

```rust
use anchor_lang::prelude::*;

declare_id!("EzSuSQLBQBTZhQnoH2WFZXXBJwvgjXxSb1dehS59QvBL");

#[program]
mod hello_anchor {
    use super::*;
    pub fn initialize(ctx: Context<Initialize>, value: u64, new_value: u64) -> Result<()> {
        msg!("data = {}", value);
        msg!("new_data = {}", new_value);
        ctx.accounts.account.val = value;
        ctx.accounts.account.new_value = new_value;
        msg!("AccountInfo = {:?}!", &ctx.accounts.account_info);
        msg!("Account = {:?}!", &ctx.accounts.account);
        msg!("SystemAccount = {:?}!", &ctx.accounts.system_account);

        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(mut)]
    pub signer: Signer<'info>,

    #[account(mut)]
    pub account_info: AccountInfo<'info>,

    #[account(init_if_needed, payer=signer, space=8 + 16)]
    pub account: Account<'info, NewAccount>,

    #[account(mut)]
    pub system_account: SystemAccount<'info>,

    pub system_program: Program<'info, System>,
}

#[account]
#[derive(Debug)]
pub struct NewAccount {
    val: u64,
    new_value: u64,
}

```

我们通过 beta.solpg.io 网站左侧的 Test 菜单里填写内容如下：

![image-20250120192853383](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250120192853383.png)

这里一共有五个账户，我们只关心中间的三个账户， 执行输出结果为

```shell
Testing 'initialize'...
RPC URL: http://localhost:8899
Default Signer: Playground Wallet
Commitment: confirmed

Transaction executed in slot 113:
  Block Time: 2025-01-20T20:13:41+08:00
  Version: legacy
  Recent Blockhash: GMDajjskkQFgGSSDMEDoh816RinBQnFQWGYnkjafRwWf
  Signature 0: 3kx9ZVTHU67vN6rWrQxbTnQisyJ6FdykQP9NhaGr7riEoBgs8PE9gFXgHdWftwWYCShwQFqbriTfE5uecjzf19ue
  Signature 1: 2ynWU4Ad3ZBfc1DMHzxmnjokKc8wCP4aRLPFVa4SVQXM53cVddQCoc7m6QxnHqpTCa4imFGSY5rrpjFFfbJYFKKt
  Account 0: srw- FwCBU1BEMonq16a4xjqfDmX4mUQgvFLwM5uzY4hJHMcJ (fee payer)
  Account 1: srw- 9ox8Dd9CSmBuoGqm94aMfvNiPdHZWn71PDEPQCiYhCec
  Account 2: -rw- 6xgZiXGnUYF9Bf6bVFHGGp6BY1wVG8Z4zuYXB9GaqJFE
  Account 3: -rw- 7nESiY3eFn55DmYHVxmCSXhpNcVBUWFuAaiQFzagc5UC
  Account 4: -r-- 11111111111111111111111111111111
  Account 5: -r-x EzSuSQLBQBTZhQnoH2WFZXXBJwvgjXxSb1dehS59QvBL
  Instruction 0
    Program:   EzSuSQLBQBTZhQnoH2WFZXXBJwvgjXxSb1dehS59QvBL (5)
    Account 0: FwCBU1BEMonq16a4xjqfDmX4mUQgvFLwM5uzY4hJHMcJ (0)
    Account 1: 7nESiY3eFn55DmYHVxmCSXhpNcVBUWFuAaiQFzagc5UC (3)
    Account 2: 9ox8Dd9CSmBuoGqm94aMfvNiPdHZWn71PDEPQCiYhCec (1)
    Account 3: 6xgZiXGnUYF9Bf6bVFHGGp6BY1wVG8Z4zuYXB9GaqJFE (2)
    Account 4: 11111111111111111111111111111111 (4)
    Data: [175, 175, 109, 31, 13, 152, 155, 237, 6, 0, 0, 0, 0, 0, 0, 0, 7, 0, 0, 0, 0, 0, 0, 0]
  Status: Ok
    Fee: ◎0.00001
    Account 0 balance: ◎96.93952132 -> ◎96.9384534
    Account 1 balance: ◎0 -> ◎0.00105792
    Account 2 balance: ◎0
    Account 3 balance: ◎0
    Account 4 balance: ◎0.000000001
    Account 5 balance: ◎0.00139896
  Log Messages:
    Program EzSuSQLBQBTZhQnoH2WFZXXBJwvgjXxSb1dehS59QvBL invoke [1]
    Program log: Instruction: Initialize
    Program 11111111111111111111111111111111 invoke [2]
    Program 11111111111111111111111111111111 success
    Program log: data = 6
    Program log: new_data = 7
    Program log: AccountInfo = AccountInfo { key: 7nESiY3eFn55DmYHVxmCSXhpNcVBUWFuAaiQFzagc5UC, owner: 11111111111111111111111111111111, is_signer: false, is_writable: true, executable: false, rent_epoch: 18446744073709551615, lamports: 0, data.len: 0, .. }!
    Program log: Account = Account { account: NewAccount { val: 6, new_value: 7 }, info: AccountInfo { key: 9ox8Dd9CSmBuoGqm94aMfvNiPdHZWn71PDEPQCiYhCec, owner: EzSuSQLBQBTZhQnoH2WFZXXBJwvgjXxSb1dehS59QvBL, is_signer: true, is_writable: true, executable: false, rent_epoch: 18446744073709551615, lamports: 1057920, data.len: 24, data: 000000000000000000000000000000000000000000000000, .. } }!
    Program log: SystemAccount = SystemAccount { info: AccountInfo { key: 6xgZiXGnUYF9Bf6bVFHGGp6BY1wVG8Z4zuYXB9GaqJFE, owner: 11111111111111111111111111111111, is_signer: false, is_writable: true, executable: false, rent_epoch: 18446744073709551615, lamports: 0, data.len: 0, .. } }!
    Program EzSuSQLBQBTZhQnoH2WFZXXBJwvgjXxSb1dehS59QvBL consumed 80309 of 200000 compute units
    Program EzSuSQLBQBTZhQnoH2WFZXXBJwvgjXxSb1dehS59QvBL success
 
Confirmed
```

如果只通过查看这些输出日志理解交易的话，可能有点困难。所以我们这里对比着在浏览器 solscan.io 查看到的交易详情理解的话，可能会比较容易。

## Solt Number

```
Transaction executed in slot 113:
```

表示当前交易所在的 slot。

对应交易详情

![image-20250120202752374](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250120202752374.png)

## 区块信息

```
  Block Time: 2025-01-20T20:13:41+08:00
  Version: legacy
  Recent Blockhash: GMDajjskkQFgGSSDMEDoh816RinBQnFQWGYnkjafRwWf
```

这里的 Recent Blockhash 是指最近的区块hash，由于当前交易刚刚写入新的区块，而离它最近的区块肯定是前一个区块，所以这里的 Blockhash 是指当前区块之前的那个区块的 hash 值。

> 注意 Solt 与 Block 两者的关系

## 签名

```
  Signature 0: 3kx9ZVTHU67vN6rWrQxbTnQisyJ6FdykQP9NhaGr7riEoBgs8PE9gFXgHdWftwWYCShwQFqbriTfE5uecjzf19ue
  Signature 1: 2ynWU4Ad3ZBfc1DMHzxmnjokKc8wCP4aRLPFVa4SVQXM53cVddQCoc7m6QxnHqpTCa4imFGSY5rrpjFFfbJYFKKt
```

这里是指两个账户(signer)的签名，这里指 signer 和 account 两个字段账户，signer是用来支付手续费的，而account 的创建又同样需要签名，这个可以 Test  菜单面看到，显示绿色  `signer` 的字段。

## 账户列表

```shell
 Account 0: srw- FwCBU1BEMonq16a4xjqfDmX4mUQgvFLwM5uzY4hJHMcJ (fee payer)  // 支付手续费账户
  Account 1: srw- 9ox8Dd9CSmBuoGqm94aMfvNiPdHZWn71PDEPQCiYhCec  // account_info 字段声明的账户
  Account 2: -rw- 6xgZiXGnUYF9Bf6bVFHGGp6BY1wVG8Z4zuYXB9GaqJFE  // account 字段声明的账户
  Account 3: -rw- 7nESiY3eFn55DmYHVxmCSXhpNcVBUWFuAaiQFzagc5UC  // system_program 字段声明的账户
  Account 4: -r-- 11111111111111111111111111111111              // System Program
  Account 5: -r-x EzSuSQLBQBTZhQnoH2WFZXXBJwvgjXxSb1dehS59QvBL  // 合约ID
```

这里一共六个账户：

 Account 0 账户是指 signer 字段，它是用来支付手续费的，因此后面有（fee payer），

接着三个账户（Account 1、Account 2、 Account 3）分别是 account_info、account 和 system_program 字段声明的账户。

Account 4 账户对应 system_program 字段声明的账户，它是指 System Program, 创建账户的操作是由它来完成的。

最后一个账户 Account 5 是指当前合约ID，在程序源文件最上面通过 `declare_id!`宏声明的。

> 这里的账户显示格式类似于 Linux 中文件权限的模式，其中 srw- 里的 s 表示签名的意思，其它几个与 Linux 下表示一致。

## 指令

接着是指令相关的信息

```
  Instruction 0
    Program:   EzSuSQLBQBTZhQnoH2WFZXXBJwvgjXxSb1dehS59QvBL (5)
    Account 0: FwCBU1BEMonq16a4xjqfDmX4mUQgvFLwM5uzY4hJHMcJ (0)
    Account 1: 7nESiY3eFn55DmYHVxmCSXhpNcVBUWFuAaiQFzagc5UC (3)
    Account 2: 9ox8Dd9CSmBuoGqm94aMfvNiPdHZWn71PDEPQCiYhCec (1)
    Account 3: 6xgZiXGnUYF9Bf6bVFHGGp6BY1wVG8Z4zuYXB9GaqJFE (2)
    Account 4: 11111111111111111111111111111111 (4)
    Data: [175, 175, 109, 31, 13, 152, 155, 237, 6, 0, 0, 0, 0, 0, 0, 0, 7, 0, 0, 0, 0, 0, 0, 0]
```

Program 是指当前合约ID, 接着是指令的一些账户信息以及传递的参数数据，其中 Data 的前八个字节对应的是 ctx参数，后面的两个字节对应的分别是value 和 new_value 参数。

对应浏览器输出

![image-20250120202302729](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250120202302729.png)

> 注意：这里账户的序号与上面的不一样，这里指的是“指令里的账户序号“

## 状态

```
  Status: Ok
    Fee: ◎0.00001
    Account 0 balance: ◎96.93952132 -> ◎96.9384534
    Account 1 balance: ◎0 -> ◎0.00105792
    Account 2 balance: ◎0
    Account 3 balance: ◎0
    Account 4 balance: ◎0.000000001
    Account 5 balance: ◎0.00139896
```

这些是每个账户对应的余额信息：

Account 0 账户是支付费用的账户，可以看到余额发生减少；

Account 1 账户 account 账户，它是一个新创建的账户。账户刚创建时初始化余额为0，后续又给这个账户转入了一些sol，因此看到余额发生变化。

对应浏览器日志详情

![image-20250120203415674](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250120203415674.png)



Account 2 和 Account 3 账户分别是 system_account 和 account_info 账户，这两个账户直接映射到网络。

Account 4 账户 system_program ，它是一个System Program 类型账户，同时显示余额。

Account 5 账户是 合约 ProgramID，同时显示余额。

## 日志

```
  Log Messages:
    Program EzSuSQLBQBTZhQnoH2WFZXXBJwvgjXxSb1dehS59QvBL invoke [1]
    Program log: Instruction: Initialize
    Program 11111111111111111111111111111111 invoke [2]
    Program 11111111111111111111111111111111 success
    Program log: data = 6
    Program log: new_data = 7
    Program log: AccountInfo = AccountInfo { key: 7nESiY3eFn55DmYHVxmCSXhpNcVBUWFuAaiQFzagc5UC, owner: 11111111111111111111111111111111, is_signer: false, is_writable: true, executable: false, rent_epoch: 18446744073709551615, lamports: 0, data.len: 0, .. }!
    Program log: Account = Account { account: NewAccount { val: 6, new_value: 7 }, info: AccountInfo { key: 9ox8Dd9CSmBuoGqm94aMfvNiPdHZWn71PDEPQCiYhCec, owner: EzSuSQLBQBTZhQnoH2WFZXXBJwvgjXxSb1dehS59QvBL, is_signer: true, is_writable: true, executable: false, rent_epoch: 18446744073709551615, lamports: 1057920, data.len: 24, data: 000000000000000000000000000000000000000000000000, .. } }!
    Program log: SystemAccount = SystemAccount { info: AccountInfo { key: 6xgZiXGnUYF9Bf6bVFHGGp6BY1wVG8Z4zuYXB9GaqJFE, owner: 11111111111111111111111111111111, is_signer: false, is_writable: true, executable: false, rent_epoch: 18446744073709551615, lamports: 0, data.len: 0, .. } }!
    Program EzSuSQLBQBTZhQnoH2WFZXXBJwvgjXxSb1dehS59QvBL consumed 80309 of 200000 compute units
    Program EzSuSQLBQBTZhQnoH2WFZXXBJwvgjXxSb1dehS59QvBL success
    
Confirmed    
```

这里以 Program Log 开头的表示系统日志或用户自定义日志输出。

浏览器日志输出详情

![image-20250120201955679](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250120201955679.png)

对于 `invoke [N]` 这种日志表示当前指令的层级，其中 invoke [1] 表示第一层指令，而 invoke [2] 则表示内嵌的第二次指令。

![image-20250120201850691](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250120201850691.png)

最后两行日志是执行合约消耗的计算单元和最终执行结果success。

## 其它

请注意，只有[System Program](https://solana.com/docs/core/accounts#system-program)能够创建新账户，这也是我们在定义 Accounts 时，如果有需要创建新账户的话，必须包含 System Program 的原因（这里指 system_program 字段）。

下图是一个总署合约时，创建的合约账户信息，`HWKNoRTHV34VJfsc5cnXqNGUeVseFLC61SBNpNZrAfna` 就是我们当前要部署的合约ID。

![image-20250120204212593](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250120204212593.png)

System Program 一旦创建了账户，便可以将该新账户的所有权转移给另一个程序（Assigned Program Id），同时给这个账户转了一些SOL。

还记得我个上面提到的创建一个合约，会自动创建哪些账户吗？在上图都可以看到这些账户的。

换言之，为自定义程序创建数据账户需要两个步骤：

1. 调用系统程序创建账户，随后将所有权转移至自定义程序
2. 调用拥有该账户的自定义程序，然后按照程序代码中的定义初始化账户数据

> 为了理解方便，一般将此数据账户创建过程通常被抽象为一个单一的步骤。

# 总结

可能看到对于账户只有 Account<'info, T> 类型的账户调用了创建账户 Create Account 指令（需要在系统内存储数据）。其它两个账户并没有创建， 在合约内 AccountInfo 是指向已经存在账户的引用，不需要初始化，只要在使用的时候，将账户传递给合约即可。对于 SystemAccount 同样也不会创建，它是一个系统账户，在执行一些系统级操作时，需要使用这种账户类型。

在 Anchor 里只有 Account 可以实现自定义数据的存储，如果要在其它账户上实现此功能的知，只能自己来手动扩展，实现序列化与反序列化了。

对于 AccountInfo、Account 和 SystemAccount 三种账户类型的区别大概如下

| **特性**           | **AccountInfo**      | **Account<T>**         | **SystemAccount** |
| ------------------ | -------------------- | ---------------------- | ----------------- |
| **抽象级别**       | 低级别（手动操作）   | 高级封装               | 高级封装          |
| **支持自定义数据** | 需要手动管理         | 是（通过泛型绑定）     | 否                |
| **支持转账**       | 是（手动实现）       | 否（不直接支持）       | 是                |
| **安全性**         | 程序员负责           | Anchor 自动检查        | Anchor 自动检查   |
| **用途**           | 灵活处理所有账户类型 | 管理带自定义数据的账户 | 仅存储和转账 SOL  |

本篇主要讲了账户在执行指令时，它们的区别，后面会再写一篇通过转账合约示例的文章来看一下三个账户的在转账时用法区别。

# 其它账户描述

| 账户类型           | 描述                            | 常见用途                     |
| ------------------ | ------------------------------- | ---------------------------- |
| `Signer`           | 必须签名的账户                  | 验证授权操作                 |
| `UncheckedAccount` | 未验证类型账户                  | 灵活处理账户                 |
| `TokenAccount`     | SPL Token 账户                  | 管理代币余额                 |
| `Mint`             | SPL Token 铸币账户              | 管理代币供应                 |
| `CpiAccount`       | 跨程序调用时使用的账户          | 调用其他程序                 |
| `Loader`           | 加载程序账户                    | 执行动态调用                 |
| `Program`          | Solana 程序账户                 | 调用系统或自定义程序         |
| `AssociatedToken`  | 与钱包地址关联的 SPL Token 账户 | 管理用户地址与代币账户的映射 |
| `State`            | 全局状态账户                    | 存储全局配置或共享数据       |
| 自定义账户类型     | 通过 Anchor 定义的账户结构      | 存储程序特定状态             |

根据具体需求，选择适合的账户类型，以便实现功能和安全性之间的平衡。

# 参考资料

- https://solana.com/docs/core/accounts
- https://docs.rs/anchor-lang/0.30.1/anchor_lang/?search=Account
