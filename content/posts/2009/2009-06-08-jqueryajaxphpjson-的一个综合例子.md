---
title: jQuery/Ajax/PHP/Json 的一个综合例子
author: admin
type: post
date: 2009-06-07T23:17:12+00:00
excerpt: |
 jQuery 是一个优秀的 Javascript 框架，对 js 进行了优秀的包装，提供了许多方便的功能。jQuery 对 ajax 的包装也堪称优秀。

 jQuery 可以以 json 文件传输协议来传输数据(类似 xml，而且大有取代 xml 的趋势)，而网站后台代码必须与之配合使用。PHP 是用 json_encode 函数来对返回的数组数据进行编码的，但这个函数只有 PHP5.2版本以上才支持。
url: /archives/1647
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - jQuery
 - json

---
jQuery 是一个优秀的 Javascript 框架，对 js 进行了优秀的包装，提供了许多方便的功能。jQuery 对 ajax 的包装也堪称优秀。

jQuery 可以以 json 文件传输协议来传输数据(类似 xml，而且大有取代 xml 的趋势)，而网站后台代码必须与之配合使用。PHP 是用 json_encode 函数来对返回的数组数据进行编码的，但这个函数只有 PHP5.2版本以上才支持。

从网上找到一个 json 的操作类，本人在 PHP4.4.7 版本下测试通过。本人还建了个函数 function my\_json\_encode($phparr)，使代码兼容 PHP5.2 以上版本。

示例代码(包括 json 的类包软件)可以在以下网址下载：

以下是全部代码：



jQuery Ajax 实例演示

输入姓名:

输入年龄:

输入性别:

输入工作:

提交POST提交GET提交



PHP 文件 ajax_json.php：

encode($phparr);
}
}
?>

 [1]: http://blog.csdn.net/zhangking/archive/2008/11/26/%27ajax_json.php%27