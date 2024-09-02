---
title: "Rust学习教程清单"
date: 2024-08-10T14:31:27+08:00
type: post
toc: true
url: /posts/rust-learning-tutorial
categories:
  - 程序开发
tags:
  - rust
 
---

今年又一次重新学习RUST这门编程语言，并从零开发了一个kv存储系统 [minKV](https://github.com/cfanbo/minkv)，慢慢的越来越有感觉了。

本篇主要将日常学习中收集的一些入门教程进行一下汇总，希望对于一些想学习这门开发语言的同学有所帮助。

下面教程按照推荐顺序，由浅到深依次列出。以下内容将不定期的更新，请自行收藏。 如果您有更多好的教程的话，也可以在评论区列出，大家相互学习。

# 入门教程

1. [Rust 程序设计语言 https://doc.rust-lang.org/book/](https://doc.rust-lang.org/book/) / ([中文版](https://kaisery.github.io/trpl-zh-cn/))

   官方教程，强烈推荐，同时还有非官方翻译的中文版。遗憾的是仍有些概念介绍的有一些模糊，可通过下方的一些资料自行补习。

2. https://play.rust-lang.org/   在线 RUST 程序 Playground，类似golang的 Playground，非常的方便
3. [Rust Language Cheat Sheet](https://cheats.rs/#data-layout)  看完官方的教程后，紧接着就看这篇，先了解一些内存布局，后面再看其它教程就更容易理解了
4. [The Cargo Book](https://doc.rust-lang.org/cargo/index.html#the-cargo-book) Cargo 是RUST 中的包管理工具，开发必备工具
5. https://rust-lang-nursery.github.io/rust-cookbook/ /  ([Rust Cookbook 中文版](https://rustwiki.org/zh-CN/rust-cookbook/))
6. [Rust语言圣经(Rust Course)](https://course.rs/about-book.html)
7. [rust-by-example 在实践中学 Rust ](https://rustwiki.org/rust-by-example/)
8. [Rust By Practice Rust语言实战](https://practice.course.rs/why-exercise.html) / ([中文版](https://practice-zh.course.rs/why-exercise.html))
9. [100 Exercises To Learn Rust](https://rust-exercises.com/100-exercises/01_intro/00_welcome)
10. [Comprehensive Rust](https://google.github.io/comprehensive-rust/) 由Google 的 Android 团队开发的免费 Rust 课程
11. [Tour of Rust](https://tourofrust.com/index.html)

# 进阶教程

1. [Command Line Applications in Rust](https://rust-cli.github.io/book/index.html)
2. [The Rustonomicon](https://doc.rust-lang.org/nomicon/)
3. [Rust 宏小册](https://zjp-cn.github.io/tlborm/#rust-宏小册)
4. [Rust and WebAssembly](https://rustwasm.github.io/docs/book/) WASM开发手册



# 开发规范

- [PingCAP Style Guide](https://pingcap.github.io/style-guide/rust/)



# 视频教程

如果习惯看视频学习的话，可以看以下列出的一些B站UP主视频，前两个UP主的视频与一些教程是同步的，以视频的形式重新对教程进行了讲解。

 UP主 [软件工艺师](https://space.bilibili.com/361469957)

1. [Rust编程语言入门教程] (https://www.bilibili.com/video/BV1hp4y1k7SV/) 目前B站点击量Top1的视频，如果你看完 Rust程序设计语言，再通过视频看一篇，可能会有一些新的理解
1. [Rust 编程语言中级教程 -- 接口设计的建议] (https://www.bilibili.com/video/BV1Pu4y1Z7dT)

UP主 [清华邓博士](https://space.bilibili.com/504069720/channel/collectiondetail?sid=3642485)

1. [The Golden Rust语言  系列](https://space.bilibili.com/504069720/channel/collectiondetail?sid=3642485) 主要针对Goolge Andrioid团队出品的 [Comprehensive Rust](https://google.github.io/comprehensive-rust/)  进行视频讲解

UP主 [wharton0](https://space.bilibili.com/35891473)  经常分享一些不错的文章，个人觉得很不错

UP主 [ppt的bug](https://space.bilibili.com/294056147/)

UP主 [木子牙膏](https://space.bilibili.com/240421008/)



# RUST crate 仓库

- https://crates.io/
- https://lib.rs/



# 嵌入式开发

1. [The Embedded Rust Book](https://doc.rust-lang.org/stable/embedded-book/) 嵌入式开发手册



以上教程基本包含了目前市场上绝大多数教程，后期如果有更好的教程将不定期的进行更新。另外如果有上那个什么网的条件的话，youtube.com 上有不少优化的视频，可自行查找，目前B站许多视频都是从这个网站上般过来的。
