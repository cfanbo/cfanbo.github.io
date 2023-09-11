---
title: git commit 与 git commit -a 的区别
author: admin
type: post
date: 2015-06-17T10:14:06+00:00
url: /archives/15770
categories:
 - 程序开发
tags:
 - git

---
软件版本：
操作系统：ubuntu10.04
内核版本：Linux version 2.6.32-36-generic
git 版本：git version 1.7.0.4

**目录：**

1. 文件状态
2. 提交
2.1 git commit 与 git commit -a
2.2 添加提交信息
3. 修改/取消
4. 参考资料

**1. 文件状态**

一般仓库中的文件可能存在于这三种状态：

1）Untracked files → 文件未被跟踪；
2）Changes to be committed → 文件已暂存，这是下次提交的内容；
3)  Changes bu not updated → 文件被修改，但并没有添加到暂存区。如果 commit 时没有带 -a 选项，这个状态下的文件不会被提交。

```
$git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#    new file:   file2
#
# Changed but not updated:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#    modified:   file
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#    file3
```

**2. 提交**

git 提交的命令为：git commit 。

2.1 git commit 与 git commit -a 

git commit 提交的是暂存区里面的内容，也就是 Changes to be committed 中的文件。

```
$git commit
[master 5b61c29] run git commit
 0 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 file2

$git status
# On branch master
# Changed but not updated:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#    modified:   file
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#    file3
```

git commit -a 除了将暂存区里的文件提交外，还提交 Changes bu not updated 中的文件。

```
$git commit -a
[master bd77524] run git commit -a
 1 files changed, 2 insertions(+), 0 deletions(-)
 create mode 100644 file2

$git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#    file3
```

2.2 添加提交信息

如果直接运行 git commit (-a) 则会默认使用 vi 添加描述。也可以使用 `git config --global core.editor` 命令更改为你喜欢的编辑器。还有一个方法就是使用 -m 选项直接添加提交信息。

```
$git commit -a -m "commit info"
```

**3. 修改/取消**

有时候我们会发现有几个文件漏了提交或者想修改一下提交信息，又或者忘记使用 -a 选项导致一些文件没有被提交，我们希望对上一次提交进行修改，或者说取消上一次提交，这时候我们需要使用 –amend 选项。

```
$git commit --amend
```

可以对上一次提交进行修改，比如我们发现漏了 file3 没有提交，我们可以运行一下操作：

```
$git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#    file3
nothing added to commit but untracked files present (use "git add" to track)

$git add file3
$git commit --amend
[master 671f5cc] commit --amend, add file3
 1 files changed, 2 insertions(+), 0 deletions(-)
 create mode 100644 file2
 create mode 100644 file3

$git status
# On branch master
nothing to commit (working directory clean)
```

又或者我们发现在提交时忘记使用 -a 选项，导致 Changes bu not updated 中的内容没有被提交，我们可以使用：

```
$git commit --amend -a
```

**4. 参考资料**

[1] 《pro git》