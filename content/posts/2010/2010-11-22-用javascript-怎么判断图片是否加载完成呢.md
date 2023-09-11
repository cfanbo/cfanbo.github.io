---
title: 用javascript 怎么判断图片是否加载完成呢
author: admin
type: post
date: 2010-11-21T21:14:26+00:00
url: /archives/6746
IM_contentdowned:
 - 1
categories:
 - 前端设计

---

用javascript 怎么判断图片是否加载完成呢?


> function loadImage(url){
>
> var o= new Image();
>
> o.src = url;
>
> if(o.complete){
>
>
> window.alert(‘图片加载完成:’+url);
>
>
> }else{
>
> o.onload = function(){
>
>
> window.alert(‘图片加载完成:’+url);
>
>
> };
>
> o.onerror = function(){
>
>
> window.alert(‘图片加载失败:’+url);
>
>
> };
>
> }
>
> }

如果我要先把这一个图片加载完,之后才显示怎么处理呢.

```

  function showImage(url){

  var o = document.createElement('img');

  o.src = url;

  document.body.appendChild(o);

  }

  function loadImage(url){
   var o= new Image();
   o.src = url;
   if(o.complete){

  showImage(url);

  }else{
   o.onload = function(){

  showImage(url);

  };
   o.onerror = function(){

  window.alert('图片加载失败:'+url);

  };
   }
   }


```