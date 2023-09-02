---
title: Git-svn命令对比表,svn用户必看
author: admin
type: post
date: 2016-02-23T01:55:06+00:00
url: /archives/16690
categories:
 - 程序开发
tags:
 - git
 - svn

---
提供给从svn转git的开发人员参考

Git与Subversion的命令对比表

**操作****Git****Subversion**复制数据仓库git clonesvn checkout提交git commitsvn commit
 查看提交的详细记录

 git show

 svn cat

 确认状态

 git status

 svn status

 确认差异

 git diff

 svn diff

 确认记录

 git log

 svn log
 添加git addsvn add
 移动

 git mv

 svn mv

 删除

 git rm

 svn rm

 取消修改

 git checkout / git reset

 svn revert (※1)

 创建分支

 git branch

 svn copy (※2)

 切换分支

 git checkout

 svn switch

 合并

 git merge

 svn merge

 创建标签

 git tag

 svn copy (※2)
 从服务端更新本地git pull / git fetchsvn update推送到远端git pushsvn commit (※3)
 忽略档案目录.gitignore.svnignore

※1. SVN的revert是用来取消修改，但Git的revert是用来消除提交。所以即使是同样的命令，在SVN和Git里的含义是不同的。

※2. SVN的分支与标签在构造上是相同的，但在Git其构造明显是不一样的。

※3. SVN没有本地数据库/远程数据库的概念，所以提交会马上反映到远程里。但Git的本地数据库和远程数据库的反映方法是不一样的。