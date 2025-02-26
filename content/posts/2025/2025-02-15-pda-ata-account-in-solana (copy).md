---
title: Solana中 PDA、ATA 与 普通Account 的区别与关系
date: 2025-02-15T10:02:20+08:00
type: post
url: /posts/pda-ata-account-in-solana
categories:
- 程序开发
- web3
tags:
- solana
- web3
---




# 普通账户地址

对于账户地址的创建是由一个密钥对来生成的，但在Solana中账户地址与以太坊中的账户地址还是有一些区别的。

## 以太坊账户地址

以太坊账户地址的生成过程：

1. 通过私钥生成公钥
2. 对公钥进行 Keccak-256 哈希
3. 取哈希值的最后 160 位（20 字节）作为地址
4. 将地址以 `0x` 开头，并根据需要选择是否使用 EIP-55 格式

> 地址中通常是小写字母，但也有大写字母的变种，称为 **EIP-55** 格式。在 EIP-55 中，某些字符会根据哈希值的大小写进行区分，从而增加地址的错误检查能力。

## Solana 账户地址

Solana账户地址的生成过程：

1. 通过私钥生成公钥，一般通过调用  Keypair.generate() 生成
2. 公钥直接映射为账户的地址，长度为 32 字节
3. 为了使用方便，一般对其进行 Base58 编码，将公钥转换为地址字符串

代码：

```typescript
const { Keypair } = require('@solana/web3.js');

// 生成一个新的密钥对
const keypair = Keypair.generate();

// 获取公钥，实际上就是账户地址
const publicKey = keypair.publicKey;

// 转换公钥为 Base58 编码的字符串（即账户地址）
const address = publicKey.toBase58();

console.log("Solana Account Address (Base58):", address);
```

可以看到 Solana 中的账户地址就是公钥，平时使用的账户地址，一般都是指 Base58 编码后的字符串。

## 总结

| 特点         | **Solana**                                     | **以太坊**                                    |
| ------------ | ---------------------------------------------- | --------------------------------------------- |
| **生成**     | 公钥直接作为地址                               | 公钥通过 Keccak-256 哈希，取哈希的最后 160 位 |
| **长度**     | 32 字节（256 位），Base58 编码后 43 字符       | 20 字节（160 位），16 进制编码后 40 字符      |
| **哈希算法** | 无哈希处理，直接使用公钥                       | 使用 Keccak-256 哈希处理公钥                  |
| **编码方式** | Base58 编码                                    | 16 进制编码，`0x` 前缀                        |
| **示例**     | `4Erv6yZoXckm6QqsbU6y6TbS8o8wVkAXw7KwHjsAVB9D` | `0x5b1a49d2c631eeed5d2b8e6abdd87e07d8e1a3b3`  |

虽然 Solana 和以太坊都基于公钥生成地址，但它们在地址生成的具体方式、哈希算法和编码格式上存在不同。

Solana 地址直接使用公钥并通过 Base58 编码，而以太坊则先对公钥进行哈希处理，并且使用 16 进制编码生成地址。

# PDA 账户地址

对于PDA账户地址的样子与普通账户是一样的，用户根本无法肉眼看出来一个地址是哪一种账户类型。

当创建一个交易时，一般都需要签名，对于普通账户，可以使用私钥的对交易签名。而对于 PDA 地址，虽然没有私钥，但在 Soalan中它允许以编程的方式对交易进行签名。

一个普通的账户地址是指在Ed25519 曲线（椭圆曲线加密）上的一个点，如 Alice 和 Bob 它们都属于基本账户，都有公钥和私钥。

![On Curve Address](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/address-on-curve.svg)

而 PDA 账户是表示在 Ed25519 曲线之外的点，它只有公钥，没有私钥。需要借助一些预定义的种子集来推导生成。

![Off Curve Address](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/address-off-curve.svg)

PDA 地址生成公式

```ini
PDA = pda_hash(program_id, seeds)
```

其中 program_id 字段是指当前程序ID（合约）, 以保证生成的pda账户只对当前程序有效。seeds 它是一些种子的集合，如 `["abc", "xyz", "888"]`。

示例

```typescript
import { PublicKey } from "@solana/web3.js";

const programId = new PublicKey("11111111111111111111111111111111");
const string = "helloWorld";

const [PDA, bump] = PublicKey.findProgramAddressSync(
  [Buffer.from(string)],
  programId,
);

console.log(`PDA: ${PDA}`);
console.log(`Bump: ${bump}`);
```

> 对于 bump 字段，它是由于在生成地址过程中，程序自动引入一个bump种子，它是这个0-255的数字。首先计算时先从255开始，如果计算得出的地址不是有效的 PDA 地址，则减少一，直到找到有效的地址为止。

PDA 地址的计算过程为

![PDA Derivation](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/pda-derivation.svg)



# ATA账户

ATA（Associated Token Account） 账户是一个特殊的 PDA 账户，同样也是没有私钥的。它与pda 账户唯一的区别在于在生成pda地址的过程中，添加了一个新的种子，而这个种子其实就是 `Mint Account` 地址，而 `Mint Account` 只有 `spl-token` 才有的概念，因此对于 ATA 账户只能用在 `spl-token` 代币相关的地方。

其地址生成是使用 Owner 地址和 Mint Account 地址确定性通过 PDA 派生的，它是一个 spl-token 代币地址。可以将关联代币账户视为特定**铸币**和**所有者**的“默认”代币账户。

![Associated Token Account](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/associated-token-account.svg)

示例

```typescript
import { getAssociatedTokenAddressSync } from "@solana/spl-token";

const associatedTokenAccountAddress = getAssociatedTokenAddressSync(
  USDC_MINT_ADDRESS,
  OWNER_ADDRESS,
);
```

函数定义 https://github.com/solana-labs/solana-program-library/blob/d72289c79/token/js/src/state/mint.ts#L190

```typescript
/**
 * Get the address of the associated token account for a given mint and owner
 *
 * @param mint                     Token mint account
 * @param owner                    Owner of the new account
 * @param allowOwnerOffCurve       Allow the owner account to be a PDA (Program Derived Address)
 * @param programId                SPL Token program account
 * @param associatedTokenProgramId SPL Associated Token program account
 *
 * @return Address of the associated token account
 */
export function getAssociatedTokenAddressSync(
    mint: PublicKey,
    owner: PublicKey,
    allowOwnerOffCurve = false,
    programId = TOKEN_PROGRAM_ID,
    associatedTokenProgramId = ASSOCIATED_TOKEN_PROGRAM_ID
): PublicKey {
    if (!allowOwnerOffCurve && !PublicKey.isOnCurve(owner.toBuffer())) throw new TokenOwnerOffCurveError();

    const [address] = PublicKey.findProgramAddressSync(
        [owner.toBuffer(), programId.toBuffer(), mint.toBuffer()],
        associatedTokenProgramId
    );

    return address;
}
```

可以看到，这里和生成 PDA 地址时调用的函数都是 [PublicKey.findProgramAddressSync()](https://solana-labs.github.io/solana-web3.js/classes/PublicKey.html#findprogramaddresssync) 函数，唯一的区别就是参数不一样而已。

这里主要有两点：

1. 种子使用固定的 owner、 mint 和 programId，用户没有办法自定义种子。

2. 第二个参数程序ID 是 **associatedTokenProgram**，而不是原来的 **TokenProgram** 或 **Token 2022 Program**。

对于底层最终的实现，就是将一系的值连接起来并sha256计算，使用这个结果创建 PublicKey 对象，实现代码参考 [createProgramAddressSync(seeds: Array<Buffer | Uint8Array>， programId: PublicKey)](https://github.com/solana-labs/solana-web3.js/blob/4e9988cfc561f3ed11f4c5016a29090a61d129a8/src/publickey.ts#L164-L189) 函数。

从ATA账号的创建原理可以看到，ATA 账户也是一个PDA账户，只不过是一个特殊的PDA账户，它只能用在 spl-token 代币程序中，同时与 associatedTokenProgram 有一定的关系。

# 参考资料

- https://solana.com/zh/docs/core/pda
- https://solana.com/zh/docs/core/tokens#associated-token-account
