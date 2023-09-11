---
title: Git中gitignore失效的问题
author: admin
type: post
date: 2012-08-14T06:16:01+00:00
url: /archives/13302
categories:
 - 程序开发
tags:
 - git

---
使用git来管理代码，但发现仓库中加入了.gitignore文件，但并不能解除对.gitignore文件中指定的路径及文件进行忽略。是因为加入.gitignore的之前已经进行过提交，提交中含有要忽略的文件，而这个时候.gitignore 对这些文件是失效的，为了解决这个问题，需要先删除这些中间文件，然后进行一次提交就可以解决这些问题了。

在本地仓库将.gitgnore文件里指定的相关文件及路径全部删除，再commit到本机一下。然后执行push到git Server就可以了。这样就可以将git sever上的那些临时文件删除掉。以后再使用的话，产生的文件就不会在提交到git server上去了。