---
title: JW Media Player 中文文档
author: admin
type: post
date: 2010-04-17T15:12:46+00:00
url: /archives/3427
IM_data:
 - 'a:1:{s:38:"http://onewww.net/blog/images/code.gif";s:38:"http://onewww.net/blog/images/code.gif";}'
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - JW Player

---
**The JW MP3 Player (built with Adobe’s Flash) is the easiest way to add live music or podcasts to your website. It supports playback of a single MP3 file or an RSS, XSPF or ASX playlist, a wide range of flashvars (settings) for tweaking both behavior and appearance and an extensive, documented javascript/actionscript API.**

官方网址 [http://www.jeroenwijering.com](http://www.jeroenwijering.com/?item=JW_Media_Player)

这篇文档包含了JW MP3 Player,JW FLVPlayer,JW Media Player,JW Image Rotator的安装方法和配置方
法

JW PLAYERS 3.11 文档

1，安装 （在你的站点里嵌入）
2，参数 （配置）
基本参数
颜色参数
界面参数
播放参数
互动参数


3，播放列表
4，自定义
5，支持

快速配置，可运行配置向导

1，安装
下载后，你可以得到一个例子，当用文本或HTML编辑器打开的时候，你可以发现swf是用一段短小的
javascript嵌入到页面上的。这个Javascript是Geoff Stearns写的swfobject.js，它解决了Flash需要激
活的麻烦。当复制swf到你的站点的时候，不要忘记了把swfobject.js一同复制过去。并且在页面中的
Head中加入下面代码

![程序代码](http://onewww.net/blog/images/code.gif)程序代码

该文章转载自脚本之家：http://www.jb51.net/html/200711/19/12959.htm


在swfobject.js的帮助下你可以方便的设置swf的路径，宽度，高度，背景颜色和所需要的Flash播放器的版本，也可以通过它设置一组参数来 配置swf。当页面加载的时候，javascript会根据id来替换页面中的节点。

![程序代码](http://onewww.net/blog/images/code.gif) 程序代码


[Get Flash](”http://www.macromedia.com/go/getflashplayer”) to see this player.

在你不能使用javascript的情况下（比如你的bsp不支持，如myspace），你可以使用”embed”的方式来插入swf。
需要注意的是 swf可以不在该服务器上，但是如果你需要用xml的播放列表可能会有问题，因为xml必须和swf在同一个服务器上，否则会因为安全问题被组织。 mp3，jpg，flv可以不在该服务器上。

![程序代码](http://onewww.net/blog/images/code.gif) 程序代码


type=”application/x-shockwave-flash” pluginspage=”http://www.macromedia.com/go/getflashplayer”

flashvars=”file=http://www.myfileserver.com/folder/test.flv&displayheight=300″ />


2，参数
这些参数可以配置被嵌入到html中的播放器的行为和外观。如果使用swfobject.js，可以用addVariable()方法来配置，如果使用的 embed方式，可以通过定义”flashvars”属性来配置变量，注意用”&”符号分割这些变量。

被*号标注的同样适用于 JW Image Rotator

基本变量

displayheight(number）：显示区域的高度.控制区域最小高度是20px，当该值大于或等于swf的高度时，播放列表会自动隐藏，否这 会显示。
file*(url）：要播放文件的地址。 支持单文件播放(MP3/FLV/RTMP/JPG/SWF/PNG/GIF)，也支持播放列表。JW Image Rotator只支持列表
height*(number）：设置swf的高度，当使用embed方式插入的时候，在embed的属性里已经设置了。但是有时候（特别是使用IE的时 候）高度会变的不确定，影响了布局，这时候需要通过该参数设置下，该值的单位是px
image(url）:当你播放mp3，flv的时候，你可以通过该值设置一个预览图作为专辑封面，支持 JPG/SWF/PNG/GIF file. 你也可以在播放列表中对每一项进行设置。
shownavigation*（true,false):该参数仅被JW Image Rotator支持。用来显示或隐藏图片导航。
transition* (fade,bgfade,blocks,bubbles,circles,fluids,lines,random,slowfade): 该参数仅被JW Image Rotator支持。用来设置图片替换的效果。 设置为”random” 将随机的设置效果.默认值为 “fade”.
width*(number):设置swf的宽度，其他同height；

颜色变量

backcolor*(color):设置播放器的背景颜色。JW Media Player等默认为 0xFFFFFF (白色) JW Image Rotator默认为   0x000000 (黑色).
fontcolor*(color):设置文字和按钮的颜色。JW Media Player等默认为 0x000000 (黑色) JW Image Rotator默认为   0xFFFFFF (白色).
lightcolor*(color):设置被被激活状态的颜色。JW Media Player等默认为 0x000000 (黑色) JW Image Rotator默认为   0xCC0000 (红色).

界面参数

autoscroll(true,false):当播放列表过长的时候，默认会自动显示滚动条。当该值设置为’true’的时候，会自动根据鼠标滚动播放 列表。
displaywidth(number of pixes):设置显示区域的宽度，当设置的比较小的时候，播放列表会显示在显示区域的右侧而不是底部。
kenburns* (true,false): 用以实现在运动的时候实现kb效果（Ken Burns effect），注意，当图比较大，而且网速比较快的时候，建议打开，否则关闭。建议设置transition为”slowfade”来配合。
largecontrols (true,false): 设置该值为true用来放大控制区域的按钮。建议为视力不好的用户打开
logo* (url): 设置一个图片用来作为右上角的水印，支持所有图片格式，支持通明图层的png效果最佳。
overstretch* (true,false,fit,none): 设置图片/影片在显示区域的缩放。”true”等比例拉伸用来符合显示区域。”false”仅拉伸合显示区域。”fit”全屏显示。”none”显示原始 大小。JW Media Player等默认为”fit”,JW Image Rotator默认为”false”
showdigits (true,false,total): 设置为”false”隐藏播放时间等信息用来节省控制区域的空间。设置为”total”用来显示全部时间。
showdownload (true,false):设置该值用来在控制区域显示下载按钮。链接到link所设置的地址。
showeq (true,false): 用来显示一个假的音频波动效果。当播放mp3的时候打设置该值可以得到很好的效果
showicons* (true,false): 用来显示或者隐藏显示区域中间的图片，JW Media Player等默认为true。JW Image Rotator默认为false；
showvolume (true,false): 用来设置是否显示音量控制按钮
thumbsinplaylist (true,false): 设置列表中是否显示预览图

播放参数
autostart (true,false,muted): 设置为ture，页面加载完后会自动播放。设置为muted，会在静音模式下自动播放，并且显示区域中间有静音图标。
bufferlength (number): 设置flv的缓存时间。默认为3秒
repeat* (true,false,list): 默认为flase，从当前播放位置播放到列表尾部后停止。设置为list会播放列表中所有的项目，设置为true会循环播放。
rotatetime* (number): 设置图片的显示时间。JW Media Player等默认为10秒,JW Image Rotator默认为5秒
shuffle* (true,false): 设置为false顺序播放，设置为true无序播
smoothing (true,false): 设置为false关闭视频平滑处理，推荐设置true用以得到更好效果。但对于大屏幕或者配置低的机器设置false是有好处的
start* (second): 在使用RTMP 或 HTTP 流媒体的时候（非常规的flv/mp3)，使用该变量准确的定位开始位置。该参数设置在XSPE格式的列表中以便准且的设置文件的章节。
volume* (number): 设置音量，默认为80.

互动参数

audio* (url):用这个参数来添加一个mp3文件作为单独的音频，可以作为图片的背景音乐解说等。
bwfile (url):用以带宽检测的文件的地址，可以放一个图片，或者rtmp流媒体。可以在右键菜单中查看到贷款数值。
bwstreams (comma-separated list of bitrates):
和bwfile配合使用，根据带宽值来选择不同的文件。如：你要播放video.flv并且设置该项的值为100,250,500,1000,当播放器发 现带宽为349kbps的时候，将会播放video_250.flv。所以他有一套有效的命名设置，他将会自动切换，哪怕是在采用播放列表的情况下。
callback (url):设置这个参数为服务端程序（php/asp)地址用来回传数据。在每个项播放和停止的时候会发送数据到服务器，以便在服务器端保存播放统计。
captions (url): 设置该值用以载入一个文本格式的文本作为字幕。播放器至支持SMIL格式和DVD的SRT格式的字幕。如果你的flv文件内置字体你可以设置该值 为”captionate”.如果你有多频道字幕，可以设置这个值为”captionate0″, “captionate3″等。可以在列表中设置每一个项的值。
enablejs* (true,false): 设置为true打开对javascript的支持。仅支持在线使用。javascript可以控制播放，加载媒体，获得当前播放项的详尽信息。
fsbuttonlink (url):如果用户的flashplayer版本高于（9.0.28）播放器会自动的显示一个全屏按钮。通过设置该值，你可以链接到另外的页面用以全屏 显示。服务端程可以设定将要播放的文件。
id (string): 播放器的唯一标识。将会被回传到服务器端。
javascriptid* (string):如果你的页面上有多个播放器，你可以设置这个参数给每个播放器不同的id，这样就可以方便的用javascript来控制。他将回传到 getUpdate（）事件中。
link (url): 通过这个参数用来设置一个可现在的版本，或者强制用户通过该地址下载当前项。可以在播放列表中为每一项设置该值。
linkfromdisplay* (true,false):设置显示区域被点击时要访问的页面。默认点击显示区域时会进行播放/暂停操作。
linktarget* (frame): 设置链接目标，”\_self”在当前页打开。”\_blank”在新页面中打开。
streamscript (url):设置这个参数为了兼容‘伪流媒体’FLV文件。
type (mp3,flv,rtmp,jpg,png,gif,swf,rbs,3gp,mp4,m4v): 播放器会根据文件名的最后三个字符来判断类型。在你使用服务器端语言进行重定向时，这种方法将不会再有效。所以你可以设置这个参数来告诉播放器文件类型。 你也可以在播放列表中对每一项进行设置。如果播放器找不到文件类型将会被识别为播放列表。
useaudio (true,false): 设置为false用来改变为静音状态。
usecaptions (true,false):设置为false隐藏字幕.
usefullscreen (true,false):如果你不希望用flashplayer9的全屏模式。可以设置”fsbuttonlink”参数用来替换全屏按钮的事件。
usekeys (true,false): 设置为”false”用来取消键盘操作(SPACE,UP,DOWN,LEFT,RIGHT)

注意：你必须对?=&进行字符转移。? → %3F, = → %3D, & → %26. 你的”file” 变量为getplaylist.php?id=123的时候必须设置为 getplaylist.php%3Fid%3D123.

播放列表

当播放列表中只有一个项目的时候，播放器会用单文件播放模式进行播放。播放器根据文件名来区分单文件和播放列表文件。如你的加载文件名为test.mp3 的文件，播放器会用单文件播放模式去播放mp3，因为扩展名是mp3.如果你的文件名是getlist.php,播放器会认为这是个播放列表。如果你使用 了动态脚本来加载一个单文件如

播放器支持三种常用的播放列表格式来确保通用性。XSPF，RSS和ATOM。

播放器所支持的XSPF/RSS/ATOM列表的通用标签。
表格

[![jw-media-player-1](http://blog.haohtml.com/wp-content/uploads/2010/04/jw-media-player-1.jpg)](/wp-content/uploads/2010/04/jw-media-player-1.jpg)

*Image Rotator仅支持播放列表中的第一个’audio’项，被用来做为背景音乐来播放。播放器会播放所有的音乐项目。

** “category” 元素可以用来支持广告。你可以设置该值为”preroll”, “postroll” 或 “overlay”.
使用”link”播放列表中单项的值为广告地址。

\*** “start”元素只被XSPF的播放列表格式支持。该元素用来设置流媒体文件（HTTP or RTMP)的开始位置，单位为秒。

\**** XSPF列表格式中的”location”元素中如果不包含常见的扩展名(如 “mp3” or “flv”),可以使用”meta”元素指定文件格式。(如