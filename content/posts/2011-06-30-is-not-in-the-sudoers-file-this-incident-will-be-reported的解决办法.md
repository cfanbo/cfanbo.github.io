---
title: is not in the sudoers file. This incident will be reported的解决办法
author: admin
type: post
date: 2011-06-30T06:26:03+00:00
url: /archives/10167
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - sudo

---
**在一般用户下执行sudo命令提示xxx is not in the sudoers file. This incident will be reported.解决方法：**

> $whereis sudoers
> /etc/sudoers

有时候我们只需要执行一条root权限的命令也要su到root，是不是有些不方便？这时可以用sudo代替。默认新建的用户不在sudo组，需要编辑/etc/sudoers文件将用户加入，该文件只能使用visudo命令，

1) 首先需要切换到root, su – (注意有- ，这和su是不同的，在用命令”su”的时候只是切换到root，但没有把root的环境变量传过去，还是当前用乎的环境变量，用”su -“命令将环境变量也一起带过去，就象和root登录一样)

2) 然后 visudo 或者 vim /etc/sudoers, visudo 这个和vi的用法一样，由于可能会有人不太熟悉vi，所以简要说一下步骤

移动光标，到一行 root ALL=(ALL)   ALL 的下一行，添加一行

> your\_user\_name ALL=(ALL)   ALL

然后保存退出!

这样就把自己加入了sudo组，可以使用sudo命令了。

3) 默认5分钟后刚才输入的sodo密码过期，下次sudo需要重新输入密码，如果觉得在sudo的时候输入密码麻烦，把刚才的输入换成如下内容即可：

> your\_user\_name ALL=(ALL) NOPASSWD: ALL