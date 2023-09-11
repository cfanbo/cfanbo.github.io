---
title: pushState + Ajax 技术
author: admin
type: post
date: 2017-05-18T01:27:20+00:00
url: /archives/17420
categories:
 - 前端设计
tags:
 - pjax

---
有时间我们要实现动态修改URL地址，同时更新部分页面内容，但不刷新整个页面，这种情况下用就需要用到一些pjax技术了。

window.history.replaceState 和 window.history.pushState 类似，不同之处在于replaceState不会在window.history里新增历史记录点，其效果类似于window.location.replace(url)，都是不会在历史记录点里新增一个记录点的。当你为了响应用户的某些操作，而要更新当前历史记录条目的状态对象或URL时，使用replaceState()方法会特别合适, 两者的参数是完全一样的。

window.history.replaceState(state, title , url) // “页面标题”目前浏览器暂不支持

state：一个与指定网址相关的状态对象{}，popstate事件触发时，该对象会传入回调函数。如果不需要这个对象，此处可以填null。

title：新页面的标题，但是所有浏览器目前都忽略这个值，因此这里可以填null。

url：新的网址，必须与当前页面处在同一个域。浏览器的地址栏将显示这个网址。