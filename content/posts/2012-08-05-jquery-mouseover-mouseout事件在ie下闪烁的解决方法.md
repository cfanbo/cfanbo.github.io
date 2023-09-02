---
title: jQuery mouseover mouseout事件在IE下闪烁的解决方法
author: admin
type: post
date: 2012-08-05T08:01:06+00:00
url: /archives/13239
categories:
 - 前端设计
tags:
 - jQuery

---
>

```
$("#category ul").find("li").each(
    function() {
        $(this).mouseover(
            function() {
                $("#" + this.id + "_menu").show();
                $(this).addClass("a");
            }
        );
        $(this).mouseout(
            function() {
                $(this).removeClass("a");
                $("#" + this.id + "_menu").hide();
            }
        );
    }
);
```

浏览器之间的不兼容一直令前端开发者的头疼，而 IE 更是噩梦。鼠标在下拉菜单移动时菜单会不断闪烁，说明不断触发了 mouseover 和 mouseout 事件。

这貌似涉及到所谓的“事件冒泡”，我不懂 JavaScript，就不在误人子弟了，详情请自己 Google，这里只给出解决方法：将 mouseover 改成 **mouseenter**，mouseout 改成 **mouseleave**。

>

```
$("#category ul").find("li").each(
    function() {
        $(this).mouseenter(
            function() {
                $("#" + this.id + "_menu").show();
                $(this).addClass("a");
            }
        );
        $(this).mouseleave(
            function() {
                $(this).removeClass("a");
                $("#" + this.id + "_menu").hide();
            }
        );
    }
);
```

最后指出一点，mouseenter 和 mouseleave 事件是 jQuery 库中实现的，似乎（再次声明我不懂 JavaScript，所以这里用“似乎”）并不是浏览器的原生事件，所以如果你想自己写 JavaScript 实现的话，请自行 Google，或者查看 jQuery 源码，如果你能看懂的话。

_参考链接：_[_JQuery HowTo: Problems with jQuery mouseover / mouseout events_][1]

转自：http://demon.tw/programming/jquery-mouseover-mouseout.html

 [1]: http://jquery-howto.blogspot.com/2009/04/problems-with-jquery-mouseover-mouseout.html