---
title: 使用git-flow来帮助管理git代码
author: admin
type: post
date: 2014-09-14T09:16:47+00:00
url: /archives/15324
categories:
 - 程序开发
tags:
 - gitflow

---
对git不熟悉的我，经常把git提交搞得很乱，导致在master上有许多无用的commit，最终决定好好地看一下git的使用教程，却不小心发现了还有一个git-flow的工具可以帮助我管理好git项目的代码。

git-flow在ubuntu上使用比较简单。首先安装，可以通过apt-get来获取。命令如下：

> sudo apt-get install git-flow

如果是在windows下，可以参考这篇文章进行安装： [http://my.eoe.cn/sunxun/archive/158.html](http://my.eoe.cn/sunxun/archive/158.html)

如果你的git已经装好，则方便多了，下载下面两个地址的文件，并解压出getopt.exe和libintl3.dll放到git的安装目录的bin目录下。 [http://sourceforge.net/projects/gnuwin32/files/util-linux/2.14.1/util-linux-ng-2.14.1-bin.zip/download](http://sourceforge.net/projects/gnuwin32/files/util-linux/2.14.1/util-linux-ng-2.14.1-bin.zip/download) [http://sourceforge.net/projects/gnuwin32/files/util-linux/2.14.1/util-linux-ng-2.14.1-dep.zip/download](http://sourceforge.net/projects/gnuwin32/files/util-linux/2.14.1/util-linux-ng-2.14.1-dep.zip/download)

然后检出github上gitflow项目，如下命令：

1. git clone –recursive git://github.com/nvie/gitflow.git


进入并执行里面的contribmsysgit-install.cmd，提示复制成功，就可以了。

接下来是初始化项目。我在我原来的git项目上执行以下命令来进行初始化：

1. git flow init


它会创建或转换一个新的版本分支结构，当然在初始化的过程中，会问到以下这边问题，我都选择了默认：

01. Which branch should be used for bringing forth production releases?

02.    – master

03. Branch name for production releases: [master]

04. Branch name for “next release” development: [develop]

06. How to name your supporting branch prefixes?

07. Feature branches? [feature/]

08. Release branches? [release/]

09. Hotfix branches? [hotfix/]

10. Support branches? [support/]

11. Version tag prefix? []


完成之后，通过git branch 命令，可以看到它为我们新建好了一个develop的分支。

接下来我将继续使用，这篇笔记再慢慢补充。

修复一个bug。

1. git flow hotfix start 3


它会创建一个基于master的分支hotfix/3，并切换到当前分支。

当修复完成后，可以执行以下命令：

1. git flow notfix finish 3


增加一个功能特性

1. git flow feature start demo


它会创建一个分支feature/demo，并切换到该分支。

当功能完成：

1. git flow feature finish demo


它会有feature/demo分支合并到develop分支，然后切换回develop分支，并删除feature/demo分支。

功能完成，要合并到主分支，这时可以执行

1. git flow release start v0.7.0


它会创建一个release/v0.7.0分支，并切换到该分支。

然后在这里进行测试。如果测试没问题，则执行以下命令：

1. git flow release finish v0.7.0


它会将release/v0.7.0分支的内容合并到master分支和develop分支，并且打上tag v0.7.0，然后删除release/v0.7.0分支。

**相关教程：**

[gitflow 开发流程](http://blog.haohtml.com/archives/15317)