---
title: WordPress教程：wp_list_pages()函数
author: admin
type: post
date: 2012-04-02T16:42:53+00:00
url: /archives/12653
IM_contentdowned:
 - 1
categories:
 - 其它

---
让页面可以设置号码，就代表着可以根据那些号码进行排序，下午研究了下代码，去官方网站查看了下wp\_list\_pages()的介绍，转载过来顺便介绍下函数怎么应用，方便自己及其他人修改模板。

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
```

```
$defaults = array(

'depth' => 0,

'show_date' => '',

'date_format' => get_option('date_format'),

'child_of' => 0,

'exclude' => '',

'title_li' => __('Pages'),

'echo' => 1,

'authors' => '',

'sort_column' => 'menu_order, post_title');
```

默认的参数如上，你可以在wp_list_pages()中采用wp_list_pages(’arguments’);的形式调用。

**参数如下：**

* depth：深度。页面可以建N级的，你可以调用一层，也可以调用N层。

* show_date：创建日期。这里可以设置是不是显示创建的日期。

* date_format：如果你设置显示日期，你可以在这里设置限制的格式，无非就是那个y了d了啥的。

* child_of：这里设置是不是显示子页面。

* exclude：设置显示哪些页面，根据页面设置的代码。

* title_li：这里设置显示不显示标题，就像侧栏那里日志分类上面有个h2包起来的“日志分类”一样。

* echo：  有两个值，1和0，1就显示页面链接，2就不显示。

* authors：  wp里面这个很多了，就是显示不显示作者了。

* sort_columu：这个下面好像内容很多，是设置根据啥排序的。默认的是根据”post_title”，还有”menu_order”；”post_date”；”ID”等等。


更多参考： [http://codex.wordpress.org/zh-cn:Main_Page](http://codex.wordpress.org/zh-cn:Main_Page)