---
title: 基于mootools重写js(tab,tree)控件包
author: admin
type: post
date: 2008-03-01T12:36:41+00:00
url: /archives/272
IM_data:
 - 'a:2:{s:103:"http://www.javaeye.com/upload/attachment/7132/d7a07383-d8f5-4c5a-8e20-7293a5ba52b6-thumb.jpg?1199174530";s:103:"http://www.javaeye.com/upload/attachment/7132/d7a07383-d8f5-4c5a-8e20-7293a5ba52b6-thumb.jpg?1199174530";s:103:"http://www.javaeye.com/upload/attachment/7133/3a4bfaa8-907a-4e09-b8e4-e3cc4b1c68e4-thumb.jpg?1199174530";s:103:"http://www.javaeye.com/upload/attachment/7133/3a4bfaa8-907a-4e09-b8e4-e3cc4b1c68e4-thumb.jpg?1199174530";}'
IM_contentdowned:
 - 1
categories:
 - 前端设计

---

以前写过一个js包,里面的tab和tree都是纯粹用js的function手写,没有使用框架,

存在几个问题


1. 扩展比较困难

2. 接下去在添加新的控件,没有一个统一的实现方式,显得混乱,不好管理


基于以上理由,重新基于 mootools1.1 重写了tab和tree控件,为将来添加更多的控件打个好的基础


代码中有详细的注释,也有demo,一看全明白了


下面附上源码和效果图


- [demo.rar](http://www.javaeye.com/topics/download/1a5be506-47ce-44c7-b946-8efad4f8304f) (80.1 KB)

- 描述: 示例和源码

- 下载次数: 708


- [![D7a07383-d8f5-4c5a-8e20-7293a5ba52b6-thumb](http://www.javaeye.com/upload/attachment/7132/d7a07383-d8f5-4c5a-8e20-7293a5ba52b6-thumb.jpg?1199174530)](http://www.javaeye.com/topics/download/d7a07383-d8f5-4c5a-8e20-7293a5ba52b6)
- 描述: tab效果图

- 大小: 5.1 KB

- 查看次数: 240


- [![3a4bfaa8-907a-4e09-b8e4-e3cc4b1c68e4-thumb](http://www.javaeye.com/upload/attachment/7133/3a4bfaa8-907a-4e09-b8e4-e3cc4b1c68e4-thumb.jpg?1199174530)](http://www.javaeye.com/topics/download/3a4bfaa8-907a-4e09-b8e4-e3cc4b1c68e4)
- 描述: tree效果图

- 大小: 2 KB

- 查看次数: 117