---
title: jQuery Datepicker 中文
author: admin
type: post
date: 2012-08-05T10:43:08+00:00
url: /archives/13243
categories:
 - 前端设计
tags:
 - Datepicker
 - jQuery

---
[![](http://blog.haohtml.com/wp-content/uploads/2012/08/jquery-ui-datepicker-chinese.png)][1]

以前在使用 js 日历时，没有使用过 [jQuery Datepicker](http://jqueryui.com/demos/datepicker/)，今天第一次使用发现非常的好用。使用时需要将日历文字显示为中文，打开前边的链接在文章底部就可以看到将 jQuery Datepicker 文字显示为中文的方法，在 [http://jquery-ui.googlecode.com/svn/trunk/ui/i18n/](http://jquery-ui.googlecode.com/svn/trunk/ui/i18n/) 可以看到各种版本的语言，中文文件内容如下：

```
jQuery(function($){
        $.datepicker.regional['zh-CN'] = {
                closeText: '关闭',
                prevText: '<上月',
                nextText: '下月>',
                currentText: '今天',
                monthNames: ['一月','二月','三月','四月','五月','六月',
                '七月','八月','九月','十月','十一月','十二月'],
                monthNamesShort: ['一','二','三','四','五','六',
                '七','八','九','十','十一','十二'],
                dayNames: ['星期日','星期一','星期二','星期三','星期四','星期五','星期六'],
                dayNamesShort: ['周日','周一','周二','周三','周四','周五','周六'],
                dayNamesMin: ['日','一','二','三','四','五','六'],
                weekHeader: '周',
                dateFormat: 'yy-mm-dd',
                firstDay: 1,
                isRTL: false,
                showMonthAfterYear: true,
                yearSuffix: '年'};
        $.datepicker.setDefaults($.datepicker.regional['zh-CN']);
});
```

使用时引入该文件， jQuery Datepicker 就会显示简体中文，但为了网站性能，最好将它整合到一个 js 文件中。

```
$(document).ready(function() {
    $("#sdate,#edate").datepicker();
});
```

使用时也可以添加其他的选项，如下所示，具体的可以参考 [jQuery Datepicker](http://jqueryui.com/demos/datepicker/) 官方文档。

```
$(document).ready(function () {
    $("#sdate,#edate").datepicker({
        numberOfMonths: 2,  //显示两个月
        minDate: 0  //从当前日期起可选
    });
});
```

本篇简单介绍了下 jQuery UI Datepicker 显示中文的方法，希望能对遇到这个问题又不太熟悉英文的朋友带来一点帮助。

相关文章：

 [1]: http://blog.haohtml.com/wp-content/uploads/2012/08/jquery-ui-datepicker-chinese.png