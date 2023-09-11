---
title: jquery.validate.js简介
author: admin
type: post
date: 2010-03-31T18:34:29+00:00
url: /archives/3157
IM_data:
 - 'a:1:{s:43:"http://www.javaeye.com/images/icon_copy.gif";s:69:"http://blog.haohtml.com/wp-content/uploads/2011/03/ef22_icon_copy.gif";}'
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - jQuery

---

参看: [http://docs.jquery.com/Plugins/Validation](http://docs.jquery.com/Plugins/Validation) 并整理


jquery.validate.js是jquery旗下的一个验证框架 [http://bassistance.de/jquery-plugins/jquery-plugin-validation/](http://bassistance.de/jquery-plugins/jquery-plugin-validation/)，借助jquery的优势，我们可以迅速验证一些常见的输入,并且可以自己 扩充自己的验证方法，并且对国际化也有很好的支持.

使用这个函数很简单，看以下的代码


Html代码


02. “http://www.w3.org/TR/html4/loose.dtd”**>**
03. **<html>**
04. **<head>**
05. **<script** src=“http://code.jquery.com/jquery-latest.js” **>script>**
06. **<link** rel=“stylesheet”href=“http://dev.jquery.com/view/trunk/plugins/validate/jquery.validate.css”type=“text/css”media=“screen”**/>**
07. **<script** type=“text/javascript”src=“http://dev.jquery.com/view/trunk/plugins/validate/jquery.validate.js” **>script>**
08. **<style** type=“text/css”**>**
09. * { font-family: Verdana; font-size: 96%; }
10. label { width: 10em; float: left; }
11. label.error { float: none; color: red; padding-left: .5em; vertical-align: top; }
12. p { clear: both; }
13. .submit { margin-left: 12em; }
14. em { font-weight: bold; padding-right: 1em; vertical-align: top; }
15. **style>**
16. **<script>**
17. $(document).ready(function(){
18. $(“#commentForm”).validate();
19. });
20. **script>**
22. **head>**
23. **<body>**
26. **<form** class=“cmxform”id=“commentForm”method=“get”action=“”**>**
27. **<fieldset>**
28. **<legend>** A simple comment form with submit validation and default messages **legend>**
29. **<p>**
30. **<label** for=“cname”**>**Name **label>**
31. **<em>*** **em><input** id=“cname”name=“name”size=“25”class=“required”minlength=“2”**/>**
32. **p>**
33. **<p>**
34. **<label** for=“cemail”**>**E-Mail **label>**
35. **<em>*** **em><input** id=“cemail”name=“email”size=“25”class=“required email”**/>**
36. **p>**
37. **<p>**
38. **<label** for=“curl”**>**URL **label>**
39. **<em>** **em><input** id=“curl”name=“url”size=“25”class=“url”value=“”**/>**
40. **p>**
41. **<p>**
42. **<label** for=“ccomment”**>**Your comment **label>**
43. **<em>*** **em><textarea** id=“ccomment”name=“comment”cols=“22”class=“required” **>textarea>**
44. **p>**
45. **<p>**
46. **<input** class=“submit”type=“submit”value=“Submit”**/>**
47. **p>**
48. **fieldset>**
49. **form>**
50. **body>**
51. **html>**

```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
                    "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
  <script src="http://code.jquery.com/jquery-latest.js"></script>
  <link rel="stylesheet" href="http://dev.jquery.com/view/trunk/plugins/validate/jquery.validate.css" type="text/css" media="screen" />
<script type="text/javascript" src="http://dev.jquery.com/view/trunk/plugins/validate/jquery.validate.js"></script>
<style type="text/css">
* { font-family: Verdana; font-size: 96%; }
label { width: 10em; float: left; }
label.error { float: none; color: red; padding-left: .5em; vertical-align: top; }
p { clear: both; }
.submit { margin-left: 12em; }
em { font-weight: bold; padding-right: 1em; vertical-align: top; }
</style>
  <script>
  $(document).ready(function(){
    $("#commentForm").validate();
  });
  </script>

</head>
<body>

 <form id="commentForm" method="get" action="">
 <fieldset>
   <legend>A simple comment form with submit validation and default messages</legend>
   <p>
     <label for="cname">Name</label>
     <em>*</em><input id="cname" name="name" size="25" minlength="2" />
   </p>
   <p>
     <label for="cemail">E-Mail</label>
     <em>*</em><input id="cemail" name="email" size="25"  />
   </p>
   <p>
     <label for="curl">URL</label>
     <em>  </em><input id="curl" name="url" size="25"  value="" />
   </p>
   <p>
     <label for="ccomment">Your comment</label>
     <em>*</em><textarea id="ccomment" name="comment" cols="22" ></textarea>
   </p>
   <p>
     <input type="submit" value="Submit"/>
   </p>
 </fieldset>
 </form>
</body>
</html>
```

总结，我们只要在加入如下的JAVASCRIPT代码，就可以把指定的FORM加上验证


Html代码


1. $(document).ready(function(){
2. $(“#commentForm”).validate();
3. });

```
$(document).ready(function(){
    $("#commentForm").validate();
  });
```

不过我们还要在需要验证的INPUT里面class加入required说明是必填项，其他的就是验证相关数据比如email就是验证email的 数据结构


以下列出validate自带的默认验证


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

maxlength: jQuery.format(“请输入一个长度最多是 {0} 的字符串”),

minlength: jQuery.format(“请输入一个长度最少是 {0} 的字符串”),

rangelength: jQuery.format(“请输入一个长度介于 {0} 和 {1} 之间的字符串”),

range: jQuery.format(“请输入一个介于 {0} 和 {1} 之间的值”),

max: jQuery.format(“请输入一个最大为 {0} 的值”),

min: jQuery.format(“请输入一个最小为 {0} 的值”)


### **validate** ()的可选项

### debug：进行调试模式

```
$

(".selector").validate

({
   debug: true
})
```

把调试设置为默认


```
$

.validator.setDefaults

({
   debug: true
})
```

**submitHandler：用其他方式替代默认的SUBMIT,比如用AJAX的方式提交**

```
$

(".selector").validate

({
   submitHandler: function(form) {
    $

(form).ajaxSubmit();
   }
})
```

**ignore：忽略某些元素不验证**

```
$

("#myform").validate

({
   ignore: ".ignore"
})
```

rules： 默认下根据form的classes, attributes, metadata判断，但也可以在validate函数里面声明

Key/value 可自定义规则. Key是对象的名字 value是对象的规则，可以组合使用 class/attribute/metadata rules.


以下代码特别验证selector类中name=’name’是必填项并且 email是既是必填又要符合email的格式


```
$

(".selector").validate

({
   rules: {
     // simple rule, converted to {required:true}
     name: "required",
     // compound rule
     email: {
       required: true,
       email: true
     }
   }
})
```

messages：更改默认的提示信息


```
$

(".selector").validate

({
   rules: {
     name: "required",
     email: {
       required: true,
       email: true
     }
   },
   messages: {
     name: "Please specify your name",
     email: {
       required: "We need your email address to contact you",
       email: "Your email address must be in the format of name@domain.com"
     }
   }
})
```

groups：定义一个组，把几个地方的出错信息同意放在一个地方，用errorPlacement控制把出错信息放在哪里


```
$

("#myform").validate

({
  groups: {
    username: "fname lname"
  },
  errorPlacement: function(error, element) {
     if (element.attr("name") == "fname" || element.attr("name") == "lname" )
       error.insertAfter("#lastname");
     else
       error.insertAfter(element);
   },
   debug:true
 })
```

- [jquery.validate.zip](http://www.javaeye.com/topics/download/b6dbcd26-b6f5-3c25-94b4-0979ee0ecabb) (211.8 KB)

- 下载次数: 248