---
title: SSH+Git Server on Windows – 在Windows下搭建Git服务器(教程)
author: admin
type: post
date: 2012-07-16T04:34:58+00:00
url: /archives/13162
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - git
 - ssh

---
推荐软件： [Windows 的 Git 服务器GitStack](http://www.oschina.net/news/60555/gitstack-2-3-7)

会看英文



**软件需求:**
1.windowXP， win7 都测试通过
2.Copssh\_3.1.4\_Installer.exe
3.Git-1.7.3.1-preview20101002.exe

**搭建git服务器步骤:**
1.安装copssh
1.1  我选择安装路径c:\ICW,其他选项都选默认.
1.2 设置环境变量,系统的Path中添加C:\ICW\bin


1.3 右键 我的电脑,选择 管理,打开 系统工具->本地用户和组->用户,  在用户窗口点击右键,选择 新用户,用户   名输入git,密码输入git.
1.4.选择git用户,右键 选属性, 点击 隶属于->添加,使git用户被添加到administrator组,并拥有administrator权限.
1.5 选择 开始->所有程序->copssh->0.1 activate a user,在user name下拉列表中选择刚刚新建的git用户,点击next,输入 Type a passhrase,并记住输入的Type a passhrase,点击 activate.

**2.安装git**
2.1  我选择安装路径c:\git,其他选项都选默认.
2.2 设置环境变量,系统的Path中添加C:\git\bin

**3.检验设置**
3.1 打开一个cmd,输入 ssh git@127.0.0.1,按照提示输入密码,(我上面设置的是git),出现远程登录,git用户ssh登录成功
3.2 登录成功后,可以使用ls,cd,rm,chmod等命令,但是不能使用git命令,也就是不能使用ssh协议管理git仓库.

**4.设置使用ssh协议 管理git 仓库**
4.1开始-> CopSSH > Start a unix bash shell.
4.2 cd /Bin
4.3 创建 4个符号连接指向 `git.exe`, `git-receive-pack.exe`, `git-upload-archive.exe`, `git-upload-pack.exe`:

```
$ ln -s /cygdrive/c/git/bin/git.exe git.exe
$ ln -s /cygdrive/c/git/libexec/git-core/git-receive-pack.exe git-receive-pack.exe
$ ln -s /cygdrive/c/git/libexec/git-core/git-upload-archive.exe git-upload-archive.exe
$ ln -s /cygdrive/c/git/libexec/git-core/git-upload-pack.exe git-upload-pack.exe
```

```
4.4 退出git账号,打开一个cmd,输入ssh git@127.0.0.1,重新登录,登录成功后,输入git 命令,会出现git命令的使用帮助.
     或者直接打开一个cmd,输入git,同样会出现git命令的使用帮助,表明可以正常使用git命令了.
 4.5 启动一个cmd,进入到C:\ICW\var目录下,依次执行
     mkdir test
     cd test
     git --bare init
     touch a b
     git add .

     git config --global user.name "jackylee"  //用于添加提交用户信息
     git config --global user.email "orange.jackylee@gmail.com"//用于添加用户提交信息

     git commit -m  "first commit"

     使用git show 可以看到提交的信息和用户信息
```

```
在初始化远程仓库时最好使用 git -–bare init   而不要使用：git init
```

4.5 启动一个cmd,我准备要把服务器管理的test仓库 拷贝到e:\,   所以输入 cd e:\ ,执行拷贝
**git clone git@127.0.0.1:../../var/test  test**     (路径是相对路径,相对于git账号登录后的c:\ICW\home\git目录)

4.6拷贝完成.cmd输出.
Cloning into test…
git@127.0.0.1’s password:
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (3/3), done.

4.7 创建git账号信息,用于提交时区分哪个账号提交了什么内容.
登录git账号,输入pwd,输出为/home/git,
输入
**touch .gitconfig**
echo “[user]” > .gitconfig
echo “name=jackylee” >> .gitconfig
echo “email=orange.jackylee@gmail.com” >> .gitconfig

其他账号创建与创建git账号相同

**5. 让git 管理其他路径下文件.比如要让git管理e:\project目录**
启动cmd,进入e:\project
依次输入
git init
git add .
git commit -m “first commit”

启动一个cmd,进入C:\ICW\var
依次输入
ln -s e:\project  project

现在已经可以使用git clone来管理e:\project目录了
输入:git clone git@127.0.0.1:../../var/project  project_backup

以上一个完整的xp  git服务器已经设置完成.

问题备忘：
– if cannot push to remote repo
change the bare = true  in \.git\config
if is bare option, cannot checkout

– ssh git@v:..\..\test test
related location
3. c:\git;c:\icw has settings
==============================================================
如果在使用Git Push代码到数据仓库时，提示如下错误:

> [remote rejected] master -> master (branch is currently checked out)
> remote: error: refusing to update checked out branch: refs/heads/master
> remote: error: By default, updating the current branch in a non-bare repository
> remote: error: is denied, because it will make the index and work tree inconsistent
> remote: error: with what you pushed, and will require ‘git reset –hard’ to match
> remote: error: the work tree to HEAD.
> remote: error:
> remote: error: You can set ‘receive.denyCurrentBranch’ configuration variable to
> remote: error: ‘ignore’ or ‘warn’ in the remote repository to allow pushing into
> remote: error: its current branch; however, this is not recommended unless you
> remote: error: arranged to update its work tree to match what you pushed in some
> remote: error: other way.
> remote: error:
> remote: error: To squelch this message and still keep the default behaviour, set
> remote: error: ‘receive.denyCurrentBranch’ configuration variable to ‘refuse’.
> To git@192.168.1.X:/var/git.server/…/web
> ! [remote rejected] master -> master (branch is currently checked out)
> error: failed to push some refs to ‘git@192.168.1.X:/var/git.server/…/web’

**这是由于git****默认拒绝了push****操作，需要进行设置，修改.git/config****添加如下代码：**

[receive]
denyCurrentBranch = ignore

（当然如果是git –bare init这样建立的仓库，在server端不含有.git目录，当然就不需要的了，也不会遇到上面的错误）。

**在初始化远程仓库时最好使用 git –-bare init   ****而不要使用：git init**

如果使用了git init初始化，则远程仓库的目录下，也包含work tree，当本地仓库向远程仓库push时,   如果远程仓库正在push的分支上（如果当时不在push的分支，就没有问题）, 那么push后的结果不会反应在work tree上（而且可能会出现上面的错误）,  也即在远程仓库的目录下对应的文件还是之前的内容，必须得使用git reset –hard才能看到push后的内容.