---
title: MongoDB 客户端 MongoVue
author: admin
type: post
date: 2012-05-12T04:00:33+00:00
url: /archives/12984
IM_data:
 - 'a:6:{s:75:"http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202156425175.png";s:78:"http://blog.haohtml.com/wp-content/uploads/2012/05/ce7c_201105202156425175.png";s:74:"http://images.cnblogs.com/cnblogs_com/shanyou/201105/20110520215648644.png";s:77:"http://blog.haohtml.com/wp-content/uploads/2012/05/8bb4_20110520215648644.png";s:75:"http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202156569851.png";s:78:"http://blog.haohtml.com/wp-content/uploads/2012/05/a290_201105202156569851.png";s:75:"http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202157041435.png";s:78:"http://blog.haohtml.com/wp-content/uploads/2012/05/f04b_201105202157041435.png";s:75:"http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202157113806.png";s:78:"http://blog.haohtml.com/wp-content/uploads/2012/05/b02f_201105202157113806.png";s:75:"http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202157132462.png";s:78:"http://blog.haohtml.com/wp-content/uploads/2012/05/bf21_201105202157132462.png";}'
IM_contentdowned:
 - 1
categories:
 - nosql
tags:
 - MongoDB
 - mongovue

---
今天在同事那里看到了一个很不错的MongoDB的客户端工具MongoVue，地址是[http://www.mongovue.com/][1]。做的不错，1.0版本的开始收费了，费用也不贵才35＄。真正需要的同学可以掏点钱买个吧，也算是支持这个工具，如果只是学习研究用的话我这里还有一个0.9.7版本，虽然比起1.0版来说有些bug，平常使用也够了，需要的同学可以单独联系我。

1.0版之后超过15天后功能受限。可以通过删除以下注册表项来解除限制：

> [HKEY\_CURRENT\_USER\Software\Classes\CLSID\{B1159E65-821C3-21C5-CE21-34A484D54444}\4FF78130]

把这个项下的值全删掉就可以了。

下面上图给大家感受下强大的MongoVue，可以提高你使用MongoDB的幸福指数好几十点，上图是王道：

1、配置连接

[![image](http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202156425175.png)][2]

2、试下新建一个名为AccessLog的Collection ：

[![image](http://images.cnblogs.com/cnblogs_com/shanyou/201105/20110520215648644.png)][3]

[![image](http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202156569851.png)][4]

3、插入一个Document

[![image](http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202157041435.png)][5]

4、查看我们插入的数据，数据可以通过多种方式展示（树形、表格、文本）

[![image](http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202157113806.png)][6]

上面我们都是通过图形界面的操作的吧，下面有一个窗口列出了上述操作的客户端命令哦，这是学习的好资源，在用图形界面的时候依然可以学习熟悉下命令行。

[![image](http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202157132462.png)][7]

当然上述只是介绍了下最基本的功能，还有更新，删除数据库，从mysql数据库导入数据等等功能，想了解更详细的内容请访问官方网站：[http://www.mongovue.com/][1]

[http://blog.nosqlfan.com/tags/mongodb][8]

 [1]: http://www.mongovue.com/ "http://www.mongovue.com/"
 [2]: http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202156379291.png
 [3]: http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202156459404.png
 [4]: http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202156514807.png
 [5]: http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202157028144.png
 [6]: http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202157089643.png
 [7]: http://images.cnblogs.com/cnblogs_com/shanyou/201105/201105202157122038.png
 [8]: http://blog.nosqlfan.com/tags/mongodb "http://blog.nosqlfan.com/tags/mongodb"