---
title: 自定义jquery validate 插件的默认提示语
author: admin
type: post
date: 2011-04-18T22:38:44+00:00
url: /archives/9342
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - jQuery
 - jquery插件

---
**法一：**

一、默认校验规则
(1)required:true 必输字段
(2)remote:”check.php” 使用ajax方法调用check.php验证输入值
(3)email:true 必须输入正确格式的电子邮件
(4)url:true 必须输入正确格式的网址
(5)date:true 必须输入正确格式的日期
(6)dateISO:true 必须输入正确格式的日期(ISO)，例如：2009-06-23，1998/01/22 只验证格式，不验证有效性
(7)number:true 必须输入合法的数字(负数，小数)
(8)digits:true 必须输入整数
(9)creditcard: 必须输入合法的信用卡号
(10)equalTo:”#field” 输入值必须和#field相同
(11)accept: 输入拥有合法后缀名的字符串（上传文件的后缀）
(12)maxlength:5 输入长度最多是5的字符串(汉字算一个字符)
(13)minlength:10 输入长度最小是10的字符串(汉字算一个字符)
(14)rangelength:[5,10] 输入长度必须介于 5 和 10 之间的字符串”)(汉字算一个字符)
(15)range:[5,10] 输入值必须介于 5 和 10 之间
(16)max:5 输入值不能大于5
(17)min:10 输入值不能小于10

二、默认的提示
messages: {
required: “This field is required.”,
remote: “Please fix this field.”,
email: “Please enter a valid email address.”,
url: “Please enter a valid URL.”,
date: “Please enter a valid date.”,
dateISO: “Please enter a valid date (ISO).”,
dateDE: “Bitte geben Sie ein g眉ltiges Datum ein.”,
number: “Please enter a valid number.”,
numberDE: “Bitte geben Sie eine Nummer ein.”,
digits: “Please enter only digits”,
creditcard: “Please enter a valid credit card number.”,
equalTo: “Please enter the same value again.”,
accept: “Please enter a value with a valid extension.”,
maxlength: $.validator.format(“Please enter no more than {0} characters.”),
minlength: $.validator.format(“Please enter at least {0} characters.”),
rangelength: $.validator.format(“Please enter a value between {0} and {1} characters long.”),
range: $.validator.format(“Please enter a value between {0} and {1}.”),
max: $.validator.format(“Please enter a value less than or equal to {0}.”),
min: $.validator.format(“Please enter a value greater than or equal to {0}.”)
},
如需要修改，可在js代码中加入：
jQuery.extend(jQuery.validator.messages, {
required: “必选字段”,
remote: “请修正该字段”,
email: “请输入正确格式的电子邮件”,
url: “请输入合法的网址”,
date: “请输入合法的日期”,
dateISO: “请输入合法的日期 (ISO).”,
number: “请输入合法的数字”,
digits: “只能输入整数”,
creditcard: “请输入合法的信用卡号”,
equalTo: “请再次输入相同的值”,
accept: “请输入拥有合法后缀名的字符串”,
maxlength: jQuery.validator.format(“请输入一个长度最多是 {0} 的字符串”),
minlength: jQuery.validator.format(“请输入一个长度最少是 {0} 的字符串”),
rangelength: jQuery.validator.format(“请输入一个长度介于 {0} 和 {1} 之间的字符串”),
range: jQuery.validator.format(“请输入一个介于 {0} 和 {1} 之间的值”),
max: jQuery.validator.format(“请输入一个最大为 {0} 的值”),
min: jQuery.validator.format(“请输入一个最小为 {0} 的值”)
});
推荐做法，将此文件放入messages_cn.js中，在页面中引入

============================================

**法二：**

我们查看下jQuery.Validate源代码，在236行果然有提示消息的定义方式，我们就可以进行修改这边的消息改成中文的方式，或者自定义 了一个中文的消息jQuery.Validate.message_cn.js，然后使用jQuery.extend来覆盖 jQuery.Validate自身的消息，代码如下：

```

  //定义中文消息
  var cnmsg = {
      required: "必选字段",
      remote: "请修正该字段",
      email: "请输入正确格式的电子邮件",
      url: "请输入合法的网址",
      date: "请输入合法的日期",
      dateISO: "请输入合法的日期 (ISO).",
      number: "请输入合法的数字",
      digits: "只能输入整数",
      creditcard: "请输入合法的信用卡号",
      equalTo: "请再次输入相同的值",
      accept: "请输入拥有合法后缀名的字符串",
      maxlength: jQuery.format("请输入一个长度最多是 {0} 的字符串"),
      minlength: jQuery.format("请输入一个长度最少是 {0} 的字符串"),
      rangelength: jQuery.format("请输入一个长度介于 {0} 和 {1} 之间的字符串"),
      range: jQuery.format("请输入一个介于 {0} 和 {1} 之间的值"),
      max: jQuery.format("请输入一个最大为 {0} 的值"),
      min: jQuery.format("请输入一个最小为 {0} 的值")
  };
  jQuery.extend(jQuery.validator.messages, cnmsg);



```