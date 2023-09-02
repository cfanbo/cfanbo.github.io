---
title: Ubuntu下apt-get 命令参数
author: admin
type: post
date: 2010-09-02T05:54:37+00:00
url: /archives/5430
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Ubuntu

---
常用的APT命令参数

apt-cache search package 搜索包

apt-cache show package 获取包的相关信息，如说明、大小、版本等

sudo apt-get install package 安装包

sudo apt-get install package – – reinstall 重新安装包

sudo apt-get -f install 修复安装”-f = ――fix-missing”

sudo apt-get remove package 删除包

sudo apt-get remove package – – purge 删除包，包括删除配置文件等

sudo apt-get update 更新源

sudo apt-get upgrade 更新已安装的包

sudo apt-get dist-upgrade 升级系统

sudo apt-get dselect-upgrade 使用 dselect 升级

apt-cache depends package 了解使用依赖

apt-cache rdepends package 是查看该包被哪些包依赖

sudo apt-get build-dep package 安装相关的编译环境

apt-get source package 下载该包的源代码

sudo apt-get clean && sudo apt-get autoclean 清理无用的包

sudo apt-get check 检查是否有损坏的依赖

其中：

1 有SUDO的表示需要管理员特权！

2 在UBUNTU中命令后面参数为短参数是用“-”引出，长参数用“――”引出

3 命令帮助信息可用man 命令的方式查看或者

命令 -H（――help）方式查看

4 在MAN命令中需要退出命令帮助请按“q”键！！

选项 含义 作用

sudo -h Help 列出使用方法，退出。

sudo -V Version 显示版本信息，并退出。

sudo -l List 列出当前用户可以执行的命令。只有在sudoers里的用户才能使用该选项。

sudo -u username|#uid User 以指定用户的身份执行命令。后面的用户是除root以外的，可以是用户名，也可以是#uid。

sudo -k Kill 清除“入场卷”上的时间，下次再使用sudo时要再输入密码。

sudo -K Sure kill 与-k类似，但是它还要撕毁“入场卷”，也就是删除时间戳文件。

sudo -b command Background 在后台执行指定的命令。

sudo -p prompt command Prompt 可以更改询问密码的提示语，其中%u会代换为使用者帐号名称，%h会显示主机名称。非常人性化的设计。

sudo -e file Edit 不是执行命令，而是修改文件，相当于命令sudoedit。