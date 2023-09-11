---
title: 延迟加载图片的 jQuery 插件：Lazy Load
author: admin
type: post
date: 2010-11-22T05:12:27+00:00
url: /archives/6749
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - jQuery

---
网站的速度非常重要，现在有很多网站优化的工具，如 Google 的 [Page Speed][1]，Yahoo 的 YSlow，对于网页图片，Yahoo 还提供 [Smush.it][2] 这个工具对图片进行批量压缩，但是对于图片非常多的网站，载入网页还是需要比较长的时间，这个时候我们可以使用 Lazy Load 这个 jQuery 插件来延迟加载图片。

Lazy loader 是一个延迟加载图片的 jQuery 插件，在一些图片非常多的网站中非常有用，在在浏览器可视区域外的图片不会被载入，直到用户将页面滚动到它们所在的位置才加载，这样对于含有很多图片的比 较长的网页来说，可以加载的更快，并且还能节省服务器带宽。

Lazy Loader 使用也非常简单，首先确保你的页面已经加载 jQuery Javascript 库，然后在加载 Lazy Load 的 Javascript 文件：

>

```
<script src="jquery.js" type="text/javascript"></script>
<script src="jquery.lazyload.js" type="text/javascript"></script>
```

然后在页面的 header 添加如下代码即可：

>

```
<script type="text/javascript"></script>
$(document).ready(function(){
    $("img").lazyload({
        placeholder : "/images/grey.gif",
        effect : "fadeIn"
    });
}
</script>
```

当然 Lazy Load 也有更多复杂的设置，你可以参考 [Lazy Load 原文介绍](http://www.appelsiini.net/projects/lazyload) 或者 [mg12 的翻译](http://www.neoease.com/lazy-load-jquery-plugin-delay-load-image/)。

我爱水煮鱼已经增加了这个功能，你可以在一些[图片较多的日志页面][3]预览下。

我这时用的是其它网站上提供的两个js,推荐使用:

>
>

来源： [http://fairyfish.net/2010/07/07/jquery-lazy-load/](http://fairyfish.net/2010/07/07/jquery-lazy-load/)

 [1]: http://fairyfish.net/2009/06/08/google-page-speed/
 [2]: http://fairyfish.net/2009/12/14/smush-it/
 [3]: http://fairyfish.net/2010/07/05/best-real-estate-wordpress-theme/