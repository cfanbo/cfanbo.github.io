---
title: jquery Form 验证 validate插件使用
author: admin
type: post
date: 2011-04-18T22:29:55+00:00
url: /archives/9340
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - jQuery

---
不过我们还要在需要验证的INPUT里面class加入required说明是必填项，其他的就是验证相关数据比如email就是验证email的数据结构

以下列出validate自带的默认验证

> required: “必选字段”,
>
> remote: check.php “使用ajax方法调用check.php验证输入值段”,
>
> email: “请输入正确格式的电子邮件”,
>
> url: “请输入合法的网址”,
>
> date: “请输入合法的日期”,
>
> dateISO: “请输入合法的日期 (ISO).”,
>
> number: “请输入合法的数字”,
>
> digits: “只能输入整数”,
>
> creditcard: “请输入合法的信用卡号”,
>
> equalTo: “请再次输入相同的值”,
>
> accept: “请输入拥有合法后缀名的字符串”,
>
> maxlength: jQuery.format(“请输入一个长度最多是 {0} 的字符串”),
>
> minlength: jQuery.format(“请输入一个长度最少是 {0} 的字符串”),
>
> rangelength: jQuery.format(“请输入一个长度介于 {0} 和 {1} 之间的字符串”),
>
> range: jQuery.format(“请输入一个介于 {0} 和 {1} 之间的值”),
>
> max: jQuery.format(“请输入一个最大为 {0} 的值”),
>
> min: jQuery.format(“请输入一个最小为 {0} 的值”)



对于如何修改插件的默认提示语方法请参考：

[http://blog.haohtml.com/archives/9342](http://blog.haohtml.com/archives/9342) validate ()的可选项

**debug：**进行调试模式

> $(“.selector”).validate({ debug: true})
>
> 把调试设置为默认
>
> $.validator.setDefaults({
>
> debug: true
>
> })



**submitHandler：**用其他方式替代默认的SUBMIT,比如用AJAX的方式提交



> $(“.selector”).validate({
>
> submitHandler: function(form) {
>
> $(form).ajaxSubmit();
>
> }
>
> })



**ignore：**忽略某些元素不验证

> $(“#myform”).validate({ ignore: “.ignore”})



**rules：** 默认下根据form的classes, attributes, metadata判断，但也可以在validate函数里面声明

Key/value 可自定义规则. Key是对象的名字 value是对象的规则，可以组合使用 class/attribute/metadata rules.

以下代码特别验证selector类中name=’name’是必填项并且 email是既是必填又要符合email的格式

> $(“.selector”).validate({
>
> rules: {
>
> // simple rule, converted to {required:true}
>
> name: “required”,
>
> // compound rule
>
> email: {
>
> required: true,
>
> email: true
>
> }
>
> }
>
> })



**messages：**更改默认的提示信息

> $(“.selector”).validate({
>
> rules: {
>
> name: “required”,
>
> email: {
>
> required: true,
>
> email: true
>
> }
>
> },
>
> messages: {
>
> name: “Please specify your name”,
>
> email: {
>
> required: “We need your email address to contact you”,
>
> email: “Your email address must be in the format of name@domain.com”
>
> }
>
> }
>
> })

**groups：**定义一个组，把几个地方的出错信息同意放在一个地方，用errorPlacement控制把出错信息放在哪里

> $(“#myform”).validate({
>
> groups: {
>
> username: “fname lname”
>
> },
>
> errorPlacement: function(error, element) {
>
> if (element.attr(“name”) == “fname” || element.attr(“name”) == “lname” )
>
> error.insertAfter(“#lastname”);
>
> else
>
> error.insertAfter(element);
>
> },
>
> debug:true
>
> })

对于更多的使用方法请参考： [http://zhudp-cn.iteye.com/blog/255168](http://zhudp-cn.iteye.com/blog/255168)