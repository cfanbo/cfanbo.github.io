---
title: Solana中如何实现转账
date: 2025-01-10T10:02:20+08:00
type: post
url: /posts/how-to-transfer-in-solana
categories:
- 程序开发
- web3
tags:
- solana
- web3
---



在 Solana 中转账有两种方式。

# 修改结构体字段

一种是在不调用（`invoking` ）系统程序（ `System Program` ）的情况下，将 `lamports` 从一个账户转移到另一个账户。它的实现是直接通过修改结构体的 `lamports` 字段值来实现的。

这种方法可以实现将 lamports 从任何由您的程序**拥有**的账户转移到任何账户。[文档](https://solana.com/zh/developers/cookbook/programs/transfer-sol)

```rust
/// Transfers lamports from one account (must be program owned)
/// to another account. The recipient can by any account
fn transfer_service_fee_lamports(
    from_account: &AccountInfo,
    to_account: &AccountInfo,
    amount_of_lamports: u64,
) -> ProgramResult {
    // Does the from account have enough lamports to transfer?
    if **from_account.try_borrow_lamports()? < amount_of_lamports {
        return Err(CustomError::InsufficientFundsForTransaction.into());
    }
    // Debit from_account and credit to_account
    **from_account.try_borrow_mut_lamports()? -= amount_of_lamports;
    **to_account.try_borrow_mut_lamports()? += amount_of_lamports;
    Ok(())
}
```

上面是直接操作底层数据库的 `AccountInfo.lamports` 字段，不过这种方式在链上没有变更历史，因此一般不推荐使用这种转账方式。

这种方式的优点是直接且高效，避免了通过外部系统程序（如 `SystemProgram::transfer`）进行转账，因此可能减少一些复杂性和资源消耗

# CPI 调用转账

另一种是通过系统调用（System Program）来实现转账，一般是指CPI 跨程序调用，一般是通过 [invoke](https://docs.rs/solana-sdk/latest/solana_sdk/program/fn.invoke.html) 和 [invoke_signed](https://docs.rs/solana-sdk/latest/solana_sdk/program/fn.invoke_signed.html) 两个函数来实现(还有两个不常用的 [invoke_signed_unchecked](https://docs.rs/solana-sdk/latest/solana_sdk/program/fn.invoke_signed_unchecked.html) 和 [invoke_unchecked](https://docs.rs/solana-sdk/latest/solana_sdk/program/fn.invoke_unchecked.html) 函数)。同时还有一些函数也可以实现CPI转账，但它们底层均是对这两个函数的封装，如 `anchor_lang::system_program::transfer()`。

跨程序调用参考 https://solana.com/zh/developers/cookbook/programs/cross-program-invocation。

CPI 转账调用示例 https://beta.solpg.io/github.com/ZYJLiu/doc-examples/tree/main/cpi-pda

这在 Solana 中是推荐的使用方法，CPI 是开发中经常使用到的一种方法，很值的了解学习它的用法。

对于转账的场景，有一个情况需要考虑到。如果支付账户是否为 ATA 账户，因为转账需要交易签名，而ATA账户是没有私钥的，因此没有办法直接调用 `invoke` 函数转账，只能通过 `invoke_signed` 函数调用，同时指定生成ATA账号的签名信息才可以。如 

```rust
    pub fn sol_transfer_two(ctx: Context<SolTransfer>, amount: u64) -> Result<()> {
        let from_pubkey = ctx.accounts.pda_account.to_account_info();
        let to_pubkey = ctx.accounts.recipient.to_account_info();
        let program_id = ctx.accounts.system_program.to_account_info();

        let seed = to_pubkey.key();
        let bump_seed = ctx.bumps.pda_account;
        let signer_seeds: &[&[&[u8]]] = &[&[b"pda", seed.as_ref(), &[bump_seed]]]; // 账号签名

        let instruction =
            &system_instruction::transfer(&from_pubkey.key(), &to_pubkey.key(), amount);

        invoke_signed(instruction, &[from_pubkey, to_pubkey, program_id], signer_seeds)?; // 指定签名
        Ok(())
    }
```

这里 `from_pubkey` 是一个 `ATA` 账号，它没有私钥，只能通过组装 `signer_seeds` 作为签名实现转账功能。

对于CPI 的调用，也分几种情况，有兴趣的话可以看这里 https://www.anchor-lang.com/docs/basics/cpi#example-explanation-1

# 区别

第一种方法比较高效，但有一个缺点，那就是没有交易记录，你在链上看不到账户余额的变化，因此平时都推荐使用第二种CPI的方式转账。

# 参考文档

- https://solana.com/zh/developers/cookbook/programs/transfer-sol
- https://solana.com/developers/courses/connecting-to-offchain-data/oracles#8-withdraw
- https://beta.solpg.io/github.com/ZYJLiu/doc-examples/tree/main/cpi-pda
- https://solana.com/developers/guides/getstarted/how-to-cpi-with-signer
- https://www.anchor-lang.com/docs/basics/cpi