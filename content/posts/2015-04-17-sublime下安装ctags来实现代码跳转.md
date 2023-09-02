---
title: sublime下安装ctags插件来实现代码跳转
author: admin
type: post
date: 2015-04-16T18:35:14+00:00
url: /archives/15603
categories:
 - 程序开发
tags:
 - sublime

---
本次操作是在sublime text 2下进行。

1、先到 [http://sublime.wbond.net/Package%20Control.sublime-package](http://sublime.wbond.net/Package%20Control.sublime-package "下载地址") 下载Package Control.sublime-package，然后打开Preferences->Browes Packages,显示当前目录是Packages,跳到上一级目录看到Installed Packages,就把Package Control.sublime-package放入Installed Packages.

测试安装成功了没：

在sublime下快捷键Ctrl+Shift+P,输入install，如果有显示出安装列表，则表明安装成功，则可以进行下一步。

2、在sublime下快捷键Ctrl+Shift+P,输入install，然后在安装列表下输入ctags插件，选择然后安装。

之后在win7下或者linux下安装ctags软件([ctags.exe][1])，然后配置环境变量(或者不用)，然后在打开的工程目录上运行

ctags -R -f .tags  生成  .tags文件

[![sublime_ctags](http://blog.haohtml.com/wp-content/uploads/2015/04/sublime_ctags.png)][2]

然后在sublime下就可以用ctrl+t ctrl+t来跳转,用ctrl+t ctrl+b来返回到原来位置。

生成.tags文件后，这用sublime打开项目以后，就可以用下面方法跳转到函数声明

```
ctrl+t   ctrl+t   //鼠标在函数出执行，跳到函数处

ctrl+t   ctrl+b  //调回函数
```

当然用 ctrl+shift+鼠标左键 也可以跳到!

以上步骤参考 [http://alfred-long.iteye.com/blog/1668074](http://alfred-long.iteye.com/blog/1668074) [http://www.cnblogs.com/qq78292959/p/3811467.html](http://www.cnblogs.com/qq78292959/p/3811467.html)

也可以使用liteide编辑器来实现此功能，参考: [http://segmentfault.com/q/1010000002664962](http://segmentfault.com/q/1010000002664962)

 [1]: http://blog.haohtml.com/wp-content/uploads/2015/04/ctags.exe_.txt
 [2]: http://blog.haohtml.com/wp-content/uploads/2015/04/sublime_ctags.png