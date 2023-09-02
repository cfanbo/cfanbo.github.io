---
title: ios8中action segue
author: admin
type: post
date: 2015-05-01T17:37:33+00:00
url: /archives/15678
categories:
 - 程序开发
tags:
 - swift

---
os8 action segue 有几种方法，一般选择哪一个，每种方法都有什么用，在什么环境下使用？

[![57_400607_5f360e47ff35d47](http://blog.haohtml.com/wp-content/uploads/2015/05/57_400607_5f360e47ff35d47.jpg)][1]

Apple的解释在这里： [https://developer.apple.com/library/ios/recipes/xcode_help-IB_storyboard/chapters/StoryboardSegue.html](https://developer.apple.com/library/ios/recipes/xcode_help-IB_storyboard/chapters/StoryboardSegue.html)
我的翻译：
**Show**: 在master或detail区域展现内容(典型的如iPad的设置界面，左侧是master，右侧是detail)，究竟是在哪个区要取决于屏幕上的内容，如果不分master/detail，就单纯的把新的内容push到当前view controller stack的顶部
**Show Detail**: 大致同Show，在detail区域展现内容，如果不分master/detail，新的内容取代当前view controller stack的顶部
**Present Modally**：模态展示内容
**Present as Popover**：在当前的view上出现一个小窗口来展示内容，无处不在的“选中文字后出现 复制/翻译 按钮就是这个
**Custom**：自定义的

 [1]: http://blog.haohtml.com/wp-content/uploads/2015/05/57_400607_5f360e47ff35d47.jpg