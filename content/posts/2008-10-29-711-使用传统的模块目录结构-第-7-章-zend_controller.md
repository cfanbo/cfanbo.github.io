---
title: 7.11. 使用传统的模块目录结构 第 7 章 Zend_Controller
author: admin
type: post
date: 2008-10-29T04:47:57+00:00
excerpt: |
 |
 使用传统的模块目录结构:
 传统的模块目录结构允许你把不同的MVC应用程序分离成为独立的单元，并和不同的前端控制器配合再使用。示例一下这样的目录结构：
url: /archives/477
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - mvc

---

## 7.11. 使用传统的模块目录结构

### 7.11.1. 简介

传统的模块目录结构允许你把不同的MVC应用程序分离成为独立的单元，并和不同的前端控制器配合再使用。示例一下这样的目录结构：


```
docroot/
    index.php
application/
    default/
        controllers/
            IndexController.php
            FooController.php
        models/
        views/
            scripts/
                index/
                foo/
            helpers/
            filters/
    blog/
        controllers/
            IndexController.php
        models/
        views/
            scripts/
                index/
            helpers/
            filters/
    news/
        controllers/
            IndexController.php
            ListController.php
        models/
        views/
        views/
            scripts/
                index/
                list/
            helpers/
            filters/
```

在这个范例中，模块名作为它所包含的控制器的前缀。上面的例子包含三个模块控制器：’Blog_IndexController’ ‘News_IndexController’ 和’News_ListController’。也定义了两个全局控制器：’IndexController’ 和 ‘FooController’。它们都将不需要命名空间前缀。这个目录结构在本章中用作为例子。


在缺省模块中不用命名空间前缀

注意在缺省模块中，控制器不需要一个命名空间前缀。这样，在上例中，在缺省模块中的控制器不需要’Default_’这样的前缀－－根据它们的基本控制器名’IndexController’ 和 ‘FooController’被简单地派遣。然而，命名空间前缀被用于所有其它模块。


那么，你怎样用Zend Framework MVC组件来实现这样的目录结构？


### 7.11.2. 指定模块控制器目录

利用模块的第一步是来修改你如何在前端控制器指定控制器目录列表。在基本的MVC系列，传递数组或字符串给 `setControllerDirectory()`，或者传递路径给 `addControllerDirectory()`。当使用模块，你需要稍微修改对这些方法的调用。


用 `setControllerDirectory()`，你将需要传递一个关联数组和指定‘模块名/目录路径’的‘键/值’对。特殊的键 `default` 将被用作全局控制器（不需要模块命名空间）。所有条目应该包含指向一个单个路径的字符串键，并且 `default` 键必须出现。例如：


```
<?php
$front->setControllerDirectory(array(
      'default' => '/path/to/application/controllers',
      'blog'    => '/path/to/application/blog/controllers'
));
```

`addControllerDirectory()` 将带有可选的第二个参数。当使用模块，传递模块名作为第二个参数；如果没有指定，路径将被加到 `default` 命名空间。例如：


```
<?php
$front->addControllerDirectory('/path/to/application/news/controllers', 'news');
```

把最好的保留到最后，指定模块的最容易的方法是整体来做，把所有模块放到一个通用的目录并使用相同的结构。这可以用 `addModuleDirectory()` 来完成：


```
<?php
/**
 * Assuming the following directory structure:
 * application/
 *     modules/
 *         default/
 *             controllers/
 *         foo/
 *             controllers/
 *         bar/
 *             controllers/
 */
$front->addModuleDirectory('/path/to/application/modules');
```

上面的例子将定义 `default`、 `foo` 和 `bar` 模块，每个都分别指向它们的模块的 `controllers` 子目录。


你可以在模块中通过 `setModuleControllerDirectoryName()` 来定制模块子目录：


```
<?php
/**
 * Change the controllers subdirectory to be 'con'
 * application/
 *     modules/
 *         default/
 *             con/
 *         foo/
 *             con/
 *         bar/
 *             con/
 */
$front->setModuleControllerDirectoryName('con');
$front->addModuleDirectory('/path/to/application/modules');
注意
```

你可以通过传递一个空值给 `setModuleControllerDirectoryName()` 来指定在你的模块中没有控制器子目录。


### 7.11.3. Routing to modules

在 `Zend_Controller_Router_Rewrite` 中缺省的路由是一个 `Zend_Controller_Router_Route_Module` 类型的对象。这个路由使用下面路由计划之一：


- `:module/:controller/:action/*`
- `:controller/:action/*`

换句话说，它将自己或通过先前的模块来匹配控制器和动作。匹配规则指定一个模块将只匹配在传递给前端控制器和派遣器的控制器目录数组中存在同名的键。


### 7.11.4. 模块或全局缺省控制器

在缺省的路由器中，如果在URL中没有指定控制器，缺省控制器就被使用（ `IndexController`，除非另外要求）。对于模块控制器，如果一个模块被指定但没有控制器，派遣器首先寻找在这个模块路径中的缺省控制器，然后回到在’default’、全局、命名空间中发现的缺省控制器。


如果你总愿意缺省到全局命名空间，在前端控制器中设置 `useDefaultControllerAlways` 参数：


```
<?php
$front->setParam('useDefaultControllerAlways', true);
```