---
title: IIS启用gzip的方法,IIS如何开启gzip
author: admin
type: post
date: 2012-06-20T06:39:31+00:00
url: /archives/13093
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - deflate
 - gzip
 - iis

---
现代的浏览器IE6和Firefox都支持客户端Gzip，也就是说，在服务器上的网页，传输之前，先使用Gzip压缩再传输给客户端，客户端接收 之后由浏览器解压显示，这样虽然稍微占用了一些服务器和客户端的CPU，但是换来的是更高的带宽利用率。对于纯文本来讲，压缩率是相当可观的。如果每个用 户节约50%的带宽，那么你租用来的那点带宽就可以服务多一倍的客户了。

IIS6已经内建了Gzip压缩的支持，可惜，没有设置更好的管理界面。所以要打开这个选项，还要费些功夫。

首先，如果你需要压缩静态文件(HTML)，需要在硬盘上建一个目录，并给它“IUSR_机器名”这个用户的写权限。如果压缩动态文件 (PHP，asp，aspx)就不需要了，因为它的页面是每次都动态生成的，压缩完就放弃。然后在IIS管理器中，“网站”上面右键-属性，不是下面的某 个站点，而是整个网站。进入“服务”标签，选上启用动态内容压缩，静态内容压缩。

然后选中网站下面那个服务器扩展，新建一个服务器扩展。名字无所谓，下面的添加文件的路径是：

c:\windows\system32\inetsrv\gzip.dll，然后启用这个扩展。

这时候静态内容是可以压缩的，但是对于动态内容，aspx文件却不在压缩范围内。因为默认的可压缩文件并没有这个扩展名。而管理界面中你又找不到可以增加扩展名的地方，这时候只能去修改它的配置文件了。

在 c:\windows\system32\inetsrv\下面有个MetaBase.xml文件，可以用记事本打开，找到 IIsCompressionScheme，有三个相同名字的段，分别是deflate,gzip,Parameters，第三段不用管它，前两段有基本 相同的参数，在这两段的参数 HcScriptFileExtensions下面都加上一行aspx，如果你有其它的动态程序要压缩，也加在这里。 HcDynamicCompressionLevel 改成**9**，(0-10，9是性价比最高的一个)。

**1.首先备份 IIS 的配置文件，**

复制　C:\Windows\system32\inetsrv\metabase.xml　到另外的备份文件夹中.

C:\Windows\system32\inetsrv\metabase.xml　是 IIS 的核心配置文件，该文件的完整性一但被破坏，IIS 将无法正常运行，严重到需要重新安装系统.

2. 在开始菜单中启动 Internet 信息服务(IIS)管理器，右键点击“网站”属性，打开“服务”选项卡，勾选“HTTP 压缩”的两个选项。“临时目录”和“临时目录最大容量”可根据需要自行设置。设置完成后点击确定。

[![](http://blog.haohtml.com/wp-content/uploads/2012/06/iis6_http_gzip.jpg)][1]

3. 右键点击“网站”下方的 “Web服务扩展”，添加一个新的Web服务扩展，扩展名填写为“**HTTP Compression**”或其他，都可以。“要求的文件”添加：c:\windows\system32\inetsrv\gzip.dll ，并勾选“设置扩展状态为允许”，完成后点击确定。

[![](http://blog.haohtml.com/wp-content/uploads/2012/06/iis6_gzip_dll.jpg)][2]

4.下面的步骤有些复杂，如果没有确定的把握能理解，最好不要尝试，右键点击“Internet 信息服务的”“本地计算机”属性，勾选“允许直接编辑配置数据库”并确定。

5. 在开始菜单中运行 notepad C:\Windows\system32\inetsrv\metabase.xml ，打开metabase.xml 文件，请在任何改动前再次确认该文件已经备份。

6. 搜索并找到 metabase.xml 文件中的

```
<IIsCompressionScheme    Location ="/LM/W3SVC/Filters/Compression/deflate"
HcCompressionDll="%windir%\system32\inetsrv\gzip.dll"
HcCreateFlags="0"
HcDoDynamicCompression="TRUE"
HcDoOnDemandCompression="TRUE"
HcDoStaticCompression="FALSE"
HcDynamicCompressionLevel="9"
HcFileExtensions="htm
html
js
css
txt"
HcOnDemandCompLevel="10"
HcPriority="1"
HcScriptFileExtensions="asp
aspx
asmx
dll
exe"
>
</IIsCompressionScheme>
<IIsCompressionScheme    Location ="/LM/W3SVC/Filters/Compression/gzip"
HcCompressionDll="%windir%\system32\inetsrv\gzip.dll"
HcCreateFlags="1"
HcDoDynamicCompression="TRUE"
HcDoOnDemandCompression="TRUE"
HcDoStaticCompression="TRUE"
HcDynamicCompressionLevel="9"
HcFileExtensions="htm
html
js
css
txt"
HcOnDemandCompLevel="10"
HcPriority="1"
HcScriptFileExtensions="asp
aspx
asmx
dll
exe"
>
</IIsCompressionScheme>
```

注意“**Compression/deflate**”和“**Compression/gzip**”两个片段都需要修改。动态压缩等级，HcDynamicCompressionLevel　建议设置为“**9**”

7. 保存并关闭 metabase.xml 文件。

8. 重新启动 IIS 服务，运行“IISReset”或重新启动 WWW 服务。

 [1]: http://blog.haohtml.com/wp-content/uploads/2012/06/iis6_http_gzip.jpg
 [2]: http://blog.haohtml.com/wp-content/uploads/2012/06/iis6_gzip_dll.jpg