---
title: 清除缓存，php,html,jsp,asp中防止模式窗口页面不更新的情况
author: admin
type: post
date: 2008-03-10T10:07:02+00:00
excerpt: |
 清除缓存，php,html,jsp,asp中防止页面不更新的情况
 Code:

 HTML

 ASP
 <%
 Response.Expires = -1
 Response.ExpiresAbsolute = Now() - 1
 Response.cachecontrol = "no-cache"
 %>
 PHP

 header("Expires: Mon, 26 Jul 1997 05:00:00 GMT");
 header("Cache-Control: no-cache, must-revalidate");
 header("Pragma: no-cache");
url: /archives/279
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - 缓存

---
清除缓存，php,html,jsp,asp中防止页面不更新的情况
Code:

HTML

ASP

<%

Response.Expires = -1

Response.ExpiresAbsolute = Now() – 1

Response.cachecontrol = “no-cache”

%>

PHP

header(“Expires: Mon, 26 Jul 1997 05:00:00 GMT”);

header(“Cache-Control: no-cache, must-revalidate”);

header(“Pragma: no-cache”);

JSP

response.setHeader(“Pragma”,”No-Cache”);

response.setHeader(“Cache-Control”,”No-Cache”);

response.setDateHeader(“Expires”, 0);