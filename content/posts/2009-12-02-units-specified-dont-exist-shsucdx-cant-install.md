---
title: Units specified don’t exist SHSUCDX can’t install
author: admin
type: post
date: 2009-12-02T01:24:46+00:00
excerpt: |
 |
 用的是联想thinkpad品牌笔记本的，在dos下用光盘来ghost xp系统的时候，经常出现：Units specified don't exist SHSUCDX can't install，然后就没有反映了，试了三张安装盘都不行的，光盘也都没有什么问题的呀，真是奇了怪了！

 网上有人说用PE进入到系统里，使用里面的ghost来恢复，找了几个安装盘，发现在PE系统里识别不到硬盘，巧的是也没有ghost安装文件，没有办法心想可能是品牌机本本设置的问题，后来在网上baidu了一下，发现了解决的办法，如下：

 BIOS里面硬盘模式AHＣI改为IDE~
 要到BIOS将硬盘的模式改成compatibility模式 就可以装
url: /archives/2651
IM_contentdowned:
 - 1
categories:
 - 其它
tags:
 - ghost

---
用的是联想thinkpad品牌笔记本的，在dos下用光盘来ghost xp系统的时候，经常出现：Units specified don’t exist SHSUCDX can’t install，然后就没有反映了，试了三张安装盘都不行的，光盘也都没有什么问题的呀，真是奇了怪了！

网上有人说用PE进入到系统里，使用里面的ghost来恢复，找了几个安装盘，发现在PE系统里识别不到硬盘，巧的是也没有ghost安装文件，没有办法心想可能是品牌机本本设置的问题，后来在网上baidu了一下，发现了解决的办法，如下：

```
BIOS里面硬盘模式AHＣI改为IDE~
要到BIOS将硬盘的模式改成compatibility模式 就可以装

上面是我参考的网上的回答

我个人意见是品牌机有保护需要在blos里把保护去掉，然后重新格式化硬盘

后来到BIOS里面将硬盘格式改为SATA（据说类似于IDE），重装，OK……搞定

我认为也是这样的，blos里面对硬盘的设置有问题

按以下方法成功解决问题！
呵呵，这个是问题的最终原因了

主要是sata硬盘驱动无法加载,我也是费了好大的劲才找到这个好东西,非常好用,我经成功把sr28的系统换成了xp.. J4 q' \0 B3 z
安装盘下载地址:http://www.namipan.com/d/Sony.Xp ... 4ba3d790b6d00804129/ V0 G- A+ j; O$ v4 }
安装盘下载后直接做成光盘.
安装前先把区分了,注意可以不删本来的隐藏分区,我用diskgen分的区.接下来就是用做的安装盘安装就行了.

你说的现象我没遇到过

只能帮你在网上找答案了

我给你的解决方法 因该是网上都有能找到的方法了

至于你说的那个格成ex格式的那个我认为不对，ex是linux的文件系统
xp用不上

希望能帮到你

另外用pe格盘很容易
与林木风盘里面因该就有pe
你可以在网上下个pq硬盘板

网上有个绿色版本很好
还是单文件的我用过

你可以试试

希望能帮到你
```