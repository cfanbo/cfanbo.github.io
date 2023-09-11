---
title: windows下chrome器安装Speed Tracer
author: admin
type: post
date: 2011-04-01T02:20:07+00:00
url: /archives/8830
categories:
 - 前端设计

---
在上一篇文章( [15个对Web设计和开发有用的Chrome插件](http://blog.haohtml.com/archives/8828))我们提到一个****Speed Tracer 速度追踪**** 插件,现在我们就来安装一下,我这里用的是windows的操作系统,chrome的版本为10.0.648.204.目前为最新的版本了.

**1.安装”Speed Tracer 速度追踪器”插件**

[![](https://blogstatic.haohtml.com//uploads/2023/09/speed-tracer-by-google.bmp)][1]**2.配置chrome浏览器**

安装完插件以后,将chorme浏览器关闭,记得要全部关闭.下面直接将官方的安装过程贴出来了,虽然是英文的,不过有图片很容易懂的.

 1. **Start Google Chrome with a flag that enables Speed Tracer to work**, as described next. The process is different for Windows and Macintosh.**– Windows**

Start Google Chrome with the following flag, either from the command line or by modifying your desktop shortcut for Google Chrome:


`--enable-extension-timeline-api`

To modify your desktop shortcut for Google Chrome, right-click on the Google Chrome shortcut icon and choose Properties:

[![](https://blogstatic.haohtml.com//uploads/2023/09/windows-cmdline-1.png)](http://blog.haohtml.com/wp-content/uploads/2011/04/windows-cmdline-1.png)

Then, paste the `--enable-extension-timeline-api` flag into the Target field at the very end of the string (with a space separating it from chrome.exe).

[![](https://blogstatic.haohtml.com//uploads/2023/09/windows-cmdline-2.png)](http://blog.haohtml.com/wp-content/uploads/2011/04/windows-cmdline-2.png)

Click OK to save the setting and dismiss the dialog. To start Google Chrome, double-click on the shortcut icon you just modified.



 **– Mac OS X**

 On Mac OS X, you need a bootstrap application to set the appropriate flag in Google Chrome. First [download the Speed Tracer bootstrap application][2], then **Quit Google Chrome** and restart it by running the “ChromeWithSpeedTracer” application that you just downloaded. Please ensure that you are running the Dev Channel version of Google Chrome from step 1.

 Alternatively, you can start Chrome from the command line (or an alias) with this flag:



```
$ /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --enable-extension-timeline-api &
```




 * **Install Speed Tracer** – Now Google Chrome is ready for you to add Speed Tracer extension. Click on this install button:
 [![](https://blogstatic.haohtml.com//uploads/2023/09/install-speedtracer.png)][3]
  By installing this extension, you agree to the [Google Chrome Extension Gallery Terms of Service][4].

 官方地址:

 说白了,就是将chorme浏览器的快捷方式上,右键点击打开”属性”对话框.在”目标”右侧的框里最后面添加上  –enable-extension-timeline-api, 注意这里是在双引号的最后面添加的.上面的文档容易让人模糊的.在–enable-extension-timeline-api前面还有一个空格,这个很重要的.最后添加完的结果为

 > “C:\Documents and Settings\Administrator\Local Settings\Application Data\Google\Chrome\Application\chrome.exe” –enable-extension-timeline-api

 对于MAX OSX的情况应该都一样的.

[1]: http://blog.haohtml.com/wp-content/uploads/2011/04/speed-tracer-by-google.bmp
[2]: http://dl.google.com/gwt/speedtracer/ChromeWithSpeedTracer.dmg
[3]: http://blog.haohtml.com/wp-content/uploads/2011/04/install-speedtracer.png
[4]: https://chrome.google.com/extensions/intl/en/gallery_tos.html