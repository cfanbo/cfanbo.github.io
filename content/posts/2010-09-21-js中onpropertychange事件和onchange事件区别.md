---
title: JS中onpropertychange事件和onchange事件区别
author: admin
type: post
date: 2010-09-21T02:14:13+00:00
url: /archives/5788
IM_contentdowned:
 - 1
categories:
 - 前端设计

---
当一个HTML元素的属性用js改变的时候，都能通过 onpropertychange来捕获。例如一个 对象的value属性被页面的脚本修改的时候，onchange无法捕获到，而onpropertychange却能够捕获。
也就是说：onpropertychange事件在用键盘每改变一下文本框的值或用js改变其值便会触发一下，而onchange只有在用键盘改变其值，然后在失去焦点(onblur)后才触发，用js改变其值不能触发!onpropertychange和onchange都不管文本框中的实际值有没有变，只要有改的相应操作就可能触发。有时当上面两时间都不能满足需求时，可以考虑只用onblur。

还有一点要注意到，当onblur和onchange事件一起用时，onblur会出问题。。。。详见如下

测试页面：


 通过js改变文本框中的值后触发的事件：onpropertychange事件

**测试onpropertychange事件和onchange事件一起用时：**

测试结果：onpropertychange事件在用键盘每改变一下文本框的值或用js改变其值便会触发一下，而onchange只有在用键盘改变其值，然后在失去焦点后才触

发，用js改变其值不触发

**测试只有onblur和onchange事件时：**

测试结果：onchange先触发，onblur后触发

**测试当onblur和onpropertychange事件一起用时：**

测试结果：onblur好象出了问题，只要用键盘在文本框中随便输入一个值，便会触发它。可能是onpropertychange把它惹毛了。。。^-^

**测试有onblur、onpropertychange事件和onchange事件一起用时：**

测试结果：onblur在和onpropertychange一起用时的问题仍然存在



注意:此方法在firefox下无效,支持方法请见: [IE下onpropertychange与firefox下oninput用法](http://blog.haohtml.com/index.php/archives/6518)