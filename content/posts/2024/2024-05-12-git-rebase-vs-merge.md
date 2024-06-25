---
title: git 操作中那些常常被忽略的用法
date: 2024-05-12T20:23:15+08:00
type: post
toc: true
url: /posts/git-rebase-vs-merge-amend-squash
categories:
  - 程序开发
tags:
  - git
---



# git merge 与 git rebase  的区别

对于两者的区别，网上已经有很多文章做了介绍，不过有些初学者没有亲自实验过，多数也是作为八股文死记硬背而已。本文为了让大家彻底搞懂两者的区别，所以搞了一个实验环境并模拟了一些真实环境的操作。

实验环境主要用到两个分支，其中 main 分支做了主要分支，而 dev 作为开发功能分支。

> 在生产环境中需要选择合适的分支，这里只是实验环境，所以分支名不是本文关注的重点。 

| main                                                         | dev                                                          | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| mkdir git-demo && cd git-demo && git init                    |                                                              |                                                              |
| touch 1.txt && git add . && git commit -m 'add 1.txt'        |                                                              | f3c82ba                                                      |
| touch 2.txt && git add . && git commit -m 'add 2.txt'        |                                                              | eb061da                                                      |
| touch 3.txt && git add . && git commit -m 'add 3.txt'        |                                                              | 0e71e97                                                      |
| ➜ git log --oneline<br />0e71e97 (HEAD -> main) add 3.txt<br/>eb061da add 2.txt<br/>f3c82ba add 1.txt |                                                              |                                                              |
|                                                              | git checkout -b dev                                          |                                                              |
|                                                              | ➜ git log --oneline<br />0e71e97 (HEAD -> dev, main) add 3.txt<br/>eb061da add 2.txt<br/>f3c82ba add 1.txt |                                                              |
|                                                              | touch a.txt && git add . && git commit -m 'add a.txt'        | 42b08d4                                                      |
|                                                              | touch b.txt && git add . && git commit -m 'add b.txt'        | 17abda2                                                      |
|                                                              | touch c.txt && git add . && git commit -m 'add c.txt'        | 2405133                                                      |
|                                                              | ➜ git log --oneline<br />2405133 (HEAD -> dev) add c.txt<br/>17abda2 add b.txt<br/>42b08d4 add a.txt<br/>0e71e97 (main) add 3.txt<br/>eb061da add 2.txt<br/>f3c82ba add 1.txt |                                                              |
| git checkout main                                            |                                                              |                                                              |
| touch 4.txt && git add . && git commit -m 'add 4.txt'        |                                                              | a1dede3                                                      |
|                                                              | git checkout dev                                             |                                                              |
|                                                              | touch d.txt && git add . && git commit -m 'add d.txt'        | 63bf443                                                      |
|                                                              | touch e.txt && git add . && git commit -m 'add e.txt'        | a3180d0                                                      |
|                                                              | ➜ git log --oneline<br />a3180d0 (HEAD -> dev) add e.txt<br/>63bf443 add d.txt<br/>2405133 add c.txt<br/>17abda2 add b.txt<br/>42b08d4 add a.txt<br/>0e71e97 add 3.txt<br/>eb061da add 2.txt<br/>f3c82ba add 1.txt |                                                              |
| git checkout main                                            |                                                              |                                                              |
| touch 5.txt && git add . && git commit -m 'add 5.txt'        |                                                              | 95fb908                                                      |
|                                                              | git checkout dev                                             |                                                              |
|                                                              | touch f.txt && git add . && git commit -m 'add f.txt'        | 527b20f                                                      |
|                                                              | ➜ git log --oneline<br />527b20f (HEAD -> dev) add f.txt<br/>a3180d0 add e.txt<br/>63bf443 add d.txt<br/>2405133 add c.txt<br/>17abda2 add b.txt<br/>42b08d4 add a.txt<br/>0e71e97 add 3.txt<br/>eb061da add 2.txt<br/>f3c82ba add 1.txt |                                                              |
| git checkout main                                            |                                                              |                                                              |
| ➜ git log --oneline<br />c8459e8 (HEAD -> main) add 5.txt<br/>f992a80 add 4.txt<br/>0e71e97 add 3.txt<br/>eb061da add 2.txt<br/>f3c82ba add 1.txt |                                                              |                                                              |
| ➜ git rebase dev                                             |                                                              | git rebase 合并分支                                          |
| ➜ git log --oneline<br />95fb908 (HEAD -> main) add 5.txt<br/>a1dede3 add 4.txt<br/>527b20f (dev) add f.txt<br/>a3180d0 add e.txt<br/>63bf443 add d.txt<br/>2405133 add c.txt<br/>17abda2 add b.txt<br/>42b08d4 add a.txt<br/>0e71e97 add 3.txt<br/>eb061da add 2.txt<br/>f3c82ba add 1.txt |                                                              | 在dev 分支添加c.txt后，在main分支添加了4.txt。<br /><br /><br />在 dev 分支添加 e.txt 后，在main分支添加了5.txt |

注意实际提交顺序与合并后`git log` 记录显示的顺序差异：在 `git log `中可以看出在 `a.txt - e.txt` 之间是连续的，尽管在其提交过程中在main分支也进行了一些提交，git log 的最后才将在main分支中的修改追加在 dev 分支历史记录的后面（4.txt 和 5.txt）。
总结一下，就是当一个分支dev 从另一个基分支 main 拆分出来后。当在基分支通过 git rebase 合并分支时，会将被合并分支所有commit id 顺序提交到git log 后面。如果在其间main分支也做了一些commit, 时会在合并之后再追加到后面。因此如果有冲突的话，合并的时候要确认清楚哪些是最新的代码，这个与使用 git merge  命令是不一样，它是基于时间点顺序合并的。



我们还基于上面的来确认一下,

| main                                                         | dev  |                                                              |
| ------------------------------------------------------------ | ---- | ------------------------------------------------------------ |
| ➜ git reflog<br />95fb908 (HEAD -> main) HEAD@{0}: rebase (finish): returning to refs/heads/main<br/>95fb908 (HEAD -> main) HEAD@{1}: rebase (pick): add 5.txt<br/>a1dede3 HEAD@{2}: rebase (pick): add 4.txt<br/>527b20f (dev) HEAD@{3}: rebase (start): checkout dev<br/>c8459e8 HEAD@{4}: checkout: moving from dev to main<br/>527b20f (dev) HEAD@{5}: checkout: moving from main to dev<br/>c8459e8 HEAD@{6}: checkout: moving from dev to main<br/>527b20f (dev) HEAD@{7}: commit: add f.txt<br/>a3180d0 HEAD@{8}: checkout: moving from main to dev<br/>c8459e8 HEAD@{9}: commit: add 5.txt<br/>f992a80 HEAD@{10}: checkout: moving from dev to main<br/>a3180d0 HEAD@{11}: commit: add e.txt<br/>63bf443 HEAD@{12}: commit: add d.txt<br/>2405133 HEAD@{13}: checkout: moving from main to dev<br/>f992a80 HEAD@{14}: commit: add 4.txt<br/>0e71e97 HEAD@{15}: checkout: moving from dev to main<br/>2405133 HEAD@{16}: commit: add c.txt<br/>17abda2 HEAD@{17}: commit: add b.txt<br/>42b08d4 HEAD@{18}: commit: add a.txt<br/>0e71e97 HEAD@{19}: checkout: moving from main to dev<br/>0e71e97 HEAD@{20}: commit: add 3.txt<br/>eb061da HEAD@{21}: commit: add 2.txt<br/>f3c82ba HEAD@{22}: commit (initial): add 1.txt |      | 在 main 分支通过 git reflog 查看所有操作记录                 |
| git reset --hard c8459e8                                     |      | 通过 git reset --hard 恢复原工作区。现在恢复到执行 git rebase 命令前的状态 |
| ➜ git log --oneline<br />c8459e8 (HEAD -> main) add 5.txt<br/>f992a80 add 4.txt<br/>0e71e97 add 3.txt<br/>eb061da add 2.txt<br/>f3c82ba add 1.txt |      |                                                              |
| ➜ git merge dev                                              |      | git merge 合并分支                                           |
| ➜ git log --oneline<br />42d66af (HEAD -> main) Merge branch 'dev'<br/>527b20f (dev) add f.txt<br/>c8459e8 add 5.txt<br/>a3180d0 add e.txt<br/>63bf443 add d.txt<br/>f992a80 add 4.txt<br/>2405133 add c.txt<br/>17abda2 add b.txt<br/>42b08d4 add a.txt<br/>0e71e97 add 3.txt<br/>eb061da add 2.txt<br/>f3c82ba add 1.txt |      | 1. git log 顺序与实际操作中完全一样<br />2. 每次执行git mgerge 命令都会生成一个新的合并commit id |

可以看到 git log 显示commit 顺序与实际操作中的顺序是完全一样的，因它他们是因于时间线进行合并排序的，这是与使用 git rebase 本质的区别。



# 修正操作 amend commit

在 Git 中，`amend commit` 是一个非常有用的功能，它常常用于修改最近一次的提交。这种操作可以用来修正上一次提交中的错误，比如遗漏了某些文件，提交消息有误，或者已提交的内容包含一些bug或测试代码，需要更新已经提交的内容。

`amend commit` 通过在现有的提交之上创建一个新的提交来实现，这样看起来就像是修改了之前的提交，而不会留下多余的提交历史。这个操作将会自动删除上一次的commit id，并创建一个新的 commit id。

下面是实验环境

```shell
mkdir git-demo && cd git-demo
git init
echo 'a' > a.txt && git add . && git commit -m 'add file a.txt'
echo '123' > a.txt && git add . && git commit -m 'modify a'
echo '1234' > a.txt && git add . && git commit -m 'modify a'
```

创建一个新仓库，并做了一些commit。

```
$ git log --oneline
ef505cc (HEAD -> main) modify a
20bb75b modify a
e2bebb1 add file a.txt
```

这里假如我们在第二次提交 20bb75b 后，发现提交的内容有误，这时经过修改正确后，再按正常的方法来提交，会发现在 `git log` 里会有两条记录，但它们所做的事情都是一样的，就是为了修改 a.txt 文件内容。如果此类误操作比较多的话，在 git log 会显的不够清晰整洁，而如果我们及时的发现并修正成一条操作commit 就显的比较合适了, 这里我们就可以通过  amend commit 来实现。

下面我们模拟修正最后一次提交操作，将上次修改的文件内容内容 1234 改为 12345。

```shell
echo '12345' > a.txt && git add . 
git commit -m 'modify a' --amend
[main ade9cca] modify a
 Date: Tue Jun 25 19:47:20 2024 +0800
 1 file changed, 1 insertion(+), 1 deletion(-)
```

git log 日志

```shell
$ git log --oneline
ade9cca (HEAD -> main) modify a
20bb75b modify a
e2bebb1 add file a.txt
```

可以看到最后一次的 commit `ef505cc` 不见了，出现一个新的 commit `ade9cca`。

可以发现对于 amend commit 只能修正最后一次的提交，如果对于多次提交怎么解决呢？就像这里的 两次commit `ade9cca` 和  `20bb75b` ，如果将它们合并成一个commit 呢？这个就需要用到另一个命令 `squash`

## 注意事项

- **历史记录的重写**：`git commit --amend` 实际上是创建一个新的提交替换之前的提交。因此，如果你已经将提交推送到远程仓库，并且其他人已经基于此提交进行了开发，这样的操作可能会引起问题。需要与团队成员沟通，并使用 `git push --force` 强制推送到远程仓库。
- **合作开发中的使用**：在团队协作中，频繁使用 `--amend` 和 `--force` 推送会导致其他开发者的困惑和冲突。通常建议在本地开发的分支上进行 `amend` 操作，并在确保不会影响其他人的情况下才强制推送。

 

# 合并操作 squash



在 Git 中，`squash` 是用于将多个提交合并为一个提交的操作。这种操作通常在代码重构、合并分支或整理提交历史时非常有用。通过 `squash`，可以使提交历史更简洁、更易读，特别是当你在一个功能分支上有多个小的提交时，可以在合并到主分支前将其压缩为一个逻辑提交。

## 操作步骤

`squash` 通常与 `git rebase -i`（交互式 rebase）一起使用。以下是使用步骤：

1. 开始交互式 rebase

```
git rebase -i HEAD~n
```

其中 `n` 是你想要压缩或合并的提交数。例如，`HEAD~3` 表示最近的三个提交。



2. 编辑 Rebase 计划

 这将打开一个文本编辑器，显示最近的 `n` 个提交。每个提交前都有一个命令，比如：

```
pick e3d1e3c Commit message 1
pick f7e2c5a Commit message 2
pick d8c7f5b Commit message 3
```

将你想要压缩的提交前的 `pick` 修改为 `squash`（或简写为 `s`）。例如：

```
pick e3d1e3c Commit message 1
squash f7e2c5a Commit message 2
squash d8c7f5b Commit message 3
```

这表示将 `f7e2c5a` 和 `d8c7f5b` 提交压缩到 `e3d1e3c` 提交中。

3. 保存并编辑新的 message 退出



大概就这三步，现在我们以上面的实验环境为例

```
$ git log --oneline
ade9cca (HEAD -> main) modify a
20bb75b modify a
e2bebb1 add file a.txt
```

合并最后两次的commit，即 ade9cca 与 20bb75b

```
$ git rebase -i HEAD~2
```

```
  1 pick 20bb75b modify a
  2 pick ade9cca modify a
  3
  4 # Rebase e2bebb1..ade9cca onto e2bebb1 (2 commands)
  5 #
  6 # Commands:
  7 # p, pick <commit> = use commit
  8 # r, reword <commit> = use commit, but edit the commit message
  9 # e, edit <commit> = use commit, but stop for amending
 10 # s, squash <commit> = use commit, but meld into previous commit
 11 # f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
 12 #                    commit's log message, unless -C is used, in which case
 13 #                    keep only this commit's message; -c is same as -C but
 14 #                    opens the editor
 15 # x, exec <command> = run command (the rest of the line) using shell
 16 # b, break = stop here (continue rebase later with 'git rebase --continue')
 17 # d, drop <commit> = remove commit
 18 # l, label <label> = label current HEAD with a name
 19 # t, reset <label> = reset HEAD to a label
 20 # m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
 21 #         create a merge commit using the original merge commit's
 22 #         message (or the oneline, if no original merge commit was
 23 #         specified); use -c <commit> to reword the commit message
 24 # u, update-ref <ref> = track a placeholder for the <ref> to be updated
 25 #                       to this position in the new commits. The <ref> is
 26 #                       updated at the end of the rebase
 27 #
 28 # These lines can be re-ordered; they are executed from top to bottom.
 29 #
 30 # If you remove a line here THAT COMMIT WILL BE LOST.
 31 #
 32 # However, if you remove everything, the rebase will be aborted.
 33 #
```

编辑内容为
```
 1 pick 20bb75b modify a
 2 s ade9cca modify a
```

> 一般最上面第一个 pick，后续被合并是 squash

保存，接着弹出另一个编辑窗口，此窗口用来编辑 git log 中显示的变更停牌

```
  1 # This is a combination of 2 commits.
  2 # This is the 1st commit message:
  3
  4 modify file  # 这行将 modify a 修改为 modify file
  5
  6 # This is the commit message #2:
  7
  8 modify a
  9
 10 # Please enter the commit message for your changes. Lines starting
 11 # with '#' will be ignored, and an empty message aborts the commit.
 12 #
 13 # Date:      Tue Jun 25 19:46:59 2024 +0800
 14 #
 15 # interactive rebase in progress; onto e2bebb1
 16 # Last commands done (2 commands done):
 17 #    pick 20bb75b modify file a.txt
 18 #    squash ade9cca modify a
 19 # No commands remaining.
 20 # You are currently rebasing branch 'main' on 'e2bebb1'.
 21 #
 22 # Changes to be committed:
 23 #       modified:   a.txt
```

保存并退出。

```
$ git rebase -i HEAD~2
[detached HEAD 99b7c37] modify file
 Date: Tue Jun 25 19:46:59 2024 +0800
 1 file changed, 1 insertion(+), 1 deletion(-)
Successfully rebased and updated refs/heads/main.
```

生成一个新的 commit  `99b7c37`

用 git log 确认下

```
$ git log
commit 99b7c37d75d38ff9e842f14c611eb4b3f3af6c5b (HEAD -> main)
Author: cfanbo <haohtml@gmail.com>
Date:   Tue Jun 25 19:46:59 2024 +0800

    modify file

    modify a

commit e2bebb1806994d9bfcc0c6fa796975a999ebc604
Author: cfanbo <haohtml@gmail.com>
Date:   Tue Jun 25 19:45:04 2024 +0800

    add file a.txt
```

可以看来原来修改文件的两个 commit （ `ade9cca` 与 `20bb75b`）消失了，并创建了一个新的commit，即合并成一个新的 commit `99b7c37`。



## 使用场景

1. **开发新功能**：在开发新功能的过程中，可能会有许多小的提交（修复bug、调整代码等）。在将功能分支合并到主分支之前，可以使用 `squash` 将这些提交合并成一个提交，从而保持主分支历史的简洁。
2. **清理历史记录**：在代码提交到主分支之前，通过 `squash` 清理提交历史，可以删除那些不必要的提交，如调试信息、临时更改等。
3. **代码审查**：在进行代码审查时，合并所有相关的提交到一个提交，使审查者能够更清晰地看到所有的更改，而不会被大量的中间提交分散注意力。

## 注意事项

**避免在已推送的分支上进行 squash**：如果你已经将分支推送到远程仓库，并且其他人基于此分支进行了开发，`squash` 会改变提交历史，导致冲突和问题。建议只在本地分支上进行 squash，或在与团队成员沟通后进行。

**强制推送**：如果你需要在远程分支上应用 squash 后的更改，必须使用 `git push --force`，这可能会影响其他开发者。



总之，通过正确使用 `squash`，可以有效和管理和优化 Git 提交历史，使其更具可读性和维护性。

