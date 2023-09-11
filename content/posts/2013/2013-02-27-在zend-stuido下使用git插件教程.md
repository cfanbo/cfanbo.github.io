---
title: 在zend stuido下使用git插件教程
author: admin
type: post
date: 2013-02-27T03:02:45+00:00
url: /archives/13689
categories:
 - 程序开发
tags:
 - git

---
[上一节zend stuido下安装了git软件插件](http://blog.haohtml.com/archives/13679)。下面我们来讲一下git插件的使用方法.

由于我们目前已经创建好了git项目。所以这里只介绍直接从现成的git项目仓库导入.

选择菜单”文件(File)”->”Import”

[![zendstudio-git-guide-1](https://blogstatic.haohtml.com//uploads/2023/09/zendstudio-git-guide-1.png)][1]

[![zendstudio-git-guide-2](https://blogstatic.haohtml.com//uploads/2023/09/zendstudio-git-guide-2.png)][2] [![zendstudio-git-guide-3](http://blog.haohtml.com/wp-content/uploads/2013/02/zendstudio-git-guide-3.png)][3]



点击”Browse…”选择存放git的目录，然后点击”Search”按钮这样就可以读取一些git配置信息，并在上面显示出来项目目录下的所有文件.

[![zendstudio-git-guide-4](https://blogstatic.haohtml.com//uploads/2023/09/zendstudio-git-guide-4.png)][4] [![zendstudio-git-guide-5](http://blog.haohtml.com/wp-content/uploads/2013/02/zendstudio-git-guide-5.png)][5]最后一步是选择当前项目的名字，这个随便起的。最后点击”Finish”按钮就可以了。

这时在IDE左侧会看到项目名字及项目结构信息。

[![zendstudio-git-guide-6](https://blogstatic.haohtml.com//uploads/2023/09/zendstudio-git-guide-6.png)][6]

下面可以修改一个文件，然后在左侧的导航里选择修改的文件，右键点击，选择”Team” 菜单，再选择”Commit”菜单，会弹出一个对话框，在”Commit message”对话框时里输入提示备注信息。点击”Commit”按钮就可以了。

[1]: http://blog.haohtml.com/wp-content/uploads/2013/02/zendstudio-git-guide-1.png
[2]: http://blog.haohtml.com/wp-content/uploads/2013/02/zendstudio-git-guide-2.png
[3]: http://blog.haohtml.com/wp-content/uploads/2013/02/zendstudio-git-guide-3.png
[4]: http://blog.haohtml.com/wp-content/uploads/2013/02/zendstudio-git-guide-4.png
[5]: http://blog.haohtml.com/wp-content/uploads/2013/02/zendstudio-git-guide-5.png
[6]: http://blog.haohtml.com/wp-content/uploads/2013/02/zendstudio-git-guide-6.png