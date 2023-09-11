---
title: HTTP 1.1 中Transfer-Encoding chunked编码
author: admin
type: post
date: 2010-07-24T14:42:28+00:00
url: /archives/4777
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - 网页优化

---
大多数的站点相应用户请求时发送的HTTP Headers中包含Content-Length头.此头信息定义在HTTP1.0协议 [_RFC_ 1945](http://www.ietf.org/rfc/rfc1945.txt) 10.4章节中.该信息是用来告知用户代理,通常意义上就是浏览器,服务端发送的文档内容长度.浏览器接受到此信息后,接收完Content-Length中定义的长度字节后开始解析页面.如果服务端有部分数据延迟发送,那么浏览器就会白屏.这样导致比较糟糕的用户体验.

解决方法在HTTP1.1协议. [_RFC2616_](http://www.ietf.org/rfc/rfc2616.txt) 中14.41章节中定义的Transfer-Encoding:chunked的头信息.chunked编码定义在3.6.1中.根据此定义浏览器不需要等到内容字节全部下载完成,只要接收到一个chunked块就可解析页面.并且可以下载html中定义的页面内容,包括js,css,image等.采用chunked编码有两种选择,一种是设定Server的IO buffer长度让Server自动flush buffer中的内容，另一种是手动调用IO中的flush函数。不同的语言IO中都有flush功能:

 * php:    ob_flush(); flush();
 * perl:   STDOUT->autoflush(1);
 * java:  out.flush();
 * python:  sys.stdout.flush()
 * ruby:  stdout.flush

从下面两张图中可以看出采用HTTP1.1的Transfer-Encoding:chunked,并且把IO的buffer flush下来,以便浏览器更早的下载页面配套资源.

==============================================

正如上篇日志所述，当不能预先确定报文体的长度时，不可能在头中包含Content-Length域来指明报文体长度，此时就需要通过Transfer-Encoding域来确定报文体长度。

通常情况下，Transfer-Encoding域的值应当为chunked,表明采用chunked编码方式来进行报文体的传输。chunked编码是HTTP/1.1 RFC里定义的一种编码方式，因此所有的HTTP/1.1应用都应当支持此方式。

chunked编码的基本方法是将大块数据分解成多块小数据，每块都可以自指定长度，其具体格式如下（BNF文法）:

Chunked-Body   = *chunk            //0至多个chunk

last-chunk         //最后一个chunk

trailer            //尾部

CRLF               //结束标记符

chunk          = chunk-size [ chunk-extension ] CRLF

chunk-data CRLF

chunk-size     = 1*HEX

last-chunk     = 1*(“0”) [ chunk-extension ] CRLF

chunk-extension= *( “;” chunk-ext-name [ “=” chunk-ext-val ] )

chunk-ext-name = token

chunk-ext-val  = token | quoted-string

chunk-data     = chunk-size(OCTET)

trailer        = *(entity-header CRLF)

解释：

Chunked-Body表示经过chunked编码后的报文体。报文体可以分为chunk, last-chunk，trailer和结束符四部分。chunk的数量在报文体中最少可以为0，无上限；每个chunk的长度是自指定的，即，起始的数据必然是16进制数字的字符串，代表后面chunk-data的长度（字节数）。这个16进制的字符串第一个字符如果是“0”，则表示chunk-size为0，该chunk为last-chunk,无chunk-data部分。可选的chunk-extension由通信双方自行确定，如果接收者不理解它的意义，可以忽略。

trailer是附加的在尾部的额外头域，通常包含一些元数据（metadata, meta means “about information”），这些头域可以在解码后附加在现有头域之后。

实例分析：

下面分析用ethereal抓包使用Firefox与某网站通信的结果（从头域结束符后开始）：

Address  0……………………..  f

000c0                                31

000d0    66 66 63 0d 0a ……………   // ASCII码:1ffc\r\n, chunk-data数据起始地址为000d5

很明显，“1ffc”为第一个chunk的chunk-size,转换为int为8188.由于1ffc后马上就是

CRLF,因此没有chunk-extension.chunk-data的起始地址为000d5, 计算可知下一块chunk的起始

地址为000d5+1ffc + 2=020d3,如下：

020d0    .. 0d 0a 31 66 66 63 0d 0a …. // ASCII码:\r\n1ffc\r\n

前一个0d0a是上一个chunk的结束标记符，后一个0d0a则是chunk-size和chunk-data的分隔符。

此块chunk的长度同样为8188, 依次类推，直到最后一块

100e0                          0d 0a 31

100f0    65 61 39 0d 0a……            //ASII码：\r\n\1ea9\r\n

此块长度为0x1ea9 = 7849, 下一块起始为100f5 + 1ea9 + 2 = 11fa0,如下：

100a0    30 0d 0a 0d 0a                  //ASCII码：0\r\n\r\n

“0”说明当前chunk为last-chunk, 第一个0d 0a为chunk结束符。第二个0d0a说明没有trailer部分，整个Chunk-body结束。

解码流程：

对chunked编码进行解码的目的是将分块的chunk-data整合恢复成一块作为报文体，同时记录此块体的长度。

RFC2616中附带的解码流程如下：(伪代码）

length := 0         //长度计数器置0

read chunk-size, chunk-extension (if any) and CRLF      //读取chunk-size, chunk-extension

//和CRLF

while(chunk-size > 0 )   {            //表明不是last-chunk

read chunk-data and CRLF            //读chunk-size大小的chunk-data,skip CRLF

append chunk-data to entity-body     //将此块chunk-data追加到entity-body后

read chunk-size and CRLF          //读取新chunk的chunk-size 和 CRLF

}

read entity-header      //entity-header的格式为name:valueCRLF,如果为空即只有CRLF

while （entity-header not empty)   //即，不是只有CRLF的空行

{

append entity-header to existing header fields

read entity-header

}

Content-Length:=length      //将整个解码流程结束后计算得到的新报文体length

//作为Content-Length域的值写入报文中

Remove “chunked” from Transfer-Encoding  //同时从Transfer-Encoding中域值去除chunked这个标记

length最后的值实际为所有chunk的chunk-size之和，在上面的抓包实例中，一共有八块chunk-size为0x1ffc(8188)的chunk,剩下一块为0x1ea9(7849),加起来一共73353字节。

注：对于上面例子中前几个chunk的大小都是8188,可能是因为:”1ffc” 4字节，”\r\n”2字节，加上块尾一个”\r\n”2字节一共8字节，因此一个chunk整体为8196,正好可能是发送端一次TCP发送的缓存大小。

正如上篇日志所述，当不能预先确定报文体的长度时，不可能在头中包含Content-Length域来指明报文体长度，此时就需要通过Transfer-Encoding域来确定报文体长度。    通常情况下，Transfer-Encoding域的值应当为chunked,表明采用chunked编码方式来进行报文体的传输。chunked编码是HTTP/1.1 RFC里定义的一种编码方式，因此所有的HTTP/1.1应用都应当支持此方式。    chunked编码的基本方法是将大块数据分解成多块小数据，每块都可以自指定长度，其具体格式如下（BNF文法）:    Chunked-Body   = *chunk            //0至多个chunk                     last-chunk         //最后一个chunk                      trailer            //尾部                     CRLF               //结束标记符
chunk          = chunk-size [ chunk-extension ] CRLF                           chunk-data CRLF   chunk-size     = 1\*HEX   last-chunk     = 1\*(“0”) [ chunk-extension ] CRLF
chunk-extension= \*( “;” chunk-ext-name [ “=” chunk-ext-val ] )   chunk-ext-name = token   chunk-ext-val  = token | quoted-string   chunk-data     = chunk-size(OCTET)   trailer        = \*(entity-header CRLF)              解释：    Chunked-Body表示经过chunked编码后的报文体。报文体可以分为chunk, last-chunk，trailer和结束符四部分。chunk的数量在报文体中最少可以为0，无上限；每个chunk的长度是自指定的，即，起始的数据必然是16进制数字的字符串，代表后面chunk-data的长度（字节数）。这个16进制的字符串第一个字符如果是“0”，则表示chunk-size为0，该chunk为last-chunk,无chunk-data部分。可选的chunk-extension由通信双方自行确定，如果接收者不理解它的意义，可以忽略。    trailer是附加的在尾部的额外头域，通常包含一些元数据（metadata, meta means “about information”），这些头域可以在解码后附加在现有头域之后。    实例分析：    下面分析用ethereal抓包使用Firefox与某网站通信的结果（从头域结束符后开始）：Address  0……………………..  f000c0                                31000d0    66 66 63 0d 0a ……………   // ASCII码:1ffc\r\n, chunk-data数据起始地址为000d5         很明显，“1ffc”为第一个chunk的chunk-size,转换为int为8188.由于1ffc后马上就是         CRLF,因此没有chunk-extension.chunk-data的起始地址为000d5, 计算可知下一块chunk的起始         地址为000d5+1ffc + 2=020d3,如下：020d0    .. 0d 0a 31 66 66 63 0d 0a …. // ASCII码:\r\n1ffc\r\n         前一个0d0a是上一个chunk的结束标记符，后一个0d0a则是chunk-size和chunk-data的分隔符。         此块chunk的长度同样为8188, 依次类推，直到最后一块100e0                          0d 0a 31100f0    65 61 39 0d 0a……            //ASII码：\r\n\1ea9\r\n         此块长度为0x1ea9 = 7849, 下一块起始为100f5 + 1ea9 + 2 = 11fa0,如下：100a0    30 0d 0a 0d 0a                  //ASCII码：0\r\n\r\n         “0”说明当前chunk为last-chunk, 第一个0d 0a为chunk结束符。第二个0d0a说明没有trailer部分，整个Chunk-body结束。    解码流程：    对chunked编码进行解码的目的是将分块的chunk-data整合恢复成一块作为报文体，同时记录此块体的长度。    RFC2616中附带的解码流程如下：(伪代码）    length := 0         //长度计数器置0    read chunk-size, chunk-extension (if any) and CRLF      //读取chunk-size, chunk-extension                                                          //和CRLF    while(chunk-size > 0 )   {            //表明不是last-chunk          read chunk-data and CRLF            //读chunk-size大小的chunk-data,skip CRLF          append chunk-data to entity-body     //将此块chunk-data追加到entity-body后          read chunk-size and CRLF          //读取新chunk的chunk-size 和 CRLF    }    read entity-header      //entity-header的格式为name:valueCRLF,如果为空即只有CRLF    while （entity-header not empty)   //即，不是只有CRLF的空行    {       append entity-header to existing header fields       read entity-header    }    Content-Length:=length      //将整个解码流程结束后计算得到的新报文体length                                 //作为Content-Length域的值写入报文中    Remove “chunked” from Transfer-Encoding  //同时从Transfer-Encoding中域值去除chunked这个标记    length最后的值实际为所有chunk的chunk-size之和，在上面的抓包实例中，一共有八块chunk-size为0x1ffc(8188)的chunk,剩下一块为0x1ea9(7849),加起来一共73353字节。    注：对于上面例子中前几个chunk的大小都是8188,可能是因为:”1ffc” 4字节，”\r\n”2字节，加上块尾一个”\r\n”2字节一共8字节，因此一个chunk整体为8196,正好可能是发送端一次TCP发送的缓存大小。