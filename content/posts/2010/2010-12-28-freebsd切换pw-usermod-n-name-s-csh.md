---
title: '[freebsd切换]pw usermod -n name -s csh'
author: admin
type: post
date: 2010-12-28T01:12:02+00:00
url: /archives/7316
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - home

---
**1、让Freebsd终端也支持彩色**

ls -G就会显示彩色
csh在.cshrc文件中，添加：alias ls=”ls -G”
sh在.profile文件中，添加：alias ls=”ls -G”

**2、更改用户登陆shell**

默认安装是使用sh登陆的，sh不支持TAB键
要切换到csh，直接运行csh即可

如果需要一劳永逸，那么用下面这个命令
name：是指你登陆的名称
pw usermod -n name -s csh

**3、更换提示符**

set prompt = ” yztgx@hotmail.com # ”
也可以将这句话加到.cshrc或者.profile配置文件中

**4、Freebsd下支持dir**

alias dir “ls”
也可以将这句话加到.cshrc或者.profile配置文件中

alias类似Dos下的doskey

Linux下的修改方法参见: