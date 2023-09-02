---
title: WebApp 的 meta 标签
author: admin
type: post
date: 2015-01-12T06:56:28+00:00
url: /archives/15416
categories:
 - 程序开发
tags:
 - webapp

---
## apple-mobile-web-app-capable

设置Web应用是否以全屏模式运行。

语法：
:

说明：
: 如果content设置为yes，Web应用会以全屏模式运行，反之，则不会。content的默认值是no，表示正常显示。你可以通过只读属性window.navigator.standalone来确定网页是否以全屏模式显示。

兼容性：
: iOS 2.1 +

## apple-mobile-web-app-status-bar-style

设置Web App的状态栏（屏幕顶部栏）的样式

语法：
:

说明：
: 除非你先使用apple-mobile-web-app-capable指定全屏模式，否则这个meta标签不会起任何作用。



 如果content设置为default，则状态栏正常显示。如果设置为blank，则状态栏会有一个黑色的背景。如果设置为blank-translucent，则状态栏显示为黑色半透明。如果设置为default或blank，则页面显示在状态栏的下方，即状态栏占据上方部分，页面占据下方部分，二者没有遮挡对方或被遮挡。如果设置为blank-translucent，则页面会充满屏幕，其中页面顶部会被状态栏遮盖住（会覆盖页面20px高度，而iphone4和itouch4的Retina屏幕为40px）。默认值是default。

兼容性
: iOS 2.1 +

## format-detection

启动或禁用自动识别页面中的电话号码。

语法：
:

说明：
: 默认情况下，设备会自动识别任何可能是电话号码的字符串。设置telephone=no可以禁用这项功能。

兼容性
: iOS 1.0 +

## viewport

语法：
:

说明：
: 使用viewport meta标签可以提升页面在设备上的表现效果，典型地，你可以设置视口（viewport）的宽度和初始缩放比例。



 举个例子来说，如果页面的宽度小于980px，你可以设置视口的宽度以适应页面。如果你正在开发一款Web应用，你应该设置视口的宽度为设备的宽度。

 表 1 描述了viewport meta标签支持的属性以及它们的默认值。当有多个属性时，应该使用逗号分隔赋值表达式。设置多个属性时请遵循以下规则：

 * 不要使用分号作为分隔符。
 * 空格也可以作为分隔符，但最好使用逗号。
 * 对于属性值是数字的属性，如果属性值包含了非数字字符但是以数字开头，那么只有数字的部分被当做属性值。例如，1.0x等价于1.0，123×456等价于123。如果参数不以数字开头，则属性值为0。

 当要用到设备的尺寸数据时，你可以使用表2中的常量替代数字值。例如，使用device-width替代320（宽度），用device-height替代480（高度）。

 你不需要设置每一个属性，未设置的属性会自动采用默认值。

 设置视口的宽度为设备的宽度：



 设置初始缩放比例为1.0：



 设置初始缩放比例，同时禁止用户缩放。



兼容性
: iOS 1.0 +

表 1 Viewport 属性
 属性

 描述

 width

 视口的宽度。默认值是980，取值范围是200至10000，也可以取值为常量device-width。

 height

 视口的高度。默认值是根据width的值和设备的宽高比来计算，取值范围是223至10000，也可以取值为常量device-height。

 initial-scale

 视口的初始缩放比例。默认值取决于页面充满屏幕的缩放比例，取值范围取决于minimum-scale和maximum-scale。

 minimum-scale

 视口的最小缩放比例。默认值是0.25，取值范围是>0至10.0。

 maximum-scale

 视口的最大缩放比例。默认值是5.0，取值范围是>0至10.0。

 user-scable

 设置用户是否可以缩放视口。yes表示可以，no表示不能，默认值是yes。设置user-scable为no可以防止当用户在input标签的文本域中输入文本时页面发生滚动。
 表 2 特殊的viewport属性值
 属性值

 描述

 device-width

 设备的宽度（像素）。

 device-height

 设备的高度（像素）。


本文译自： [Safari HTML Reference](https://developer.apple.com/library/safari/documentation/AppleApplications/Reference/SafariHTMLRef/Articles/MetaTags.html#//apple_ref/doc/uid/TP40008193-SW1 "Safari HTML Reference")