---
title: redmine与git组合配置
author: admin
type: post
date: 2013-11-22T02:57:49+00:00
url: /archives/14771
categories:
 - 程序开发
tags:
 - git
 - redmine

---
参考：[http://www.redmine.org/projects/redmine/wiki/HowTo\_Easily\_integrate\_a\_(SSH\_secured)\_GIT\_repository\_into_redmine][1]

在redmine 的”配置”选项里，填写git仓库位置的时候，一定要在写完整的.git路径，如 **/data/www/redmine/repos/agent/.git** ，否则redmine会无法发现git仓库.

[![redmine-git-repo](https://blogstatic.haohtml.com//uploads/2023/09/redmine-git-repo.png)][2]

相关文章：

[http://blog.csdn.net/benkaoya/article/details/8762935](http://blog.csdn.net/benkaoya/article/details/8762935)

[1]: http://www.redmine.org/projects/redmine/wiki/HowTo_Easily_integrate_a_(SSH_secured)_GIT_repository_into_redmine
[2]: http://http://blog.haohtml.com/wp-content/uploads/2013/11/redmine-git-repo.png
