---
title: 解决多个window.onload冲突问题的方法
author: admin
type: post
date: 2010-07-06T01:09:15+00:00
url: /archives/4406
IM_contentdowned:
 - 1
categories:
 - 前端设计

---
我们在网页中可能要插入多个JS，可是如果在两个JS的代码中都使用了window.onload,可能会出现两个JS不兼容的现象。可能就显示其中一个 JS，而另一个就被忽略了，也就是同一个网页中不能出现两个window.onload。这次本人在编写网页的时候就遇到了这个问题，因为如果只用一个 JS就显得有点单调，如果重新编写JS代码又感觉太麻烦。这时我们就要用window.attachEvent和 window.addEventListener来解决问题了。
当某一事件被触发时需要执行某个函数，在IE下可用**attachEvent**，在 FF下则要用**addEventListener**。
attachEvent()有两个参数，第一个是事件名称，第二个是需执行的函数；


addEventListener() 有三个参数，第一个是事件名称，但与IE事件不同的是，事件不带”on”,比如”onsubmit”在这里应为”submit”，第二个是需执行的函数， 第三个参数为布尔值；
例如：(可以在IE和FF下分别测试)：

var isIE = (document.all && window.ActiveXObject && !window.opera) ? true : false;
if(isIE)
{
document.getElementById(’ie’).attachEvent(“onclick”, Fun);
}
else
{
document.getElementById(’ff’).addEventListener(“click”, Fun, false);
}
function Fun()
{
if(isIE)
{
alert(’I\’m IE’);
}
else
{
alert(’I\’m Not IE’);
}

}

所以我们可以直接这样编写：
if (document.all){
window.attachEvent(’onload’, 调用函数名)//对于IE
}
else{
window.addEventListener(’load’,调用函数 名,false);//对于FireFox
}