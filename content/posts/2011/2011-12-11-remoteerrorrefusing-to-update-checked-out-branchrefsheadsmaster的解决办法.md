---
title: “remote:error:refusing to update checked out branch:refs/heads/master”的解决办法
author: admin
type: post
date: 2011-12-11T10:12:09+00:00
url: /archives/12256
IM_contentdowned:
 - 1
categories:
 - 程序开发
 - 其它
tags:
 - git

---
在使用Git Push代码到数据仓库时，提示如下错误:

[remote rejected] master -> master (branch is currently checked out)

错误原型

> remote: error: refusing to update checked out branch: refs/heads/master
>
> remote: error: By default, updating the current branch in a non-bare repository
>
> remote: error: is denied, because it will make the index and work tree inconsistent
>
> remote: error: with what you pushed, and will require ‘git reset –hard’ to match
>
> remote: error: the work tree to HEAD.
>
> remote: error:
>
> remote: error: You can set ‘receive.denyCurrentBranch’ configuration variable to
>
> remote: error: ‘ignore’ or ‘warn’ in the remote repository to allow pushing into
>
> remote: error: its current branch; however, this is not recommended unless you
>
> remote: error: arranged to update its work tree to match what you pushed in some
>
> remote: error: other way.
>
> remote: error:
>
> remote: error: To squelch this message and still keep the default behaviour, set
>
> remote: error: ‘receive.denyCurrentBranch’ configuration variable to ‘refuse’.
>
> To git@192.168.1.X:/var/git.server/…/web
>
> ! [remote rejected] master -> master (branch is currently checked out)
>
> error: failed to push some refs to ‘git@192.168.1.X:/var/git.server/…/web’

**解决办法:**

这是由于git默认拒绝了push操作，需要进行设置，修改.git/config文件后面添加如下代码：

> [receive]
> denyCurrentBranch = ignore

无法查看push后的git中文件的原因与解决方法

在初始化远程仓库时最好使用

> git –bare init

而不要使用：**git init**

git init 和git –bare init 的具体区别:

=================================================

如果使用了git init初始化，则远程仓库的目录下，也包含work tree，当本地仓库向远程仓库push时, 如果远程仓库正在push的分支上（如果当时不在push的分支，就没有问题）, 那么push后的结果不会反应在work tree上,  也即在远程仓库的目录下对应的文件还是之前的内容。

**解决方法：**

必须得使用命令 git reset –hard 才能看到push后的内容.

研究了很久不得其解，然后找到一条命令凑合着能用了：

登录到远程的那个文件夹，使用

>

> git config –bool core.bare true
>

就搞定了。

贴一段参考文章：

> Create a bare GIT repository
>
> A small rant: git is unable to create a normal bare repository by itself. Stupid git indeed.
>
> To be precise, **it is not possible to clone empty repositories**. So an empty repository is a useless repository. Indeed, you normally create an empty repository and immediately fill it:
>
> git init git add .
>
> However, git add is not possible when you create a bare repository:
>
> git –bare init git add .
>
> gives an error “fatal: This operation must be run in a work tree”.
>
> You can’t check it out either:
>
> Initialized empty Git repository in /home/user/myrepos/.git/ fatal:  not found: did you run git update-server-info on the server? git –bare init git update-server-info # this creates the info/refs file chown -R : . # make sure others can update the repository
>
> The solution is to create _another repository **elsewhere**_, add a file in that repository and, push it to the bare repository.
>
> mkdir temp; cd temp git init touch .gitignore git add .gitignore git commit -m “Initial commit” git push  master cd ..; rm -rf temp