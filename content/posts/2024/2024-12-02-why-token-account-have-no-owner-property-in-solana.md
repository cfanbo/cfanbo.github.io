---
title: 在Solana中为什么Token Account没有owner属性
date: 2024-12-02T15:12:40+08:00
type: post
url: /posts/why-token-account-have-no-owner-property-in-solana
categories:
- 程序开发
- web3
tags:
- solana
- web3
---
在上一篇[《了解 Solana 中ATA账户与普通账户的关系》](https://blog.haohtml.com/posts/understanding-the-difference-between-ata-accounts-and-token-accounts-in-solana/)中，我们介绍了在Solana中，ATA账户与Token Account 的区别，其中在浏览器记录查看 user2 用户详细的时候，发现它与前两个账户有所区别，它没有 Owner 属性，这在正常情况下是不应该出现的，所以，我们来分析一下，为什么Token Account没有 `Owner` 属性？

在 Soalna中，默认情况下所有的新账户都属于 [System Program](https://solana.com/docs/core/accounts#system-program)，也只有系统程序拥有的账户才可以作为交易费用支付者（也就是账户里必须的原生币sol）。



![系统账户](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/system-account.svg)

因此示例中正常的 Token Account

![Token Accoun](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250107144557450.png)

异常账号
![](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250107145307758.png)

出现我这种情况主要有两种情况：
1. 交易还在处理中，尚未完成，这种情况下一般只需要等待一会就会正常。
2. 账户未被初始化，需要初始化后才恢复正常。

常见的初始化方法：

1. 空投原生币
2. 转账原生币
3. 通过 SystemProgram.createAccount 创建账号·

在上篇文章里，我们为了演示转账记录，对 payer 和 user1 用户都进行了空投，在空投时将自动对账户进行初始化操作，因此在浏览器里显示的 Owner 是正常状态，但对于 user2 用户，由于没有进行初始化，所以没有 Owner 属性。

那么这里可能引出一些新的疑问。

**为什么没有初始化可以根据这个账户创建ATA账号呢？** 

要Solana中对于账户来说，只需要一个合法的 address 地址即可，它和账户是否初始化无关。

**第二个张疑问是，既然没有初始化，那它怎么会在浏览器里显示呢？**
其实原理和上面的一个差不多，它只是一个地址，理解上所有有效的地址都可以在浏览上访问，唯一的区别是这个地址是否有包含一些数据信息。当你通过浏览器查看时，如果账户有相关数据信息的话，它会将这些数据信息显示出来，如果没有数据信息，则不会显示。
而我们的 `user2` 用户是没有数据信息，所以它根本不会显示 `Owner` 属性。

**验证代码**

这里为了提供演示，给出一下代码：
```typescript
import {
  Connection,
  PublicKey,
  Keypair,
  LAMPORTS_PER_SOL,
  SystemProgram,
} from "@solana/web3.js";

async function main() {
  const connection = new Connection("http://localhost:8899", "confirmed");
  const account = Keypair.generate();

  console.log("账户地址:", account.publicKey.toBase58());

  // 1. 创建后（未初始化）
  console.log("\n创建后（未初始化）:");
  let accountInfo = await connection.getAccountInfo(account.publicKey);
  console.log("账户是否存在:", accountInfo !== null); // false
  console.log("账户信息:", accountInfo); // null

  // 2. 空投 SOL（初始化过程）
  const airdropSig = await connection.requestAirdrop(
    account.publicKey,
    LAMPORTS_PER_SOL,
  );
  await connection.confirmTransaction(airdropSig);

  // 3. 初始化后
  console.log("\n初始化后:");
  accountInfo = await connection.getAccountInfo(account.publicKey);
  console.log("账户是否存在:", accountInfo !== null); // true
  console.log("账户信息:", {
    owner: accountInfo?.owner.equals(SystemProgram.programId)
      ? "System Program"
      : accountInfo?.owner.toBase58(),
    lamports: accountInfo?.lamports,
  });
}
main().catch(console.error);
```
程序输出结果：
```shell
账户地址: HYpaTT1EsmfyLBWjNcP6AhEW4uaiS8rKXSa5iyK7tGMV

创建后（未初始化）:
账户是否存在: false
账户信息: null

初始化后:
账户是否存在: true
账户信息: { owner: 'System Program', lamports: 1000000000 }
```
浏览器地址 https://solscan.io/account/HYpaTT1EsmfyLBWjNcP6AhEW4uaiS8rKXSa5iyK7tGMV?cluster=custom&customUrl=http://127.0.0.1:8899/

可以看到，在账户未初始化时，账户信息是空， 因此浏览里没有显示owner 属性。

而账户在初始化后， `Owner` 属性就是 `System Program`，所以这里显示的 Owner 属性是正常的状态。这里还有一个账户余额信息。

![image-20250107162031002](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250107162031002.png)

参考资料：https://solana.com/zh/docs/core/accounts
