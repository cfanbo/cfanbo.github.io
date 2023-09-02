---
title: FreeBSD下的fsck命令
author: admin
type: post
date: 2011-10-20T04:26:08+00:00
url: /archives/11803
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - fsck

---
对文件系统进行检查，并对损害的文件系统进行修复。
**fsck的语法如下：**
fsck (-F fstype) (-v) (-m) (-special…)
fsck (-F fstype) (-v) (-y|Y|n|N)
(-o fstype options) (special…)
其中：
-F fstype : 说明被检查的文件系统的类型
-v : 返回完成的命令行，但不运行
-y|Y: 对所有问题均回答Yes
-n|N: 对所有问题均回答No
-m: 对文件系统进行检查，不修复文件系统，
如果文件系统经检查后是可安装的，则显示
ufs fsck : sanity check : /dev/rdsk/c0t0d0s0 okay.
-o: 文件系统类型选项，选项由逗号分隔，


**最常用的选项有两个： **
P: 整理(preen)模式
F: 强制检查模式，此选项忽略文件系统状态标志。
1) 移去一个没有相关文件的目录入口　答Yes或Y来删除该目录入口
2) 重连接一个已分配但不能访问的文件
对fsck的”RECONNECT？”回答Yes，即把该I节点连接到lost+found目录下，文件名即是I节点号
3) 连接数调整　回答Yes或Y来改正连接数
4) 自由块表不一致　回答Yes或Y来修正超级块
对于fsck询问的问题大多数情况下都可以用Yes来回答，所以在实际应用时，可以用” -y”选项来执行该命令
对硬盘进行检查和修复。