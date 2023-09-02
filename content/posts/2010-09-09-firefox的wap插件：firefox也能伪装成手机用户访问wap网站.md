---
title: Firefox的wap插件：Firefox也能伪装成手机用户访问wap网站
author: admin
type: post
date: 2010-09-09T07:38:41+00:00
url: /archives/5638
IM_contentdowned:
 - 1
categories:
 - 其它
tags:
 - firefox
 - wap

---
要想Firefox能正常解析手机的wap网站，首先需要安装wml解析插件wmlbrowser。

**wmlbrowser 0.7.13** [https://addons.mozilla.org/en-US/firefox/addon/62](https://addons.mozilla.org/en-US/firefox/addon/62)

另外许多程序对来访者的user-agent进行了判断，所以还需要安装自定义user-agent的插件User Agent Switcher。

**User Agent Switcher 0.6.10** [https://addons.mozilla.org/en-US/firefox/addon/59](https://addons.mozilla.org/en-US/firefox/addon/59) 安装好以上两个插件后，重启Firefox，然后工具->User Agent Switcher->OPtions->User Agents->Add ，填写在 Description:Wap , User Agent: Symbian 确定 ，最后 工具->User Agent Switcher-> 选中刚设定的 Wap ，即可畅通无阻的浏览Wap站啦 !

用了这些插件后，你就能模拟手机用户访问wap网站了（尤其对一些同时支持WEB/WAP方式的论坛有用）！