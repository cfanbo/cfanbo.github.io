---
title: gitflow 开发流程
author: admin
type: post
date: 2014-09-14T09:07:27+00:00
url: /archives/15317
categories:
 - 程序开发
tags:
 - git
 - gitflow

---
Update: 2011/3/19 受邀有场分享 [Git介绍,使用与开发流程 at Facebook 软体开发团队工具心得分享](http://ihower.tw/blog/archives/5391)

目前有专业提供gitflow开发流程的开发工具 [SourceTree](https://www.sourcetreeapp.com/)，推荐大家可以用用，mac和windows客户端都有的。

除了gitflow还有github flow 和gitlab flow。参考： [http://www.ruanyifeng.com/blog/2015/12/git-workflow.html](http://www.ruanyifeng.com/blog/2015/12/git-workflow.html)

大家都知道 Git 开 branch 很方便，非常鼓励 topic branch，但有没有一套模型流程告诉我们应该怎麽管理 branch 呢? 有人便整理出一套最佳实践惯例 [A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)， [我们团队](http://optimisdev.com/) 就採用了这套流程。简单来说，他将 branch 分成两个主要分支，三种支援性分支：

[![git_flow](http://blog.haohtml.com/wp-content/uploads/2014/09/git_flow.png)](http://blog.haohtml.com/wp-content/uploads/2014/09/git_flow.png)

- 主要分支
  - master: 永远处在 production-ready 状态

  - develop: 最新的下次发佈开发状态
- 支援性分支
  - Feature branches: 开发新功能都从 develop 分支出来，完成后 merge 回 develop

  - Release branches: 准备要 release 的版本，只修 bugs。从 develop 分支出来，完成后 merge 回 master 和 develop

  - Hotfix branches: 等不及 release 版本就必须马上修 master 赶上线的情况。会从 master 分支出来，完成后 merge 回 master 和 develop

作者还提供了 [git-flow](https://github.com/nvie/gitflow) 指令工具帮助我们很容易的实践，用法如下:

首先是初始化动作：

```

    git flow init

```

初始化动作会问你一些问题，大抵是命名惯例：

```

No branches exist yet. Base branches must be created now.
Branch name for production releases: [master]
Branch name for "next release" development: [develop]
How to name your supporting branch prefixes?
Feature branches? [feature/]
Release branches? [release/]
Hotfix branches? [hotfix/]
Support branches? [support/]
Version tag prefix? []

```

设定完之后，预设的 branch 就变成 develop 了。有任何开发，一律都先开 branch：

```

git flow feature start some_awesome_feature
(以此类推 git flow release 和 git flow hotfix)

```

完成之后输入

```

git flow feature finish some_awesome_feature

```

就会合併回 develop 并帮你删除这个 (local) branch。

### 关于 remote branch

这个 git-flow 工具并没有帮我们处理 remote branch，所以如果你的 branch 要 push 出去分享给别人，就要自己打 git 指令啦 同事留言说有支援啦：

push 一个 feature branch 到远端：

```

git flow feature publish some_awesome_feature
或 git push origin feature/some_awesome_feature

```

追踪一个远端的 branch：

```

git flow feature track some_awesome_feature
或 git checkout -b feature/some_awesome_feature -t origin/feature/some_awesome_feature

```

删除远端的 branch：

```

git push origin :feature/some_awesome_feature

```

我们还碰到一个问题是输入 git flow feature finish 时出现以下错误：

```

warning: not deleting branch 'feature/some_awesome_feature' that is not yet merged to
         'refs/remotes/origin/feature/some_awesome_feature', even though it is merged to HEAD.
error: The branch 'feature/some_awesome_feature' is not fully merged.
If you are sure you want to delete it, run 'git branch -D feature/some_awesome_feature'.

```

原因是这个 feature branch 一开始是从远端 checkout 出来的，以及这个 feature branch 有 commit 没有 push 回去 ，所以 git flow 不敢帮你删除 local branch，这时候其实 merge 动作已经完成了，所以你可以手动输入 git branch -D feature/some_awesome_feature 强制删除 local branch 即可。(小结论：git-flow 只是个辅助工具，了解 git 还是必要的)

### 关于 feature branch 的合併

如果是开发时间比较久的 feature branch，很可能会因为 1. 不定时的 merge develop 与新版同步 2. 实验性质的修改 3. 需求的变更 等等因素，而让这个 feature branch 的 commit 记录变成葬葬的，这时候我们会用以下的方式来做 merge 动作：

1. 先对 feature branch 做 `git rebase develop`。会很苦，但是弄完会很有成就感，整个 branch commit history 会变成很乾淨。请学 interactive mode，可以让你拿掉一些 commit、合併或修改，你也可以 rebase 多次直到满意为止。

2. 在从 develop bracnh 做 `git merge feature/some_awesome_feature –no-ff`，–no-ff 的意思是会强制留一个 merge commit log 记录，这可以让 commit tree 看清楚发生了 merge 动作。(因为我们刚做了 rebase，而 git 预设的合併模式是 fast-forward，所以如果不加 –no-ff 是不会有 merge commit 的) 这个 merge commit 的另一个额外方便之处是，如果想要 reset/revert 整个 branch 只要 reset/revert 这个 commit 就可以了。

3. 如果此 feature branch 有 remote branch，要先砍掉 `git push origin :feature/some_awesome_feature` 再 `git push origin develop` (这是因为 rebase 一个已经 push 出去的 repository，然后又把修改的 history push 出去，会造成超级大灾难啊~)

先 rebase 再 merge –no-ff 这样做的好处到底是什麽? 看图体会一下吧：

[![git-branch1](http://blog.haohtml.com/wp-content/uploads/2014/09/git-branch1.jpg)](http://blog.haohtml.com/wp-content/uploads/2014/09/git-branch1.jpg)

每一次的 merge 就代表了一个 feature 完成，也可以很清楚看到这个 feature branch 底下包含哪些 commit。

对了，如果有用 Github 的话，请记得务必用一用它的 [pull request](https://github.com/blog/785-pull-request-diff-comments) 功能，我们会在 branch 完成后发一个 pull request，好让大家可以对一整个 branch 做 code review 留言。

注：什麽是 rebase 可以参考旧作: [Git 版本控制系统(3) 还没 push 前可以做的事](http://ihower.tw/blog/archives/2622)

转自： [http://ihower.tw/blog/archives/5140](http://ihower.tw/blog/archives/5140)

**相关教程：**

[使用git-flow来帮助管理git代码](http://blog.haohtml.com/archives/15324)