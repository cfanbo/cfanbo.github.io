---
title: CrossDomain.xml 作用
author: admin
type: post
date: 2010-05-21T04:51:01+00:00
url: /archives/3668
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - 跨域

---
使用crossdomain.xml让Flash可以跨域传输数据

本文来自[http://www.mzwu.com/article.asp?id=975][1]

**一、 概述**

****位于www.mzwu.com域中的SWF文件要访问www.163.com的文件时，SWF首先会检查163服 务器目录下是否有crossdomain.xml文件，如果没有，则访问不成功；若crossdomain.xml文件存在，且里边设置了允许 www.mzwu.com域访问，那么通信正常。所以要使 [**Flash**](javascript:;) 可以跨域传输数据，其关键就是 crossdomain.xml。

**二、crossdomain.xml文件格式**

crossdomain.xml 的格式非常简单，其根节点为 ，其下包含一个或多个节点，有一个属性domain，其值为允许访问的域，可以是确切的 IP 地址、一 个确切的域或一个通配符域（任何域）。下边是两个例子：

程序代码


程序代码


第 二个例子允许任何域的访问。对于crossdomain.xml文件存放位置，建议将其存放于站点根目录中！

**三、示例**

1.SWF 文件主要Actionscript：

程 序代码


on (release) {

var myvar = new LoadVars();

myvar.t = t2.text;

myvar.sendAndLoad(“ [http://www.163.com/test.asp](http://www.163.com/test.asp)“,myvar,”post”);

myvar.onLoad = function(re){

if(re){

t1.text = myvar.t;

}else{

t1.text = “fail…”;

}

}

}


2.test.asp 代码：

程序代码

<%

Dim t

t = Request.form(“t”)

Response.write(“t=” & t & ” back!”)

%>


 [1]: http://www.mzwu.com/article.asp?id=975