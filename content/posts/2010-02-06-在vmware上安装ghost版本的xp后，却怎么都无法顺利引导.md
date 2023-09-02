---
title: 在VMware上安装Ghost版本的XP后，却怎么都无法顺利引导
author: admin
type: post
date: 2010-02-06T16:04:59+00:00
excerpt: |
 你是否遇到以下情况，在VMware上安装Ghost版本的XP后，却怎么都无法顺利引导，更进不了系统。

 总是提示：No boot filename received、Operating System not found。这到底是什么原因 ？Lee也曾经被此问题困扰了好久，百思不得其解。
url: /archives/2916
IM_data:
 - 'a:7:{s:74:"http://www.huashifu.net/sites/default/files/images/3059323261229855726.gif";s:79:"http://blog.haohtml.com/wp-content/uploads/2011/03/6de9_3059323261229855726.gif";s:82:"http://www.huashifu.net/sites/default/files/images/1350605471229854343.preview.jpg";s:87:"http://blog.haohtml.com/wp-content/uploads/2011/03/2611_1350605471229854343.preview.jpg";s:82:"http://www.huashifu.net/sites/default/files/images/3276528151229854645.preview.JPG";s:87:"http://blog.haohtml.com/wp-content/uploads/2011/03/1d42_3276528151229854645.preview.JPG";s:82:"http://www.huashifu.net/sites/default/files/images/1591419581229854880.preview.jpg";s:87:"http://blog.haohtml.com/wp-content/uploads/2011/03/d198_1591419581229854880.preview.jpg";s:82:"http://www.huashifu.net/sites/default/files/images/7027148951229854960.preview.jpg";s:87:"http://blog.haohtml.com/wp-content/uploads/2011/03/816f_7027148951229854960.preview.jpg";s:82:"http://www.huashifu.net/sites/default/files/images/6304706601229855005.preview.jpg";s:87:"http://blog.haohtml.com/wp-content/uploads/2011/03/736a_6304706601229855005.preview.jpg";s:82:"http://www.huashifu.net/sites/default/files/images/9555402021229855176.preview.jpg";s:87:"http://blog.haohtml.com/wp-content/uploads/2011/03/395e_9555402021229855176.preview.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 其它
tags:
 - vmware

---
你是否遇到以下情况，在VMware上安装Ghost版本的XP后，却怎么都无法顺利引导，更进不了系统。![](http://www.huashifu.net/sites/default/files/images/3059323261229855726.gif)

总是提示：No boot filename received、Operating System not found。这到底是什么原因 ？Lee也曾经被此问题困扰了好久，百思不得其解。

今天，终于搞定了。原因就在于PQ分区时没有把新建的分区设定为作用的。



## 详细描述故障：

下载了GhostXP\_SP3电脑公司特别版\_v10，一个很不错的版本。

像往常一样，新建一个虚拟机，默认设置，一直到选择从加载镜像的虚拟光驱启动， 启动进入光盘引导界面，如下图：

![](http://www.huashifu.net/sites/default/files/images/1350605471229854343.preview.jpg)

选择 “PM8.05图形化分区工具”选项，启动PQ开始分区，轻车熟路。

分区完成后，再次重启进入上面的光盘界面，选择“1把系统安装到硬盘第一分区”选项。

开始Ghost了，等啊等，Ghost完成了，屁颠屁颠的等着进XP，结果就等到这个界面，如下图：

![](http://www.huashifu.net/sites/default/files/images/3276528151229854645.preview.JPG)

就这样，一直进不了系统。重启数次，反反复复， 复复反反，就是进不去。



## 问题解决办法：

这种情况多发生在手动使用PQ给虚拟硬盘分区并GHOST安装XP后。明明在BIOS里设置了从硬盘引导，可就是找不到硬盘。

解决方法是再次启动PQ，右键单击C盘，选择“进阶——设定为作用”。如下图：

![](http://www.huashifu.net/sites/default/files/images/1591419581229854880.preview.jpg)

在弹出的对话框中确认操作，如下图：

![](http://www.huashifu.net/sites/default/files/images/7027148951229854960.preview.jpg)

我们看C盘的状态已经变成“作用”，单击“执行”按钮，如下图：

![](http://www.huashifu.net/sites/default/files/images/6304706601229855005.preview.jpg)

之后重启。

我们就可以欣喜的看到久违的XP滚动界面了 ，如下图：

![](http://www.huashifu.net/sites/default/files/images/9555402021229855176.preview.jpg)