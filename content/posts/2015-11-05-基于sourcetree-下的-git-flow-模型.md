---
title: 基于SourceTree 下的 Git Flow 模型
author: admin
type: post
date: 2015-11-05T10:42:46+00:00
url: /archives/16039
categories:
 - 程序开发
tags:
 - git
 - gitflow
 - SourceTree

---
gitflow 开发流程参考： [http://blog.haohtml.com/archives/15317](http://blog.haohtml.com/archives/15317)

[![git_flow](http://blog.haohtml.com/wp-content/uploads/2014/09/git_flow.png)][1]
**基于SourceTree 下的 Git Flow 模型**

1. sourceTree  是一个开源的git 图形管理工具，可下载mac版本,windows版本

2. Git Flow 是一套使用Git进行源代码管理时的一套行为规范和简化部分Git操作的工具。

**基本的操作流程**

1. 先用sourceTree 创建本地git 项目，xxxProject,

2. 在项目里面先提交一次 commit 一下，默认提交在了 master分支；

3. 然后在 sourceTree工具 右上角，点击 GitFlow,开启git Flow 规范模型的开发

[![git-flow_1](http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_1.png)][2]

如上图，在开启gitFlow 之后；

生产环境分支使用：master

开发分支使用：develop

当需要新增加功能，发布版本时，创建补丁修复bug时，分别有对应的 feature,release,hotfix前缀这样的分支

这样在项目的开发过程之中，管理项目分支就变得非常的规范了；

4：开启之后，我们的项目就回到了develop 分支，以后所的开发都在这个分支上进行；当开发完成一些模块时，就可以回去 master分支 合并

5. 使用 gitFlow 添加新功能 ,点击 sourceTree 的右上角 Git Flow按钮，会出现 菜单，选择创建新功能

[![git-flow_2](http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_2.png)][3]

输出新功能名称，默认会在 新功能 分支上开发新功能；

新功能 开发完成之后，再次点击 git flow 按钮，会出现 完成新功能，按钮点击，完成新功能，，会把当前新功能合分支 合并到 develop分支，并删除新功能分支

[![git-flow_3](http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_3.jpg)][4]



6：使用Git Flow 发布新版本，同样点击 git Flow 按钮，菜单选择 创建新发布版本 ，

[![git-flow_4](http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_4.png)][5]

在发布版本分支上，完成项目发布配置之后，提交，再点击 git flow 按钮，会弹出 完成发布版本 按钮，点击，

[![git-flow_5](http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_5.png)][6]

确认之后，会发现 发布版本的分支，会合并到 develop分支 和 master 分支，表示生产上发布了一个版本

7：使用git flow 新建补丁，修复bug

比如上面发布的一个版本在生产用的时候，出现了一个 bug,这时，点击 git flow 菜单，选择 建立新的修复补丁

[![git-flow_6](http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_6.png)][7]

这时，bug修复分支，是基于 master的，在修复bug后，再次点击 git flow 弹出，完成 补丁修复

[![git-flow_7](http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_7.png)][8]

确定之后，会发现，新修复的bug分支，会合并到 master分支和develop分支

8：最后我们再来看看，经过上面的 创建项目–开启gitflow—添加新功能—发布新版本—修复bug 等流程之后，当前的 git提交状态吧

[![git-flow_8](http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_8.png)][9]

git 强大的分支管理功能，再加上 git flow 模型，，项目的代码管理开发，如此的清晰明了啊

 [1]: http://blog.haohtml.com/wp-content/uploads/2014/09/git_flow.png
 [2]: http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_1.png
 [3]: http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_2.png
 [4]: http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_3.jpg
 [5]: http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_4.png
 [6]: http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_5.png
 [7]: http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_6.png
 [8]: http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_7.png
 [9]: http://blog.haohtml.com/wp-content/uploads/2015/11/git-flow_8.png