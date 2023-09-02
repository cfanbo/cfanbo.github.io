---
title: 在FreeBSD中运行 CVSup
author: admin
type: post
date: 2009-01-03T06:38:02+00:00
excerpt: |
 您现在准备尝试升级了。命令很简单：

 # cvsup supfile

 supfile 的位置当然就是您刚刚创建的 supfile 文件名啦。 如果您在 X11 下面运行，cvsup 会显示一个有一些可以做平常事情的按钮的 GUI 窗口。 按 go 按钮，然后看着它运行。

 在这个例子里您将要升级您目前的 /usr/src 树，您将需要 用 root 来运行程序，这样 cvsup 有需要的权限来更新您的文件。 刚刚创建了您的配置文件，又从来没有使用过这个程序，紧张不安是可以理解的。有一个简单的方法不改变您当前的文件来做一次试验性的运行。只要在方便的地方创建一个空目录，并在命令行上作为一个额外的参数说明：
url: /archives/785
IM_contentdowned:
 - 1
categories:
 - 服务器

---
您现在准备尝试升级了。命令很简单：

```
# cvsup supfile
```

 `supfile` 的位置当然就是您刚刚创建的 `supfile` 文件名啦。 如果您在 X11 下面运行， `cvsup` 会显示一个有一些可以做平常事情的按钮的 GUI 窗口。 按 go 按钮，然后看着它运行。

现在好像用csup这个命令的比较的多，速度比用cvsup要快,语法基本差不多，把命令关键字替换就可以了

> **csup -g -L2 -h cvsup4.freebsdchina.org /usr/share/examples/cvsup/ports-supfile**

在这个例子里您将要升级您目前的 `/usr/src` 树，您将需要 用 `root` 来运行程序，这样 `cvsup` 有需要的权限来更新您的文件。 刚刚创建了您的配置文件，又从来没有使用过这个程序，紧张不安是可以理解的。有一个简单的方法不改变您当前的文件来做一次试验性的运行。只要在方便的地方创建一个空目录，并在命令行上作为一个额外的参数说明：

```
# mkdir /var/tmp/dest
# cvsup supfile /var/tmp/dest
```

您指定的目录会作为所有文件更新的目的路径。 **CVSup** 会检查您在 `/usr/src` 中的文件，但是不会修改或删除。任何文件更新都会被放到 `/var/tmp/dest/usr/src` 里了。在这种方式下运行 **CVSup** 也会把它的 base 目录状态文件保持原样。这些文件的新版本 会被写到指定的目录。 因为您有 `/usr/src` 目录的读权限，所以执行这种试验性的运行 甚至不需要使用 `root` 用户。

如果您没有运行 X11 或者不喜欢 GUI， 当您运行 `cvsup` 的时候需要在命令行添加 两个选项：

```
# cvsup -g -L 2 supfile
```

 `-g` 告诉 **CVSup** 不要使用 GUI。如果您 没在运行 X11 这个是自动的，否则您必须指定它。

`-L 2` 告诉 **CVSup** 输出所有正在升级的文件的细节。 有三个等级可以选择，从 `-L 0` 到 `-L 2`。默认是 0，意味着除了错误消息 什么都不输出。

还有许多其它的选项可用。想要一个简短的列表， 输入 `cvsup -H`。要查看更详细的描述， 请查看手册页。

一旦您对升级工作的方式满意了，您就 可以使用 [cron(8)][1] 来安排规则的运行 **CVSup**。 很显然的，您不应该让 **CVSup** 通过 [cron(8)][1] 运行的时候使用它的 GUI。

 [1]: http://www.freebsd.org/cgi/man.cgi?query=cron&sektion=8