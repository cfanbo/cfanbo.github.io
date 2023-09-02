---
title: Git中的git reset的三种参数的区别
author: admin
type: post
date: 2018-09-24T03:18:25+00:00
url: /archives/18265
categories:
 - 程序开发
tags:
 - git

---
我们平时在使用git的时候，经常会遇到需要撤销上次操作的需求，这时候需要用到git reset的这个命令，他的使用就是 “git-reset – Reset current HEAD to the specified state”, 注意这里主要操作的就是这个 **HEAD**。

为了方便我们先了解一下 Git 的工作流程

[![](https://blog.haohtml.com/wp-content/uploads/2018/09/git_flow.jpg)][1]

相信大家对这个图已经很熟悉了，其中index也叫stage暂存区或者暂存索引区。git reset 共有三个互斥参数分别为”–soft”、”–mixed(默认参数)” 和 “–hard”，每种参数表示一种恢复模式，下面我们将分别看一下这git reset 三个参数的用法区别。

**前提条件**
我们仓库中的Git 提交顺序为 “A(a.txt) -> B(b.txt) -> C(c.txt)“，当前分支为master。
当前 HEAD 指向C，即 a47072e9f97eac4ac02c0abac82b26a9719663fc (HEAD -> master)，我们以恢复到B(aad0c91e7b1d3577)点为准。

```
test1 git:(master) git log
commit a47072e9f97eac4ac02c0abac82b26a9719663fc (HEAD -> master)
Author: 孙兴房 <1293812389@qq.com>
Date: Mon Sep 24 10:37:14 2018 +0800

add c.txt

commit aad0c91e7b1d357729f65dc0bbeb6c9c9dd53844
Author: 孙兴房 <1293812389@qq.com>
Date: Mon Sep 24 10:22:12 2018 +0800

add b.txt

commit 68315608ef8d0cff5d229c2ee5010e59a1475cfe
Author: 孙兴房 <1293812389@qq.com>
Date: Mon Sep 24 10:21:59 2018 +0800

add a.txt

```

**–soft 模式
** 执行 git reset –soft aad0c9,恢复到B。然后再执行 git status 看下

[![](https://blog.haohtml.com/wp-content/uploads/2018/09/git_reset_soft.jpg)][2]
可以看到当时在C添加的c.txt文件还存在。但提示已经在”stage“暂存索引区了，即上图中的”Index”这一步。我们用 git log 命令确认这一点。

[![](https://blog.haohtml.com/wp-content/uploads/2018/09/git_log_1.jpg)][3]

同时如果想将此文件从索引区中取消的话，只需要执行 git reset HEAD <文件> 命令就可以了。

此时我们可以对文件进行一些修改，然后再重新commit。

[![](https://blog.haohtml.com/wp-content/uploads/2018/09/git_log_2.jpg)][4]

–soft模式总结：
1. 当执行 git reset –soft B时，当前分支(master)和HEAD同时指向B。
2. 原来在C提交的文件仍然存在，文件状态会**处于stage暂存区(index)**，当前状态为**index** ，后续命令为git commit 。
3. 原C提交的git日志记录(a47072e9f97eac4ac02c0abac82b26a9719663fc)消失。

我们此时对原来在C提交的文件进行内容修改（修改后记得需要重新执行 git add 文件到stage一次），再次commit即可。这时会出现了一个新的git 日志记录(88277bd1433b3ef0bc213e524e8943e114b50eda)。

**–mixed(默认)  模式**

执行 git reset –mixed aad0c9,恢复到B。然后再执行 git status 看下

[![](https://blog.haohtml.com/wp-content/uploads/2018/09/git_status_2.jpg)][5]可以看到当时在C添加的c.txt文件还存在(这一点和–soft 一样的)。但提示文件**尚未跟踪**，也就是说还没有对这个文件执行git add操作添加到stage暂存区中。此时的状态和新创建的文件一模一样的。为了验证这一点我们添加一个文件d.txt看一下。

[![](https://blog.haohtml.com/wp-content/uploads/2018/09/git_status_3.jpg)][6]
这里为了后续演示方便，还保持原来的样子，把新添加的d.txt删除掉后，再执行git add 和 git commit。
[![](https://blog.haohtml.com/wp-content/uploads/2018/09/git_commit_1.jpg)][7]我们用git log命令看一下

[![](https://blog.haohtml.com/wp-content/uploads/2018/09/git_log_4.jpg)][8]

此时新出现的git日志记录为“e6147eb0322042090e99ab98ff37a2e22cf9be19”。

–mixed模式总结：
1. 当使用 git reset –mixed 命令时，工作区不变，会重置索引区。当前分支(master)和HEAD都指向B，同soft一样。
2. 原来C添加的文件仍然存在，但**未处于stage暂存区(index)**，当前状态为 **workspace**，后续命令为git add 和 git commit。不与于soft 模式。
3. 原C提交的git日志记录消失。

**–hard 模式**

执行 git reset –hard aad0c9,恢复到B。然后再执行 git status 看下

[![](https://blog.haohtml.com/wp-content/uploads/2018/09/git_reset_hard.jpg)][9]

发现这次和上面的两种结果完全不一样，在C添加的文件直接消失不存在了。

[![](https://blog.haohtml.com/wp-content/uploads/2018/09/git_log_5.jpg)][10]

此时当前分支(master)和HEAD全部指向B，原来在C添加的文件也被删除了，所以大家执行此命令时一定要小心，不然会把恢复日志记录以后添加的内容全部删除的。

–hard模式总结：

1. 当使用 git reset –hard 命令时，工作区发生变化，会重置索引区。当前分支(master)和HEAD都指向B，同soft一样。
2. 原来C添加的文件仍被删除（不同于以上两种模式)
3. 原C提交的git日志记录消失。

上面我们分析了三种模式的区别，大家使用中一定要注意，特别是 hard 硬模式。

总结
[![](https://blog.haohtml.com/wp-content/uploads/2018/09/git_reset.jpg)][11]

**技巧：**
另外大家嫌不好记得话，只要记住执行 soft 模式后（回到stage暂存区状态），如果再次将原来的文件添加到仓库的话，需要git commit **一个命令**就可以了。而 mixed 模式（回到workspace状态）需要执行git add 和 git commit **两个命令**。对于 hard 模式来说是一种硬恢复，导致部分数据丢失，尽量不要用(可以使用git reflog 来恢复)。

有兴趣的可以看一下git checkout 、git revert 和 git reset 三者的区别( [https://blog.csdn.net/foolsong/article/details/75203005](https://blog.csdn.net/foolsong/article/details/75203005))

 [1]: https://blog.haohtml.com/wp-content/uploads/2018/09/git_flow.jpg
 [2]: https://blog.haohtml.com/wp-content/uploads/2018/09/git_reset_soft.jpg
 [3]: https://blog.haohtml.com/wp-content/uploads/2018/09/git_log_1.jpg
 [4]: https://blog.haohtml.com/wp-content/uploads/2018/09/git_log_2.jpg
 [5]: https://blog.haohtml.com/wp-content/uploads/2018/09/git_status_2.jpg
 [6]: https://blog.haohtml.com/wp-content/uploads/2018/09/git_status_3.jpg
 [7]: https://blog.haohtml.com/wp-content/uploads/2018/09/git_commit_1.jpg
 [8]: https://blog.haohtml.com/wp-content/uploads/2018/09/git_log_4.jpg
 [9]: https://blog.haohtml.com/wp-content/uploads/2018/09/git_reset_hard.jpg
 [10]: https://blog.haohtml.com/wp-content/uploads/2018/09/git_log_5.jpg
 [11]: https://blog.haohtml.com/wp-content/uploads/2018/09/git_reset.jpg