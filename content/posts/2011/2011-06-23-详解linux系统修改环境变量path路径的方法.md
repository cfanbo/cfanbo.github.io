---
title: 详解Linux系统修改环境变量PATH路径的方法
author: admin
type: post
date: 2011-06-23T02:54:13+00:00
url: /archives/9938
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux
 - path

---
**关于PATH的作用：
** PATH说简单点就是一个字符串变量，当输入命令的时候LINUX会去查找PATH里面记录的路径。比如在根目录/下可以输入命令ls,在/usr目录下也可以输入ls,但其实ls这个命令根本不在这个两个目录下，事实上当你输入命令的时候LINUX会去/bin,/usr/bin,/sbin等目录下面去找你此时输入的命令，而PATH的值恰恰就是/bin:/sbin:/usr/bin:……。其中的冒号使目录与目录之间隔开。

 ****

**关于新增自定义路径：
** 现在假设你新安装了一个命令在/usr/locar/new/bin下面，而你又想像ls一样在任何地方都使用这个命令，你就需要修改环境变量PATH了，准确的说就是给PATH增加一个值/usr/locar/new/bin。你只需要一行bash命令export PATH=$PATH:/usr/locar/new/bin。这条命令的意思太清楚不过了，使PATH自增:/usr/locar/new/bin,既PATH=PATH+”:/usr/locar/new/bin”;通常的做法是把这行bash命令写到/root/.bashrc的末尾，然后当你重新登陆LINUX的时候（应该是linux启动时就会执行这个文件），新的默认路径就添加进去了。当然这里你直接用source /root/.bashrc执行这个文件重新登陆了。你可以用echo $PATH命令查看PATH的值。

 ****

**关于删除自定义路径：
** 当某天你发现你新增的路径/usr/locar/new/bin已经没用了的话，你可以修改/root/.bashrc文件里面你新增的路径。或者你可以修改/etc/profile文件删除你不需要的路径

—————————————————————————————-

电脑中必不可少的就是操作系统。而Linux的发展非常迅速，有赶超微软的趋势。这里介绍Linux的知识，让你学好应用Linux系统。比如要把/etc/apache/bin目录添加到PATH中，方法有三：

1.#PATH=$PATH:/etc/apache/bin
使用这种方法,只对当前会话有效，也就是说每当登出或注销系统以后，PATH 设置就会失效

2.#vi /etc/profile

在适当位置添加 PATH=$PATH:/etc/apache/bin （注意：＝ 即等号两边不能有任何空格）
这种方法最好,除非你手动强制修改PATH的值,否则将不会被改变

3.#vi ~/.bash_profile

修改PATH行,把/etc/apache/bin添加进去
这种方法是针对用户起作用的
注意：想改变PATH，必须重新登陆才能生效，以下方法可以简化工作：

如果修改了/etc/profile，那么编辑结束后执行source profile 或 执行点命令 ./profile,PATH的值就会立即生效了。
这个方法的原理就是再执行一次/etc/profile shell脚本，注意如果用sh /etc/profile是不行的，因为sh是在子shell进程中执行的，即使PATH改变了也不会反应到当前环境中，但是source是在当前 shell进程中执行的，所以我们能看到PATH的改变。

这样你就学会Linux系统下修改环境变量PATH路径的方法。