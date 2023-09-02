---
title: 如何关闭Sublime Text2的更新提示？
author: admin
type: post
date: 2013-08-07T02:27:25+00:00
url: /archives/14183
categories:
 - 其它
tags:
 - sublime

---
每次打开Sublime Text2时都会弹出更新提示，不想让它每次打开的时候都检测提示更新，可参考以下设置：

```
There is *update_check* field in Sublime version 2.0.1 build 2217.
Just go to Preferences -> Settings-User and add there: "update_check": false.
Sublime then stops checking for new version.
```