---
title: Solana中的 Native Programs
date: 2025-02-25T10:02:20+08:00
type: post
url: /posts/native-programs-in-the-solana-runtime
toc: true
categories:
- 程序开发
- web3
tags:
- solana
- web3
---

在Solana中有一些少量的内置原生程序，它们的程序ID格式一般为 `xxx11111111111111111111111111111111`。它们与第三方自定义程序不同，原生程序是 **validator node** 运行所必须的一部分。同时它们也是集群升级的一部分，这些升级可能包括新功能的添加、BUG修复，又或者是性能的优化提升。这些内置原生程序极少的发生变化，目前提供的所有内置原生程序可以在 https://docs.anza.xyz/runtime/program 找到。

其中有两个非常重要的程序很值得我们关注，因为我们在开发中，需要经常用到他们。



# System Program

- 使用：
 - 创建新帐户
 - 分配帐户数据
 - 将帐户分配给所属的程序，即指定账户与程序的对应关系
 - 转账最小 lamports 给账户(租金)，防止账户被系统回收

- Program id: `11111111111111111111111111111111`
- Instructions: [SystemInstruction](https://docs.rs/solana-program/2.2.0/solana_program/system_instruction/enum.SystemInstruction.html)



# BPF Loader

- 作用：
  - 部署所有自定义程序，并将其作为的所有自定义程序的 owner
  - 升级和执行链条上的程序
- Program id: `BPFLoaderUpgradeab1e11111111111111111111111`
- Instructions: [LoaderInstruction](https://docs.rs/solana-sdk/2.2.0/solana_sdk/loader_upgradeable_instruction/enum.UpgradeableLoaderInstruction.html)

BPF可升级加载器将自己标记为它为存储程序而创建的可执行文件和程序数据帐户的“Owner”。当用户通过程序id调用指令时，Solana运行时会加载您的程序及其所有者BPF Upgradeable Loader。然后，运行时将程序传递给BPF可升级加载器，以便其处理指令。

对于 `BPFLoaderUpgradeab1e11111111111111111111111`程序它是作为 Token2022 专用的，它是对 Token 的扩展版，对于 Token 对应的内置程序是 `BPFLoader2111111111111111111111111111111111`，两者的区别是后者是无法升级的。



# 关系图

对于这些内置原生程序与三方程序的关系图大概是这个样子的。

![image-20250305184730290](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250305184730290.png)



# 参考资料

- https://docs.anza.xyz/runtime/program
- https://solana.com/zh/docs/core/accounts#native-programs
