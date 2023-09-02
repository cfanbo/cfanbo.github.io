---
title: jquery.validate remote 和 自定义验证方法
author: admin
type: post
date: 2012-04-23T19:36:48+00:00
url: /archives/12798
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - jQuery

---
$(function(){

var validator = $(“#enterRegForm”).validate({
debug:false, //调试模式取消submit的默认提交功能
//errorClass: “error”,//默认为错误的样式类为：error
//validClass: “check”,//验证成功后的样式，默认字符串valid
focusInvalid: true,//表单提交时,焦点会指向第一个没有通过验证的域
//focusCleanup:true;//焦点指向错误域时，隐藏错误信息，不可与focusInvalid一起使用！
onkeyup: true,
errorElement: “div”,
submitHandler: function(form){ //表单提交句柄,为一回调函数，带一个参数：form
form.submit(); //提交表单
},

rules: {
“enterprise.enName”: {
required: true,
minlength: 6,
remote:{
url: “/nameServlet”,     //后台处理程序
type: “get”,               //数据发送方式
dataType: “json”,           //接受数据格式
data: {                     //要传递的数据
enName: function() {
return $(“#enName”).val();
}
}

}
},

“user.passWord”:{
required:true,
rangelength:[6,18]
},
passWordConf:{
required:true,
rangelength:[6,18],
equalTo:”#passWord” //此处的passWord 是 一开始还以为是name的值呢，气死了
}

},

messages: { //自定义验证消息
“enterprise.enName”: {
required: “请填写企业名称！”,
minlength: $.format(“至少要{0}个字符！”),
remote:$.format(“该企业名称已存在！”)

},
“user.passWord”:{
required:”请填写确认密码！”,
rangelength:$.format(“密码要在{0}-{1}个字符之间！”)
},
passWordConf:{
required:”请填写确认密码！”,
rangelength:$.format(“确认密码要在{0}-{1}个字符之间！”),
equalTo:”确认密码要和密码一致！”
},

errorPlacement: function(error, element) { //验证消息放置的地方
//error.appendTo( element.parent(“td”).next(“td”).children(“.msg”) );
error.appendTo( element.parent(“.field”).next(“div”));
},
highlight: function(element, errorClass) { //针对验证的表单设置高亮
$(element).addClass(errorClass);
},
success: function(div) {
div.addClass(“valid”);
}
});

});

```
required除了设置为true/false之外，还可以使用表达式或者函数，比如
$("#form2").validate({
	rules: {
		funcvalidate: {
			required: function() {return $("#password").val()!=""; }
		}
	},
	messages: {
		funcvalidate: {
			required: "有密码的情况下必填"
		}
	}
});

Html
密码<input id="password" name="password" type="password" />
确认密码<input id="confirm_password" name="confirm_password" type="password" />
条件验证<input id="funcvalidate" name="funcvalidate" value="" />
```



**自定义方法;**

新建一个js文件：$(document).ready(function(){
// 字符最小长度验证（一个中文字符长度为2）
jQuery.validator.addMethod(“stringMinLength”, function(value, element, param) {
var length = value.length;
for ( var i = 0; i < value.length; i++) {
if (value.charCodeAt(i) > 127) {
length++;
}
}
return this.optional(element) || (length >= param);
}, $.validator.format(“长度不能小于{0}!”));

// 字符最大长度验证（一个中文字符长度为2）
jQuery.validator.addMethod(“stringMaxLength”, function(value, element, param) {
var length = value.length;
for ( var i = 0; i < value.length; i++) {
if (value.charCodeAt(i) > 127) {
length++;
}
}
return this.optional(element) || (length <= param);
}, $.validator.format(“长度不能大于{0}!”));

// 字符验证
jQuery.validator.addMethod(“stringCheck”, function(value, element) {
return this.optional(element) || /^[\u0391-\uFFE5\w]+$/.test(value);
}, “只能包括中文字、英文字母、数字和下划线”);

// 中文字两个字节
jQuery.validator.addMethod(“byteRangeLength”, function(value, element, param) {
var length = value.length;
for(var i = 0; i < value.length; i++){
if(value.charCodeAt(i) > 127){
length++;
}
}
return this.optional(element) || ( length >= param[0] && length <= param[1] );
}, “请确保输入的值在3-15个字节之间(一个中文字算2个字节)”);

// 字符验证
jQuery.validator.addMethod(“string”, function(value, element) {
return this.optional(element) || /^[\u0391-\uFFE5\w]+$/.test(value);
}, “不允许包含特殊符号!”);
// 必须以特定字符串开头验证
jQuery.validator.addMethod(“begin”, function(value, element, param) {
var begin = new RegExp(“^” + param);
return this.optional(element) || (begin.test(value));
}, $.validator.format(“必须以 {0} 开头!”));
// 验证两次输入值是否不相同
jQuery.validator.addMethod(“notEqualTo”, function(value, element, param) {
return value != $(param).val();
}, $.validator.format(“两次输入不能相同!”));
// 验证值不允许与特定值等于
jQuery.validator.addMethod(“notEqual”, function(value, element, param) {
return value != param;
}, $.validator.format(“输入值不允许为{0}!”));

// 验证值必须大于特定值(不能等于)
jQuery.validator.addMethod(“gt”, function(value, element, param) {
return value > param;
}, $.validator.format(“输入值必须大于{0}!”));

// 验证值小数位数不能超过两位
jQuery.validator.addMethod(“decimal”, function(value, element) {
var decimal = /^-?\d+(\.\d{1,2})?$/;
return this.optional(element) || (decimal.test(value));
}, $.validator.format(“小数位数不能超过两位!”));
//字母数字
jQuery.validator.addMethod(“alnum”, function(value, element) {
return this.optional(element) || /^[a-zA-Z0-9]+$/.test(value);
}, “只能包括英文字母和数字”);
// 汉字
jQuery.validator.addMethod(“chcharacter”, function(value, element) {
var tel = /^[\u4e00-\u9fa5]+$/;
return this.optional(element) || (tel.test(value));
}, “请输入汉字”);
// 身份证号码验证
jQuery.validator.addMethod(“isIdCardNo”, function(value, element) {
return this.optional(element) || /^[1-9]\d{7}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])\d{3}$/.test(value)||/^[1-9]\d{5}[1-9]\d{3}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])((\d{4})|\d{3}[A-Z])$/.test(value);
}, “请正确输入您的身份证号码”);

// 手机号码验证
jQuery.validator.addMethod(“isMobile”, function(value, element) {
var length = value.length;
var mobile = /^(((13[0-9]{1})|(15[0-9]{1}))+\d{8})$/;
return this.optional(element) || (length == 11 && mobile.test(value));
}, “请正确填写您的手机号码”);

// 电话号码验证
jQuery.validator.addMethod(“isTel”, function(value, element) {
var tel = /^\d{3,4}-?\d{7,9}$/;    //电话号码格式010-12345678
return this.optional(element) || (tel.test(value));
}, “请正确填写您的电话号码”);

// 联系电话(手机/电话皆可)验证
jQuery.validator.addMethod(“isPhone”, function(value,element) {
var length = value.length;
var mobile = /^(((13[0-9]{1})|(15[0-9]{1}))+\d{8})$/;
var tel = /^\d{3,4}-?\d{7,9}$/;
return this.optional(element) || (tel.test(value) || mobile.test(value));

}, “请正确填写您的联系电话”);

// 邮政编码验证
jQuery.validator.addMethod(“isZipCode”, function(value, element) {
var tel = /^[0-9]{6}$/;
return this.optional(element) || (tel.test(value));
}, “请正确填写您的邮政编码”);

});

该文件要先被引用再对form进行验证。

注意：remote 中的url 为servlet，但是这个servlet该怎么写呢

一开始以为是跟平时的一样：out.println(“用户名”+name+”已存在！”);

虽然可以验证，但是还是存在问题：输入个不存在的name还是提示存在。

后来还是Google，看到别人也是犯了这样的错误，解决方法时返回true,false.

但是 doGet 是void类型的，return true 是行不通的。

后来又请教群里的高手，正确写法是out.println(“true”);就可以了。

哎，费了老长时间

总算是把问题给解决了。