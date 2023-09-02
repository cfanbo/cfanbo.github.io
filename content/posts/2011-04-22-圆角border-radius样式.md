---
title: 圆角(border-radius)样式
author: admin
type: post
date: 2011-04-22T00:59:36+00:00
url: /archives/9378
IM_data:
 - 'a:2:{s:62:"http://www.leadbbs.com/a/file.asp?lid=13251&s=sLvGA3pLXuii";s:89:"http://blog.haohtml.com/wp-content/uploads/2011/04/37bb8.asp?lid=13251&s=sLvGA3pLXuii";s:62:"http://www.leadbbs.com/a/file.asp?lid=13250&s=sLvGA3pLXuii";s:89:"http://blog.haohtml.com/wp-content/uploads/2011/04/0817d.asp?lid=13250&s=sLvGA3pLXuii";}'
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - css

---
建议参考: [http://www.css3.info/preview/rounded-border/](http://www.css3.info/preview/rounded-border/)

圆角样式示例(仅在firefox内核,safari,chrome等内核浏览器下支持,IE内核不支持)

> **border-\*-\*-radius:** [  | <%> ] [  | <%> ]?

**CSS3的border-radius规范**

 1. 属性：
 border-top-right-radius
 border-bottom-right-radius
 border-bottom-right-radius
 border-bottom-right-radius
 值：。它们分别是定义角形状的四分之一椭圆的两个半径。如图：



 1. 第一个值是水平半径。
 2. 如果第二个值省略，则它等于第一个值，这时这个角就是一个四分之一圆角。
 3. 如果任意一个值为0，则这个角是矩形，不会是圆的。
 4. 值不允许是负值。

 1. 属性：border-radius。它是上面四个属性值的简写。
 值：{1,4} [ / {1,4} ]


 1. 如果斜线前后的值都存在，那么斜线前的值设置水平半径，且斜线后的值设置垂直半径。如果没有斜线，则水平半径和垂直半径相等。
 2. 四 个值是按照top-left、top-right、 bottom-right、 bottom-left的顺序来设置的。如果bottom-left省略，那么它等于top-right。如果bottom-right省略，那么它等于 top-left。如果top-right省略，那么它等于top-left。
 2. 应用范围：所有的元素，除了table的样式属性border-collapse是collapse时
 3. 内边半径等于外边半径减去对应边的厚度。当这个结果是负值时，内边半径是0。所以内外边曲线的圆心并不一定是一致的。
 4. border-radius也会导致该元素的背景也是圆的，即使border是none。如果background-clip是padding-box，则背景（background）会被曲线的内边裁剪。如果是border-box则被外边裁剪。border和padding定义的区域也一样会被曲线裁剪。
 5. 所有的边框样式（solid、dotted、inset等）都遵照角的曲线。如果设置了border-image，则曲线以外的部分会被裁剪掉。
 6. 如果角的两个相邻边有不同的宽度，那么这个角将会从宽的边平滑过度到窄的边。其中一条边甚至可以是0。
 7. 两条相邻边颜色和样式转变的中心点是在一个和两边宽度成正比的角上。比如，两条边宽度相同，这个点就是一个45°的角上，如果一条边是另外一条边的两倍，那么这个点就在一个30°的角上。界定这个转变的线就是连接在内外曲线上的两个点的直线
 8. 角 不允许相互重叠，所以当相邻两个角半径的和大于所在矩形区域的大小时，用户代理（浏览器）比如缩小一个或多个角半径。运算法则如下：f = min(Li/Si)，i ∈ {top, right, bottom, left}，Ltop = Lbottom = 所在矩形区域的宽，Lleft = Lright = 所在矩形区域的高。如果f < 1，那么所有角半径都乘以f。

**实际CSS应用,需要根据不同浏览器HACK**

> 01. -moz-border-radius-topleft: 11px;
>
> 02. -moz-border-radius-topright: 11px;
>
> 03. -khtml-border-top-left-radius: 11px;
>
> 04. -khtml-border-top-right-radius: 11px;
>
> 05. -webkit-border-top-left-radius: 11px;
>
> 06. -webkit-border-top-right-radius: 11px;
>
> 07. border-top-right-radius: 11px;
>
> 08. border-top-left-radius: 11px;
>
> 09. -moz-border-radius-bottomleft: 11px;
>
> 10. -moz-border-radius-bottomright: 11px;
>
> 11. -khtml-border-bottom-left-radius: 11px;
>
> 12. -khtml-border-bottom-right-radius: 11px;
>
> 13. -webkit-border-bottom-left-radius: 11px;
>
> 14. -webkit-border-bottom-right-radius: 11px;
>
> 15. border-bottom-left-radius: 11px;
>
> 16. border-bottom-right-radius: 11px;

可简写为

> 1. -moz-border-radius: 15px;
>
> 2. -khtml-border-radius: 15px;
>
> 3. -webkit-border-radius: 15px;
>
> 4. border-radius: 15px;