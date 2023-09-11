---
title: 基于 GitHub Actions 实现 Golang 项目的自动构建部署
author: admin
type: post
date: 2020-04-21T05:32:13+00:00
url: /archives/19997
categories:
 - 程序开发
tags:
 - golang

---
前几天 GitHub官网宣布 GitHub 的所有核心功能对所有人都免费开放,不得不说自从微软收购了GitHub后，确实带来了一些很大的改变。

以前有些项目考虑到协作关系的原因，虽然放在github上面，但对于一些项目的持续构建和部署一般是通过自行抢建Travis CI、jenkins等系统来实现。虽然去年推出了Actions用来代替它类三方系统，但感觉着还是不方便，必须有些核心功能无法使用，此消息的发布很有可能将这种格局打破。

本篇教程将介绍使用github的系列产品来实现项目的发布，构建，测试和部署，当然这仅仅是一个非常小的示例，有些地方后期可能会有更好的瞿恩方案。

GitHub Actions 是一款持续集成工具，包括clone代码，代码构建，程序测试和项目发布等一系列操作。更多内容参考：

如果你对CI/CD不了解的话，建议先找些文档看看。

项目源文件见

## GitHub Actions 术语

GitHub Actions 相关的术语。

（1）workflow （工作流程）：持续集成一次运行的过程，就是一个 workflow。

（2）job （任务）：一个 workflow 由一个或多个 jobs 构成，含义是一次持续集成的运行，可以完成多个任务。

（3）step（步骤）：每个 job 由多个 step 构成，一步步完成。

（4）action （动作）：每个 step 可以依次执行一个或多个命令（action）。

## 一、创建workflow 文件

在项目里创建一个workflow文件，文件格式为yaml类型。文件名可以随意起，文件后缀可以为yml 或 .yaml, 这里我们创建文件 .github/workflows/deploy.yaml，注意这里的路径。

如果你的仓库中有项目文件的话，当你点击“Actions”时，系统会自动根据你的开发语言推荐一个常用的actions，在页面的右侧也会推荐一些相应的actions.

## 二、在部署服务器上生成部署用户密钥

部署时需要用到用户的私钥，所以先登录到部署服务器获取私钥，这里为了方便，单独创建了一对公钥和公钥，

`$ cd ~/.ssh`
`$ ssh-keygen`
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id\_rsa): /root/.ssh/id\_rsa_actions
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id\_rsa\_actions.\****
Your public key has been saved in /root/.ssh/id\_rsa\_actions.pub.
The key fingerprint is:
SHA256:QPg26V17/pnRdHZrDqysG6jFgdTEysbz+aCuXusyO/Q root@iZbp1acq02ar70gdvppgh6Z
The key’s randomart image is:
+—[RSA 2048]—-+
| .o. |
| ..o. |\****
| oooo |
| .**. . |
| .+o+S. . =|
| . o++ . o ++|
| . …+o. o o.o.|
| +.E+ .o o ++ |
| .+O= ooo .+. |
+—-[SHA256]—–+

这里为部署单独生成了一对公钥和私钥，私钥路径为 /root/.ssh/id\_rsa\_actions

将公钥保存到 authorized_keys 文件中
`$ cat id_rsa_actions.pub >> authorized_keys`

查看私钥内容，后面需要用到，先将内容保存起来
`$ cat id_rsa_actions`

## 三、准备工作

对于服务的部署所以这里选择了 ssh-deploy 这一个actioins，官方网址 https://github.com/marketplace/actions/ssh-deploy。

下面开始添加deploy.yaml文件中用到了一些变量。

在项目首页右上角点击 Seetings->Secets, 找到 Add a new secret，分别添加以下变量

> SERVER\_SSH\_KEY 登录私钥，就是上面保存的私钥内容
> REMOTE_HOST 服务器地址，如202.102.224.68
> REMOTE_PORT (可选项)服务器ssh端口，一般默认为22
> REMOTE_USER 登录服务器用户名，这时指密钥所属的用户
> SOURCE (可选项)，默认为‘’, 构建服务器路径，这里为相应 $GITHUB_WORKSPACE 根目录而言的相对路径, 例如 dist/
> REMOTE_TARGET 服务器部署路径，如 /data/ghactions
> ARGS (可选项)默认值为 -rltgoDzvO

## 四、上传代码到github远程仓库

我们这里定义了当master分支发生push操作时就触发一系列workflow操作。
`$git push origin master`
这时我们可以在 https://github.com/cfanbo/github-actions-demo/actions 页面看到当前项目的构建情况。

## 五、测试

这里我们登录到远程服务器，可以发现一个server可执行文件，表示已经成功部署到生产服务器了

## 六、其它

有关 workflow 的一些环境变量可参考 https://help.github.com/en/actions/configuring-and-managing-workflows/using-environment-variables

如果你有过CI/CD的经历，会发现本教程虽然实现了最为初级的功能，离真正线上使用还有一定的距离。主要存在以下问题：
1. 教程里只用了一个job，如果你的项目允许的话，完全可以利用多个job来同时异步构建。
2. 这里构建、测试和部署同时放在了一个job里，也不是太优雅
3. 这里的部署只是将最终生成的二进制文件上传到了生产服务器，并没有对服务进行启动操作或者说没有进行服务的热更新，这在一般场景下是不允许的
由于本篇文章只是一个简单介绍actions基本用法的教程，所以如果想真正用到工作中的话，可能还需要对李篇内容再完善完善才行。

## 参考

* https://help.github.com/en/articles/workflow-syntax-for-github-actions
* https://help.github.com/en/actions/configuring-and-managing-workflows/using-environment-variables
* https://github.com/marketplace/actions/checkout
* https://github.com/marketplace/actions/setup-go-for-use-with-actions
* https://github.com/marketplace/actions/ssh-deploy
* http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html