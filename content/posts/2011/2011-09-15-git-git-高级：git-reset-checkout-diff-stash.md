---
title: '[git] Git 高级：Git Reset, Checkout, Diff, Stash'
author: admin
type: post
date: 2011-09-15T03:06:51+00:00
url: /archives/11464
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - git

---
#### **Git Reset**

有时候，你可能想要撤销 Git 操作。通常，git reset 就是你需要的命令。但是，使用 git reset 可能会非常复杂，取决于你的撤销操作。如果本文没有提到你需要的，查看 [git reset 官方文档][1]。

假设，你正在进行一个项目，想要取消所有的改变，返回到最近一次的提交，命令如下：

```
$ git reset --hard HEAD
```

–hard flag 将指定提交（上例中，HEAD）中的任意内容放入工作目录和缓存区内。因为 HEAD 是最近一次提交，所以我们没有做任何改变。运行 git status，会看到提示没有需要提交的。

还有其它一些 git reset 使用的 flag。假设你想要回滚你的项目到多个提交之前，但是想要保存没有提交的改变，尝试下面的：

```
$ git reset --soft 3ce072c72d948abfa
```

当然，替换为你自己需要的 hash。–soft flag 将保存你的改变在工作目录和缓存区。换句话来说，最新的提交将会是你选择的提交。

还有很多情况，你可以使用 git reset。如果你认为需要这个名利概念的功能，查看[文档][1]。

#### **Git Checkout**

我们之前就提到过 git checkout 命令：用于切换分支。但是还有更多的特性。比如使用 -b flag 同一时间，建立并切换到新分支：

```
$ git checkout -b newBranchName
 Switched to a new branch ‘newBranchName’
```

git checkout 也可以进行回滚撤销工作。git reset 回滚整个提交，而 git checkout 可以回滚单个文件。回滚在缓存区内的文件：

```
$ git checkout -- file.ext
```

如果需要回滚提交中的文件，替换指定提交的指针，如下所示：

```
$ git checkout HEAD file.ext
$ git checkout master~2 file.exe
```

#### **Git Diff**

另外一个很有用的 Git 命令是 git diff。使用这个命令找出两个文件，提交或分支之间的改变。假设你想看自从上一次提交后工作目录都有什么改变，可以执行如下命令：

```
$ git diff
```

如果只是想看一个文件的改变，添加文件名作为参数：

```
$ git diff index.html
```

如果想要比较最新提交和缓存区内的文件，使用 –cached flag。

```
$ git diff --cached
$ git diff --cached index.html
```

比较工作目录和一个提交，使用想要比较的提交的指针，代替上例中的 –cached flag。

```
$ git diff be96dbeab1
$ git diff HEAD^2 index.html
```

也可以使用 git diff 比较分支。将两个分支名作为参数，两个分支名中间为两个 “.”：

```
$ git diff master..otherBranch
```

这样 Git 将会 diff 两个分支的最新提交。如果需要 diff otherBranch 分支和 master 主分支，在分支名间加入三个 “.”。当然，使用你的分支名。

```
$ git diff master...otherBranch
```

#### 更多git diff 的用法也可以参考： [http://blog.haohtml.com/archives/15420](http://blog.haohtml.com/archives/15420)

利用git diff制作补丁Patch用法： [http://blog.haohtml.com/archives/15432](http://blog.haohtml.com/archives/15432)

#### **Git Stash**

有一个有趣的情景：假设你正在编写新特性，但是发现代码中有 bug。而这个bug 与新特性无关，你想要 fix 这个 bug 再继续开发新特性。但是如何处理为新特性写的代码的改变？这里就是 git stash 的用武之地！

git stash 就像是临时提交。当你想要隐藏一些改变，首先将这些改变加入缓存区，再运行下面的：

```
$ git stash
```

在主分区，保存工作目录和索引状态 (index state)：

```
HEAD is now at 3d0b0a4 other commit
```

你会得到一个跟上面很相似的信息，告诉你在特定分支正在进行的工作已经保存。

当你提交过 bug-fix 之后，你可以从 stash 中取出之前的改变。

```
$ git stash apply
```

这样会将 stash 中的内容放回缓存区，同时会得到一个很象 git status 的信息。

你可以进行多次 stash，使用 list 命令查看所有的 stashes：

```
$ git status list
 stash@{0}: On master: started form on contact list
 stash@{1}: On master: README changes
```

在这个例子中，有两个 stashed 项。但是看到了清晰的 stash message 了嘛？如果需要 stash 多次，描述信息会让你清楚你的项目状态。使用 save 命令来添加 stash message：

```
$ git stash save “message here”
```

如果想要将特定 stash 放入缓存区，使用 stash 序号名称，在 list 命令看到的每行开头的信息；如果是将上例中 README 文件放入缓存中，运行如下命令：

```
$ git stash apply stash@{1}
```

在使用Git进行push的时候,有可能遇到无法push内容.解决办法请参考: [http://blog.haohtml.com/archives/12256](http://blog.haohtml.com/archives/12256)

 [1]: http://www.kernel.org/pub/software/scm/git/docs/git-reset.html