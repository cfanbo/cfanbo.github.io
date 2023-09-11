---
title: 使用apache的rewrite功能来防迅雷
author: admin
type: post
date: 2011-03-28T10:10:05+00:00
url: /archives/8196
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - 迅雷
 - rewrite

---
不知道为什么，本来不受重视的L’Yun，却一直多灾多难，前几天空间呗停掉了，一个很以为的原因，每天将近9G的流量，晕死了，最多的一天才只有6个IP，但竟然有这么大的流量。后来查看了下日志，竟然是两首MP3引起的，每一秒钟都有人在下载。刚开始以为是百度干的，但是后来看了下在百度的位置，还不至于达到那么大的流量，然后自然而然的就想到迅雷了，看看别人的文章，可以肯定下，迅雷是个流氓！

**解决方案：**
1、对服务器的攻击屏蔽后，不用理会，不会造成太大影响。
2、被百度收录的是一部分MP3，因为不希望不访问网站就直接从后台下载网站的mp3，于是增加搜索引擎访问限制。在网站根目录下放置robots.txt，内容如下：
User-agent: Baiduspider
Disallow: /\****
*表示不允许百度搜索引擎收录的路径。相对于百度，雅虎、MSN和Google的搜索引擎机器人没有那么流氓，所以不需要屏蔽。


3、对付迅雷。
相对于有些流氓的百度搜索引擎来说，迅雷就是恶霸了。
对于小网站站长来说，迅雷的分布式下载几乎是一种灾难。尽管迅雷给广大普通用户带来快捷方便，但给小服务器的负载带来严重灾难。
调用access日志，发现瞬间连接超过1000，而连接的集中点，居然是周董的一首《七里香》。尽管迅雷隐蔽的很好，但还是从日志的蛛丝马迹里找出它的影子。
于是先删掉七里香。删掉后仍有大量链接寻找其他MP3，而且删除一首mp3也只是治标不治本。启用Apache2的Rewrite模块。
在Apache的Http.conf中，开启Rewrite模块

> LoadModule rewrite\_module modules/mod\_rewrite.so

然后增加以下Rewirte规则

> RewriteEngine On
> RewriteCond %{HTTP_REFERER} !^http://www.cfobbs.com/.*$ [NC]
> RewriteCond %{HTTP_REFERER} !^http://www.cfobbs.com$ [NC]
> RewriteRule .*\.(mp3|rm|wma)$ http://www.cfobbs.com/error.html [R,NC]

该规则表示，只有浏览器的REFERER是本站开头的连接，才可以下载MP3、rm、wma，否则转向error.html错误界面。
重启Apache后，用Flashget测试，无法下载mp3了，IE直接下载也会报错，但迅雷仍然可以下，百思不得其解，于是查阅迅雷官方资料，居然发现迅雷采用了一个十分流氓的手段：伪造下载地址的浏览器REFERER头，真是无耻。
考虑到网站的MP3全部是在网页自动播放，基本不需要额外下载，于是为了对付迅雷，采用了一个比较极端的方式：

> RewriteEngine On
> RewriteCond %{HTTP\_USER\_AGENT} !^NSPlayer.*
> RewriteCond %{HTTP\_USER\_AGENT} !^windows.*
> RewriteRule .*\.(mp3|rm|wma)$ http://www.cfobbs.com/error.html [R,NC]

也就是说，网站上mp3、rm、wma格式的文件，只允许播放器播放，不允许任何其它方式的访问，否则就转向错误页面。
重启Apache，使用迅雷下载，结果迅雷直接去下载了错误页面，初战告捷。
调用access日志，发现所有的mp3下载都提示302，转向了错误页面，而周董的七里香，则是404，直接报错。
自鸣得意一把。
顺便找到一个限速模块，对网站体积较大的文件进行限速，确保服务器稳定

> LoadModule limitipconn\_module modules/mod\_limitipconn.so
> BandwidthModule On
> ForceBandWidthModule On
> Bandwidth all 0
> MinBandwidth all 0
> LargeFileLimit *.mp3 500 50000
> LargeFileLimit *.wma 500 50000

该模块可以指定文件名、文件大小限速，上面的意思是，MP3和WMA文件，凡是大小超过500K的，限速50K，大小不超过500K的，不予限速。
启用该模块，必须先行启用status模块

> LoadModule status\_module modules/mod\_status.so
> ExtendedStatus On

否则限速模块无效。

**相关文章:**

[windows平台下apache完美限制下载速度][1]

[Apache带宽流量控制模块安装 mod_bw 配置说明](http://blog.haohtml.com/archives/8205)

 [1]: http://blog.haohtml.com/archives/8187