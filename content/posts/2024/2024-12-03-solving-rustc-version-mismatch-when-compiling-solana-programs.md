---
title: 解决编译solana程序 rustc版本号过低的问题
date: 2024-12-03T13:52:20+08:00
type: post
url: /posts/solving-rustc-version-mismatch-when-compiling-solana-programs
toc: true
categories:
- 程序开发
- web3
tags:
- solana
- web3
---

本方主要介绍在编译solana程序时，提示 rustc 版本号过低无法编译通过的问题。

# 问题描述

在参考官方教程 https://github.com/solana-developers/program-examples/tree/main/basics/favorites/native 在本地执行命令

```shell
➜  native git:(main) ✗ cargo build-sbf --manifest-path=./program/Cargo.toml --sbf-out-dir=./program/target/so
```
报错

```shell
➜  native git:(main) ✗ cargo build-sbf --manifest-path=./program/Cargo.toml --sbf-out-dir=./program/target/so
error: package `solana-program v2.1.7` cannot be built because it requires rustc 1.79.0 or newer, while the currently active rustc version is 1.75.0-dev
Note that this is the rustc version that ships with Solana tools and not your system's rustc version. Use `solana-install update` or head over to https://docs.solanalabs.com/cli/install to install a newer version.
Either upgrade to rustc 1.79.0 or newer, or use
cargo update solana-program@2.1.7 --precise ver
where `ver` is the latest version of `solana-program` supporting rustc 1.75.0-dev
```

翻译过来就是使用的 rustc 版本号过低，至少需要1.9.0 或更高才可以，这里的rustc版本与并非本地安装的rustc版本号。

在首次执行 `cargo build-sbf` 时，会自动下载对应的工具链到本地，一般在 `$HOME/.local/share/solana/install/active_release/bin/  `目录， 这里查看一下它的版本号

```shell
➜  native git:(main) ✗ ls /Users/sxf/.local/share/solana/install/active_release/bin/
agave-install         agave-validator       cargo-test-sbf        sdk                   solana-dos            solana-gossip         solana-net-shaper     solana-tokens
agave-install-init    agave-watchtower      deps                  solana                solana-faucet         solana-keygen         solana-stake-accounts spl-token
agave-ledger-tool     cargo-build-sbf       rbpf-cli              solana-bench-tps      solana-genesis        solana-log-analyzer   solana-test-validator

➜  native git:(main) ✗ /Users/sxf/.local/share/solana/install/active_release/bin/cargo-build-sbf --version
solana-cargo-build-sbf 2.0.21
platform-tools v1.42
rustc 1.75.0
```

> cargo build-sbf 命令对应的是可执行程序 cargo-build-sbf

可以看到它的rustc版本号是 `1.75.0`，版本还是比较低的。

按照错误提示信息，需要使用 `solana-install update` 命令才可以，不过现在solana install 已经被 `agave-install` 所代替(官方文档 https://github.com/anza-xyz/agave/wiki/Agave-v2.0-Transition-Guide)，因此执行

```shell
➜  native git:(main) ✗ agave-install update
Install is up to date. 99ac010 is the latest commit for stable

➜  native git:(main) ✗agave-install --version
agave-install 2.0.21 (src:99ac0105; feat:607245837, client:Agave)
```

当前已经是最新版本 `2.0.21`。这就有点让人迷茫了，最新的版本竟然还是用的rustc 1.75.0，这与官方rustc编译器版本相差的太远了。

这里错误信息里还推荐了另一种解决办法，就是直接更新 solana-program 的版本号的话，不过考虑到依赖库太多，改动后极大可能导致无法运行，因此这种方案就直接放弃了，还有其它办法吗？

# 解决办法

查看官方 https://github.com/anza-xyz/agave/releases 了解到，通过 agave-install update 更新后的版本，并非官方发布的最新版本号，在github上发布的目前最新的是 `v2.1.7`，且它只适合在 Testnet 上使用，这比目前用的版本号高不少，报着试一试的态度试一下手动指定版本号

```shell
➜  native git:(main) ✗ agave-install init v2.1.7
// 将自动从官方下载工具链安装包并解压到本地目录里
➜  native git:(main) ✗ agave-install --version
agave-install 2.1.7 (src:c79c3c9e; feat:1793238286, client:Agave)
```

我们再确认一下工具链版本号

```shell
➜  native git:(main) ✗ cargo build-sbf --version
solana-cargo-build-sbf 2.1.7
platform-tools v1.43
rustc 1.79.0
```

发现rustc 版本号升级到了 v1.79.0, 正好满足最低运行版本，再试一下前面的编译命令

```
➜  native git:(main) ✗ cargo build-sbf --manifest-path=./program/Cargo.toml --sbf-out-dir=./program/target/so
```

发现编译通过，只是存在一些waring而已。

现在我们本地存在两个版本号的工具链，即两套编译环境

```shell
➜  native git:(main) ✗ agave-install list
stable-99ac010533749c74b308bfd932cbf92f9f81dffa
2.1.7 (current)
```

以后需要的话，可以通过 `agave-install init VER`命令进行切换即可。

这个错误在 https://github.com/solana-labs/solana/issues/33504  可以找到类似的讨论，遗憾的是至今很少见到彻底的解决办法。前几天也是为解决这个问题花费了好久，也没有找到最终解决方案。 试图降低 solana-program 的版本号，最后发现程序无法正常运行，最终也是暂时搁浅了。



# 参考资料

- https://github.com/anza-xyz/agave
- https://github.com/anza-xyz/agave/wiki/Agave-v2.0-Transition-Guide
- https://solana.com/docs/intro/installation#install-the-solana-cli
- https://stackoverflow.com/questions/77928538/rust-version-problem-when-running-cargo-build-bpf
