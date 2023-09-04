---
title: PHP中如何使用header发送头部信息
author: admin
type: post
date: 2010-03-26T06:33:54+00:00
categories:
 - 程序开发
tags:
 - php

---
在照彭武兴先生的《PHP BIBLE》中所述，header可以送出Status标头，如

就可以让用户浏览器出现文件找不到的404错误，但是我试了这样是不行的。

后 来我到w3.org上查了http的相关资料，终于试出来了如何Header出状态代码(Status)，与大家分享。

其实应该是这样 的:

第一部分为 HTTP协议的版本(HTTP-Version)

第二部分为状态代码(Status)

第三部分为原因短语 (Reason-Phrase)

三部分中间用一个空格分开，且中间不能有回车,第一部分和第二部分是必需的，第三部分则是给人看的，可 写可不写甚至乱写。

**还有，这一句的输出必须在Html文件的第一行。**

下面我给出各代码所代表的意思(是从 w3.org上查到的,够权威了):

* 1xx: Informational – Request received, continuing process

* 2xx: Success – The action was successfully received, understood,and accepted

* 3xx: Redirection – Further action must be taken in order to complete the request

* 4xx: Client Error – The request contains bad syntax or cannot be fulfilled

* 5xx: Server Error – The server failed to fulfill an apparently

**valid reques**t

“100” ; Continue

“101” ; Switching Protocols

“200” ; OK

“201” ; Created

“202” ; Accepted

“203” ; Non-Authoritative Information

“204” ; No Content

“205” ; Reset Content

“206” ; Partial Content

“300” ; Multiple Choices

“301” ; Moved Permanently

“302” ; Moved Temporarily

“303” ; See Other

“304” ; Not Modified

“305” ; Use Proxy

“400” ; Bad Request

“401” ; Unauthorized

“402” ; Payment Required

“403” ; Forbidden

“404” ; Not Found

“405” ; Method Not Allowed

“406” ; Not Acceptable

“407” ; Proxy Authentication Required

“408” ; Request Time-out

“409” ; Conflict

“410” ; Gone

“411” ; Length Required

“412” ; Precondition Failed

“413” ; Request Entity Too Large

“414” ; Request-URI Too Large

“415” ; Unsupported Media Type

“500” ; Internal Server Error

“501” ; Not Implemented

“502” ; Bad Gateway“503” ; Service Unavailable“504” ; Gateway Time-out

“505” ; HTTP Version not supported