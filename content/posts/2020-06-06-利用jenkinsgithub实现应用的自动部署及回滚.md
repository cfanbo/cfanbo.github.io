---
title: 利用jenkins+github实现应用的自动部署及回滚
author: admin
type: post
date: 2020-06-06T04:31:25+00:00
url: /archives/20069
categories:
 - 程序开发
tags:
 - cicd
 - jenkins

---
对于jenkins的介绍这里不再详细写了，此教程只是为了让大家对部署和回滚原理有所了解。

## 一、创建项目 

点击左侧的“New Item”,输入项目名称，如 rollback-demo。

选中 ” 丢弃旧的构建（Discard old builds）”项，在“策略(Strategy” 选择”_Log Rotation_“, 并输入保留的最大构建个数。

## 二、常规配置 ![05a2833329fee18776c5682a1068d288](https://blogstatic.haohtml.com/uploads/2020/06/16e3bc0afb4cf3f179f02fc598c220cd.png)

设置参数，点击”_Add Parameter_“,依次选择 “_Choice Parameter_” 和 “_String Parameter_“这两，填写如下![ac7d45c20f8b6777ab153ce75439dafb](https://blogstatic.haohtml.com/uploads/2020/06/3193669cca6c1da1bdde929ec1666ff2.png)

这里的Name 项为参数名称，用户在操作的时候，会在deploy 和 rollback 两个值中选择一项。

## 三、源码管理 

我们这里选择Git.并填写github.com上的项目地址，记得设置认证 Credentials。构建分支直接使用默认的 */master 即可以了。查看代码浏览器选择 githubweb，并填写项目的github地址。

## 四、构建触发事件 

选择 “GitHub hook trigger for GITScm polling”，表示使用github webhook来触发构建操作，要实现引功能，需要在项目地址github.com里的“setting”里添加一个webhook的url地址，一般地址为_http://jenkins.com/github-webhook/_

同时为了防止网络通讯不稳定的情况，同时选择 “Poll SCM”, 在调度Schedule 杠中填写 H/5 \* \* \* \*，表示5分钟自动从github上拉取数据一次，如果有变化就进行构建。

## 五、构建环境 

我们演示为了简单，使用了php项目，这里不进行任何操作。如果java、NodeJS或者Golang的话，可能需要进行一些操作， 有时为了方便会把这些操作放在下一步shell脚本里进行。

## 六、构建配置 

1.添加构建步骤，点击Add build step，选择“Execute shell”,填写内容如下

```
#!/bin/bash
case $deploy_env in
deploy)
	echo "deploy $deploy_env"
    ;;
rollback)
	echo "rollback $deploy_env version=$version"
    cp -R ${JENKINS_HOME}/jobs/rollback-demo/builds/${version}/archive/*.* ./
    pwd && ls
    ;;
    *)
    exit
    ;;
esac
```

可以看到当rollback的时候，是从原来构建归档路径里把文件复制出来。

这里正常情况下应该有一些单元测试之类的脚本，这里省略不写了。

2.添加构建后的操作。点击“Add post-build action” -> “Archive the artifacts” 配置用于归档的文件为“*\*/\*“，表示所有文件。这点十分重要，只有每次构建完归档了才有东西回滚，另外时间长了，归档的内容越来越多，所以上面设置了最大归档个数。
3.再次添加”Send build artifacts over SSH”,配置内容如下![38bcc14ff20beef4f8b1278044912567](https://blogstatic.haohtml.com/uploads/2020/06/eaa24fc790a1e52281b0305e58400c12.png)

注意 shell脚本里的路径比 Remote directory 的路径里多一个/data 目录，这是由于在配置 ssh server 的时候指定了一个根目录为 /data.

如果在 Add post-build action 中找不到send build artifacts over ssh ，则说明需要安装一下插件，左侧点击“Manage Jenkins”-> “Manage Plugis”, 搜索“[Publish Over SSH][1]”安装即可。

到这里为了基本配置完成了。

## 测试配置 

这里我们首次手动构建一次，点击项目页面左侧菜单的“build with Parameter”,显示如下![f926257717a3e9b0b9ba90648b5c572b](https://blogstatic.haohtml.com/uploads/2020/06/3346b1fae79e3b79a550191fb9eaa6be.png)

在正常deploy的时候，version字段时忽略掉即可。如果要加滚的话，则需要选择”rollback”,同时填写 version字段号，这个字段号为页面左下角build history的编号，就是以#开始的那些数字

 [1]: https://plugins.jenkins.io/publish-over-ssh