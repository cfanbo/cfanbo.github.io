---
title: git stash和git stash pop的用法
author: admin
type: post
date: 2013-12-23T01:41:47+00:00
url: /archives/14865
categories:
 - 程序开发
tags:
 - git

---

推荐阅读： [http://www.cppblog.com/deercoder/archive/2011/11/13/160007.html](http://www.cppblog.com/deercoder/archive/2011/11/13/160007.html)

原文： [http://gitbook.liuhui998.com/4_5.html](http://gitbook.liuhui998.com/4_5.html)

**一、基本操作**

当你正在做一项复杂的工作时, 发现了一个和当前工作不相关但是又很讨厌的bug. 你这时想先修复bug再做手头的工作, 那么就可以用 git stash 来保存当前的工作状态, 等你修复完bug后,执行’反储藏'(unstash)操作就可以回到之前的工作里.

```
$ git stash save "work in progress for foo feature"
```

上面这条命令会保存你的本地修改到储藏(stash)中, 然后将你的工作目录和索引里的内容全部重置, 回到你当前所在分支的上次提交时的状态.

好了, 你现在就可以开始你的修复工作了.

… edit and test …

```
$ git commit -a -m "blorpl: typofix"
```

当你修复完bug后, 你可以用git stash apply来回复到以前的工作状态.

```
$ git stash apply
```

**二、储藏队列**

你也可多次使用’git stash’命令,　每执行一次就会把针对当前修改的‘储藏’(stash)添加到储藏队列中.

用’git stash list’命令可以查看你保存的’储藏'(stashes):

$>git stash list

stash@{0}: WIP on book: 51bea1d… fixed images

stash@{1}: WIP on master: 9705ae6… changed the browse code to the official repo

可以用类似’git stash apply stash@{1}’的命令来使用在队列中的任意一个’储藏'(stashes).

‘git stash clear‘则是用来清空这个队列.

=========================================

$git stash 可用来暂存当前正在进行的工作， 比如想pull 最新代码， 又不想加新commit， 或者另外一种情况，为了fix 一个紧急的bug,  先stash, 使返回到自己上一个commit, 改完bug之后再stash pop, 继续原来的工作。

基础命令：

```
$git stash
do some work.........
$git stash pop

```

**进阶：**

当你多次使用’git stash’命令后，你的栈里将充满了未提交的代码，这时候你会对将哪个版本应用回来有些困惑，’git stash list’命令可以将当前的Git栈信息打印出来，你只需要将找到对应的版本号，例如使用’git stash apply stash@{1}’就可以将你指定版本号为stash@{1}的工作取出来，当你将所有的栈都应用回来的时候，可以使用’git stash clear’来将栈清空。

参考：http://www.tuicool.com/articles/rUBNBvI