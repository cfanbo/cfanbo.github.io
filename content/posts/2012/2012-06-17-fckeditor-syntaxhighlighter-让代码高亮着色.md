---
title: FCKeditor + SyntaxHighlighter 让代码高亮着色
author: admin
type: post
date: 2012-06-17T17:43:27+00:00
url: /archives/13089
IM_data:
 - 'a:1:{s:66:"http://tech.cncms.com/tech/UploadPic/2010-10/20101011337812117.jpg";s:77:"http://blog.haohtml.com/wp-content/uploads/2012/06/8e5f_20101011337812117.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - FCKEditor

---
FCKeditor是现在最为流行的开源编辑器，SyntaxHighlighter是一个用JS实现的代码高亮显示插件，可以最小化修改您的程序实现效果，最终效果截图：

[![](http://blog.haohtml.com/wp-content/uploads/2012/06/fckedit_syntaxhighlighter1.jpg)][1]

演示网页：

下载FCKeditor + SyntaxHighlighter插件包：[fck_SyntaxHighlighter.zip][2]

下面分步介绍如何在FCKeditor环境中使用SyntaxHighlighter。


**后台FCKeditor编辑器的修改**

1、将包解压后，把 insertcode 文件夹上传到 FCKeditor编辑器的editor\plugins\目录下，然后修改FCKeditor编辑器的fckconfig.js此文件，在此文件中 FCKConfig.PluginsPath = FCKConfig.BasePath + ‘plugins/’ ;下面加入以下代码：
FCKConfig.Plugins.Add(‘insertcode’);

2、打开FCKeditor编辑器的editor\lang文件夹里的zh-cn.js，在DlgDivInlineStyle : “CSS 样式”,（注意这句后面一定要加一个逗号“,”）下面加入以下代码
//Plugins
InsertCodeBtn : “插入代码”

3、修改FCKeditor编辑器的fckconfig.js文件，在编辑器控制面板中加入按钮，在调用工具栏参数的Media后面加入insertcode（注意正确加上标点符号，否则会报错）。如下所示：
FCKConfig.**Sets[ “standard”] = [
[‘Source’,’Paste’,’PasteText’,’PasteWord’,’-‘,’Undo’,’Redo’,’-‘,’Bold’,’Italic’,’Underline’,’StrikeThrough’,’TextColor’,’Table’,’-‘,’JustifyLeft’,’JustifyCenter’,’JustifyRight’,’OrderedList’,’UnorderedList’,’-‘,’Image’,’Attach’,’Flash’,’Media’,‘InsertCode’],完成以上操作后，此时启动FCKeditor编辑器应该在编辑器的**上多了一个图标，点击此图标即可添加你的代码了。如果报错，提示找不到工具项，那是FCKeditor的缓存没清除，退出后台或更新缓存，刷新一下，重新进入就可以看到代码插入图标了。
**前台显示页面的SyntaxHighlighter调用**

1、将包解压后把 syntax 文件夹上传到前台根目录下的js文件夹中。

2、在需要进行高亮显示处理的HTML页面中增加SyntaxHighlighter控制代码，将如下代码，插入到HTML页面的与之间：

```
<script type="text/javascript" src="/js/syntax/scripts/shCore.js"></script>
<script type="text/javascript" src="/js/syntax/scripts/shLegacy.js"></script>
<script type="text/javascript" src="/js/syntax/scripts/shBrushAll.js"></script>
<link href="/js/syntax/styles/shCore.css" type="text/css" rel="stylesheet"/>
<link href="/js/syntax/styles/shThemeDefault.css" type="text/css" rel="stylesheet"/>
<script type="text/javascript">
SyntaxHighlighter.config.clipboardSwf = '/js/syntax/scripts/clipboard.swf';
SyntaxHighlighter.all();
</script>
```

2、在前台页的CSS文件中增加如下样式控制CSS代码（这段也可以不加，只是为了更美观）：

```
.codeHead {font-weight: bold;font-size: 12px;padding: 5px;padding-left: 15px;background: #fff;border-bottom: 1px solid #ddd;}
.codeText {border: 1px solid #ddd;width: 98%;overflow: auto;margin: 0 0 1.1em;padding: 0;word-break: break-all;background: #fff;font: 12px 'Courier New', Monospace;}
.codeText ol {list-style: decimal-leading-zero;margin: 0 1px 0 45px;padding: 5px 0;color: #5C5C5C;border-left: 1px solid #ddd;background: #fff;}
.codeText ol li {list-style-type:decimal;padding-left: 10px;background: #FFF;}
.codeText ol li.alt {background: #FFF;}
.codeText ol li span {color: #000;}
```

**注意：**这样的前台调用路径为 /js/syntax/ 是因为我上传到了这个路径，此路径根据你的需要可修改。应为你实际上传的路径。
至此修改全部结束，如果你在修改中遇到什么问题，欢迎与我们交流，tech#cncms.com
**补充：**有朋友反映加载时页面会卡一下才能显示完，原因是在页面加载时JS即开始运行，进行代码的着色，解决办法就是，让代码着色 SyntaxHighlighter.all(); 延时执行即可，我们把上面的代码稍改一下：

```
<script type="text/javascript">
SyntaxHighlighter.config.clipboardSwf = '/js/syntax/scripts/clipboard.swf';
SyntaxHighlighter.all();
</script>
```

改为：

```
<script type="text/javascript">
SyntaxHighlighter.config.clipboardSwf = '/js/syntax/scripts/clipboard.swf';
setTimeout("SyntaxHighlighter.all();",300);
</script>
```

这样改后，就感觉不到加载时的卡了。
演示网页：

下载FCKeditor + SyntaxHighlighter插件包：[fck_SyntaxHighlighter.zip][2]

摘自: [http://tech.cncms.com/web/qita/103517.html](http://tech.cncms.com/web/qita/103517.html)

 [1]: http://blog.haohtml.com/wp-content/uploads/2012/06/fckedit_syntaxhighlighter1.jpg
 [2]: http://tech.cncms.com/UploadFiles/20101001/fck_SyntaxHighlighter.zip