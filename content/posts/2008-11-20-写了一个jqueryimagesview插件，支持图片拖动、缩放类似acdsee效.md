---
title: 写了一个jquery.imagesview插件，支持图片拖动、缩放类似ACDSEE效果
author: admin
type: post
date: 2008-11-19T22:19:45+00:00
url: /archives/620
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - jQuery

---
[][1]     做项目的时候客户总是比较关心前台界面，这不最近又遇到一个难缠的客户。要求在前台的缩略图点开后查看高分辨率的图片，并且最好能像ACDSEE那样方便浏览，支持拖动、按比例放大缩小。

     这样的效果记得在SINA的某个栏目看到过，影响中好像是FLASH做的，我很多年没碰过FLASH了，对现在的ACTION脚本不是很熟悉，于是想到结合JQUERY来实现，JQUERY的ThickBox支持图片浏览但不是客户要的效果，在GOOGLE搜索了半天没找到一个好用的JS代码，只好自己来写了。

     先看效果（**点下面的图片和连接即可**）



[][2]{.imagesview}[![](http://blog.haohtml.com/wp-content/uploads/2008/11/rs4-199x300.jpg)][3]

[同样支持文字连接][2]{.imagesview}

支持 IE 7+、FireFox 1.5+ (PC, Mac)、Safari2+、Opera9+、Chrome0.3+



　　 我测试了一下应该在 谷歌浏览器、IE7、FIREFOX3.0 下都能看到效果的，呵。。

　　 为了给大家展示这个效果可费了我不少时间，主要是这个博客不支持在内容里定义CSS样式文件。

       而且CNBLOGS后台编辑对 Chrome 支持不是很好，只好开N个浏览器来回折腾~~

　　 看不到效果的只好看下面这张图了：



[![](http://blog.haohtml.com/wp-content/uploads/2008/11/1.jpg)][1]
　　 **相关文件（文章最后的附件中都有这些文件）： **

**————————————————————————————————     imagesview.css　　–样式文件**

　　 imagesview.js　　 –ImageView JS文件

　　 toolsico.gif　　 –按钮图标

　　 loadingAnimation.gif　 –加载动画

　　 jquery.js　　 –JQUERY脚本库，请使用jQuery 1.2.6 以上版，否则拖动有点问题

　　 **使用方法：**

————————————————————————————————页面头部引用 imagesview.css、imagesview.js、jquery.js即可（两个GIF图片文件和CSS文件放在同一目录）

　　 图片连接： [![](”images/rs4a.jpg”)](”images/rs4.jpg” "”ImageView") 文字连接： [同样支持文字连接](”images/rs4.jpg” "”ImageView") 其中 class=”imagesview”  是必须的，title=”。。”  这里是标题，可以不定义。

　 所有相关文件附件中提供下载：
          附件： [jquery.imagesview.rar][4]

 [1]: http://blog.haohtml.com/wp-content/uploads/2008/11/1.jpg
 [2]: http://files.cnblogs.com/relax/imagesview/rs4.jpg "ImageView code by jinks.zhang"
 [3]: http://blog.haohtml.com/wp-content/uploads/2008/11/rs4.jpg
 [4]: http://files.cnblogs.com/relax/jquery.imagesview.rar "jquery.imagesview.rar"