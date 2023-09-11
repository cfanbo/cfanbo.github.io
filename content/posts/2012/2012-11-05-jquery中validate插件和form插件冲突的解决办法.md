---
title: jquery中validate插件和form插件冲突的解决办法
author: admin
type: post
date: 2012-11-05T08:19:08+00:00
url: /archives/13482
categories:
 - 前端设计
tags:
 - jQuery
 - validate

---
如题：用jquery form提交表单，用jquery validate做数据验证 ，现在的问题是分别使用validate有作用，一起使用，则validate不起作用，谁遇到过帮忙解决下。

02.   $(document).ready(function() {

03.     $(“#inputForm”).validate({

04.     …

05.          });

06.   });

07.   function onsubmit(){

08.     var options ={

09.        …

10.     };

11.     $(‘#inputForm’).ajaxSubmit(options); //options

12.     return false;

13.   }


==================

补充一下，这个submitHandler:function(){}方法内可以写任何方法。但最后要有一个form.submit或form.ajaxSubmit
比如我这个

01. $(document).ready(function(){

02.             $(“#loginForm”).validate({

03.                 rules: {

04.                     shouJiHaoMa: {

05.                         required: true,

06.                         digits: true

07.                     },

08.                     pwd: {

09.                         required: true

10.                     }

11.                 },

12.                 messages: {

13.                     shouJiHaoMa: {

14.                         required: ‘请输入手机号码’,

15.                         digits: ‘请输入正确的手机号码’

16.                     },

17.                     pwd: {

18.                         required: ‘请输入密码’

19.                     }

21.                 },

22.                 // specifying a submitHandler prevents the default submit, good for the demo

23.                 submitHandler: function() {

24.                             login();

25.                 return false;

26.                 },

27.                 errorElement: “em”,

28.                 success: function(label) {

30.                 label.text(” “)             //清空错误提示消息

31.                     .addClass(“success”);   //加上自定义的success类

32.                 }

33.             });


我的login()内最后提交了form

01. login = function() {

02.     $(‘#login_tmp’).remove();

03.     var options = {

04.         type : “get”,

05.         dataType : “json”,

06.         url : “login.do”,

07.         data : {“method” : “ajaxLogin”},

08.         cache : “false”,

09.         beforeSubmit : beforeCallBack,

10.         success : loginCallBack,

11.         error : errorCallBack

12.     };

13.     // 异步提交登陆请求

14.     $(“#loginForm”).ajaxSubmit(options);

15. }


参考： [http://www.iteye.com/problems/35657](http://www.iteye.com/problems/35657)