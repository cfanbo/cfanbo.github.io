---
title: mysql utf8mb4与emoji表情
author: admin
type: post
date: 2016-02-24T06:19:32+00:00
url: /archives/16709
categories:
 - MySQL
tags:
 - 编码

---

- [Emoji表情符号兼容方案](http://my.oschina.net/wingyiu/blog/153357#OSC_h3_1)
- [How to support full Unicode in MySQL databases](http://my.oschina.net/wingyiu/blog/153357#OSC_h1_2)

MYSQL 5.5 之前， UTF8 编码只支持1-3个字节，只支持BMP这部分的unicode编码区， BMP是从哪到哪，到 [http://en.wikipedia.org/wiki/Mapping_of_Unicode_characters](http://en.wikipedia.org/wiki/Mapping_of_Unicode_characters) 这里看，基本就是0000～FFFF这一区。 从MYSQL5.5开始，可支持4个字节UTF编码utf8mb4，一个字符最多能有4字节，所以能支持更多的字符集。

```
utf8mb4 is a superset of utf8
```

utf8mb4兼容utf8，且比utf8能表示更多的字符。

至于什么时候用，看你的做什么项目了。。

在做移动应用时，会遇到ios用户会在文本的区域输入emoji表情，如果不做一定处理，就会导致插入数据库异常。

### **Emoji表情符号兼容方案**

**一 什么是Emoji**

emoji就是表情符号；词义来自日语（えもじ，e-moji，moji在日语中的含义是字符）
表情符号现已普遍应用于手机短信和网络聊天软件。


emoji表情符号，在外国的手机短信里面已经是很流行使用的一种表情。
手机上如何使用emoji：
1.iphone、ipad系统：安装emoji free，再设置-通用-键盘-国际键盘-添加新的键盘，然后把emoji添加在里面即可在发短信和一些输入文本的文本框中输入表情。
IOS 5用户可直接从通用中添加emoji 键盘，无需再安装emoji free
2.android系统：安装“GO输入法国际版”后，在输入法里面点选安装emoji插件可以使用。另外“百度输入法”也自带emoji表情
3.Windows Phone : 安装此 Emoji Keys，在其中输入之后复制粘贴到需要输入表情的地方即可
<此段摘自百度百科 [http://baike.baidu.com/view/2631589.htm](http://baike.baidu.com/view/2631589.htm) >

[![](http://static.oschina.net/uploads/img/201308/17162057_g5HE.png)](http://static.oschina.net/uploads/img/201308/17162057_g5HE.png)

**二 Emoji表情符号问题**
1 问题：
IOS版本之间发送的Emoji表情符号不兼容，只看到方块
不同IOS版本在数据库存数据时，有时会发生系统错误
2 现象：
IOS 4 输入Emoji表情符，在IOS5.01 显示正常，在IOS5.1中（大陆版）显现为方块， 但IOS5.01/5.1输入的表情符号，显示正      常
IOS5.01/5.1 输入表情符，在IOS5.01/5.1中显示正常，但在IOS4.X显示为方块
输入Emoji入帖子正文， 可正常存储。 但用户昵称在IOS4.X 输入Emoji，系统正常， 而IOS5.01/5.1则提示系统错误。
3 本质:
iOS 5 and OS X 10.7 (Lion) use the Unicode 6.0 standard ‘unified’ code points for emoji.
iOS 5 Emoji  采用Unicode 6 标准来统一code points

iOS 4 on SoftBank iPhones used a set of unofficial code points in the Unicode Private Use Area, and so aren’t      compatible with any other systems
iOS 4 采用SoftBank Unicode， 一种非官方的, 采用私有Unicode 区域。
4 举例:
[![](http://static.oschina.net/uploads/img/201308/17162057_3PCy.png)](http://static.oschina.net/uploads/img/201308/17162057_3PCy.png) one emoji symbol “tiger”, it is “\U0001f42f” in iOS5, but “\ue050” in earlier iOS version
虎脸Emoji符号在iOS5 为Unicode：\U0001f42f；而在IOS4.x 为：\ue050 (SoftBank 编码)
另外： 按理讲， 从iOS5 应该兼容以前版本的emoji, 但现在出现5.01版本完美兼容（无论大陆版，美版，还是港版）， 而5.1     大陆版出现了不兼容现象（腾讯微信也出现了同样的问题）。
**三 问题分析**
1 系统存储错误问题（如昵称，帖子内容）
原因：
由于IOS5.X 采用新的Unicode， 其UTF8 编码大多为4个字节， 而由于昵称/帖子内容column并没设成utf8mb4，因此存储会    发生错误。
解决方法：
将昵称/帖子内容设成utf8mb4
2 不同iOS 之间Emoji 不兼容的问题。
原因：
iOS 5 到4 不兼容的问题，很简单，unicode6 和softbank编码的不同
iOS 4 到 5，按理说应该兼容，也就是说，iOS应该自动判断如果是softbank编码，自动转成unicode6。但现在看来， iOS5.1（大陆版）好像只支持unicode6, 而不支持softbank.
解决方法：
客户端发送emoji-encoding: Softbank或unicode6, 由服务端分别给出相应的编码表。
**四 解决方案**
1 数据存储(MySQL varchar  数据类型对UTF8 支持问题)
MYSQL 5.5 之前， UTF8 编码只支持1-3个字节， 从MYSQL5.5开始，可支持4个字节UTF编码，但要特殊标记。例如我们的帖子内容项，我们加上了这个支持。服务端mysql统一存储为ios5.x也就是Unicode编码。
对应alter语句:

```
ALTER TABLE topic MODIFY COLUMN content varchar(500) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '内容';

```

2 编码转换:
iphone手机方案
客户端输入内容时候，统一存储为unicode编码(这里需要从softbank编码转换为unicode编码)。客户端请求内容的时候，需要根据不同的客户端给出不同的编码,ios4采用softbank编码做替换,ios5采用unicode编码直接支持。
android或wp其他手机方案:
如果没有emoji表情库，将无法输入。针对输入问题,将统一采用unicode编码存储。客户端请求内容的时候，将统一用softbank编码，客户端需要把emoji表情符号内置到客户端，做对应的编码和img替换。
web解决方案:
参考android或wp其他手机方案
**五 部分代码**
1 sql代码

```
CREATE TABLE `ios_emoji` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '自增ID',
  `unicode` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT 'Unicode编码',
  `utf8` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT 'UTF8编码',
  `utf16` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT 'UTF16编码',
  `sbunicode` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT 'SBUnicode编码',
  `filename` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '文件名',
  `filebyte` longblob COMMENT '文件内容字节',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=COMPACT COMMENT='ios

```