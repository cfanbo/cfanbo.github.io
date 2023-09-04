---
title: CentOS系统如何查看cpu
author: admin
type: post
date: 2011-04-11T08:13:18+00:00
url: /archives/9236
IM_data:
 - 'a:1:{s:60:"http://images.51cto.com/files/uploadimg/20100402/1147240.jpg";s:67:"http://blog.haohtml.com/wp-content/uploads/2011/04/5385_1147240.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - cpu

---
在使用一个系统的时候，我们肯定要看看系统配置，而CentOS系统下看配置，可没有windows那么直观。你只能一个一个查看。如何查看CentOS系统的CPU也就让很多人不知所措了。

下面，我们就来学习一下如何在CentOS系统中查看CPU吧。

**一：CentOS系统cpu**

[root@srv /]# more /proc/cpuinfo | grep “model name”

model name    : Intel(R) Xeon(R) CPU          X3220 @ 2.40GHz

model name    : Intel(R) Xeon(R) CPU          X3220 @ 2.40GHz

model name    : Intel(R) Xeon(R) CPU          X3220 @ 2.40GHz

model name    : Intel(R) Xeon(R) CPU          X3220 @ 2.40GHz

[root@srv /]# grep “model name” /proc/cpuinfo

model name    : Intel(R) Xeon(R) CPU          X3220 @ 2.40GHz

model name    : Intel(R) Xeon(R) CPU          X3220 @ 2.40GHz

model name    : Intel(R) Xeon(R) CPU          X3220 @ 2.40GHz

model name    : Intel(R) Xeon(R) CPU          X3220 @ 2.40GHz

[root@srv /]# grep “model name” /proc/cpuinfo | cut -f2 -d:

Intel(R) Xeon(R) CPU          X3220 @ 2.40GHz

Intel(R) Xeon(R) CPU          X3220 @ 2.40GHz

Intel(R) Xeon(R) CPU          X3220 @ 2.40GHz

Intel(R) Xeon(R) CPU          X3220 @ 2.40GHz

[![](https://blogstatic.haohtml.com//uploads/2023/09/1147240.gif)][1]

这样，我们就查看了CentOS系统的CPU。你也就了解了你的电脑。

更多信息请参考：

[1]: http://blog.haohtml.com/wp-content/uploads/2011/04/1147240.gif