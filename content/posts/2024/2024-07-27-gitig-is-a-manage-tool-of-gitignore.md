---
title: 一款管理 .gitignore 的CLI工具- gitig
date: 2024-07-27T15:31:52+08:00
type: post
toc: true
url: /posts/gitig-is-a-manage-tools-of-gitignore
categories:
  - 程序开发
tags:
  - gitig
  - rust
  - git
---

`gitig` 是一款基于  https://github.com/github/gitgnore 仓库开发的`.gitignore` 客户端CLI 管理工具，也是每个开发者必不可少的提高工作效率的必务工具。

它基于官方仓库丰富的 `.gitignore` 数据源，帮助开发者快速实现添加各类开发项目的git版本控制忽略文件清单。

# 开发背景

工作中，经常需要开发各类项目，如基于 vscode 编写  rust 项目工具，对于版本控制基本会采用 `Git`，这时为了方便进行管理控制，有些项目文件是不需要提交到git仓库的，因此需要将一些文件写到 `.gitignore` 文件忽略。

如果手动编辑 `.gitignore`文件将这些忽略项写进去的话，则可能会有一些忽略项被遗忘或写错，这时如果有一些工具可以将行业能用的配置项一键写入 `.gitignore` 文件似乎是一个不错的主意。

其中著名的 https://github.com/github/gitgnore  仓库就是一个专门收集各类开发语句或IDE 需要忽略的 `.gitignore` 推荐配置，目前也一直在更新。

官方对它的介绍

> This is GitHub’s collection of [`.gitignore`](http://git-scm.com/docs/gitignore) file templates. We use this list to populate the `.gitignore` template choosers available in the GitHub.com interface when creating new repositories and files.

收集的各种开发语言或ide 推荐忽略文件

![image-20240730155036672](https://blogstatic.haohtml.com//uploads/2024/04/image-20240730155036672.png)

 `gitig`就是一款专用解决这个问题的工具，软件安装后自动包含数据源信息，后期随着数据源仓库的更新也会更新发布新版本，它不需要联网支持直接离线操作，只有不到 `2M` 大小。

# 功能介绍

对于它的安装方法也很简单，可以参考 https://github.com/cfanbo/gitig.git 介绍，安装完成后，查看版本号

```shell
$ gitig -v
gitig v0.0.3-f306ec4
```

## 添加配置项

最常用的操作就是将仓库推荐类型的忽略清单添加到本地的  `.gitignore` 文件，如上面用 vscode 开发 rust 项目，这时就需要将与 VSCode 相关忽略项与 Rust 语言相关忽略项一并添加到本地 `.gitignore`文件。

```shell
$ gitig add rust visualstudiocode
!.vscode/*.code-snippets
!.vscode/extensions.json
!.vscode/launch.json
!.vscode/settings.json
!.vscode/tasks.json
**/*.rs.bk
*.pdb
*.vsix
.history/
.vscode/*
Cargo.lock
debug/
target/

本次成功更新 13 个忽略条目
```

> 这里的  `.gitignore` 主要指当前工作目录下的文件

如果你是 `java` 开发者，也可以这样

```shell
$ gitig add java
*.class
*.ctxt
*.ear
*.jar
*.log
*.nar
*.rar
*.tar.gz
*.war
*.zip
.mtj.tmp/
hs_err_pid*
replay_pid*

本次成功更新 13 个忽略条目
```

上面类型名称不区分大小写，`JAVa`  和 `java` 是一样的。

## 搜索配置项

上面我们添加vscode 配置项的时候，发现名字太长，不好记，这时可利用搜索功能

```shell
$ gitig search visual
1: visualstudio
2: visualstudiocode
```

这时会找到系统支持的类型。

## 查看忽略项

如果你想查看某类型的忽略清单，则可以

```shell
$ gitig show rust
1: **/*.rs.bk
2: *.pdb
3: Cargo.lock
4: debug/
5: target/
```

## 查看所有支持项

如果你想知道当前版本支持多少类型，则可c以 

```shell
$ gitig list
1: actionscript
2: ada
3: agda
4: al
5: alteryx
...
```

 所有项目名称这里以小写输出。

# 其它

当前版本存在一个小问题，就是有些时候添加类型不方便，如上面的 `visualstudiocode` 时，平时我们都是习惯了使用 `vscode`代替名称，这里用了完整的名字，稍微有点不方便，不过还好这是很少见的情况，还可以接受的。

另外还有一个现象，官方推荐的忽略配置项可能与当前项目的忽略配置项不完全一致，可能有些配置项在当前项目中是不需要的，也被添加到 `.gitignore`文件了。不过多添加几个配置项这并不影响项目的开发，因此也可以忽略不计的。

# 参考资料

- https://github.com/cfanbo/gitig
