---
title: lighttpd配置DiscuzX伪静态规则详细图文教程
author: admin
type: post
date: 2010-06-17T03:54:54+00:00
url: /archives/3854
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - discuz

---
VPS下lighttpd配置DiscuzX伪静态规则

第一步：进入Kloxo VPS控制面板点击域名选项.
第二步：进入域名选项后，选中你所要配置lighttpd的DiscuzX伪静态规则的域名， 这里所需要配置的域名为找iPad论坛 [www.cn0393.com](http://www.cn0393.com) , 点击它，进入站点选项列表.
第三步：点击lighttpd地址重写规则按纽,进入lighttpd配置界面.
第四步：把由找ipad论坛提供的lighttpd的DiscuzX伪静态规则文件粘贴进空白框中.
第五步：然后点击Update按纽，应用并使lighttpd生效.
        注：lighttpd会自动重启可以不用像IIS那样需要手工重启。
第六步：进入DiscuzX管理中心，点击–》全局–》优化设置–》URL静态化
        把箭头所指的勾全选中–》点提交
最后一步，就是更新一下缓存就OK了，超简单。哥你懂的，就不截图了。自已看效果吧！