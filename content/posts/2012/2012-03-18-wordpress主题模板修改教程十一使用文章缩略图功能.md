---
title: WordPress主题(模板)修改教程(十一):使用文章缩略图功能
author: admin
type: post
date: 2012-03-18T14:01:09+00:00
url: /archives/12606
IM_data:
 - 'a:4:{s:70:"http://www.favortt.com/wp-content/uploads/2011/12/thumbnail-action.jpg";s:70:"http://www.favortt.com/wp-content/uploads/2011/12/thumbnail-action.jpg";s:70:"http://www.favortt.com/wp-content/uploads/2011/12/thumbnail-select.jpg";s:70:"http://www.favortt.com/wp-content/uploads/2011/12/thumbnail-select.jpg";s:68:"http://www.favortt.com/wp-content/uploads/2011/12/thumbnail-show.jpg";s:68:"http://www.favortt.com/wp-content/uploads/2011/12/thumbnail-show.jpg";s:69:"http://www.favortt.com/wp-content/uploads/2011/12/thumbnail-style.jpg";s:69:"http://www.favortt.com/wp-content/uploads/2011/12/thumbnail-style.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 其它

---
今天磊子说的这个缩略图功能，其实网上已经传烂了，不知道大家有没有看过。今儿拿出来原因是，感觉网上说的不是很具体，一些设置没有提到，所以先给大家提前说一下。

首页你要看下你所用的主题有没有开启文章缩略图功能，如果看起的话，会在wordpress后台编辑页面或者文章时在右下角的地方看到一个特色图像的设置，如下图：

[![](http://blog.haohtml.com/wp-content/uploads/2012/03/thumbnail-action.jpg)][1]

如果没有说明你还没有激活这功能。我们需要在你主题functions.php里面加一段代码。

```
<?php

add_theme_support( 'post-thumbnails' ); //激活文章和页面的缩略图功能。

//如果你想分别激活它们，可以使用下面的代码：

add_theme_support( 'post-thumbnails', array( 'post' ) ); // 激活文章缩略图功能

add_theme_support( 'post-thumbnails', array( 'page' ) ); // 激活页面缩略图功能

?>
```

这样你的缩略图功能就激活了，然后我们添加图片或者直接点击设置特色图片的时候，你会发现多了一个设置如图。

[![](http://blog.haohtml.com/wp-content/uploads/2012/03/thumbnail-select.jpg)][2]

我们单击作为特色图像，这样你就可以将这个图片作为特色图片显示了。

[![](http://blog.haohtml.com/wp-content/uploads/2012/03/thumbnail-show.jpg)][3]

做好之后我们就要对它进行调用然后在前台显示出来，代码是：

```
<?php

the_post_thumbnail();

//需要将这段代码放在你的主循环中比如:

<?php
 if ( have_posts() ) while ( have_posts() ) : the_post();
 the_post_thumbnail();
 endwhile;
?>
```

这样缩略图就可以显示了。这个基本的方法掌握好之后，下面是一些它的其他使用方法。

1.自定义缩略图的大小(放在主题functions.php里面add\_theme\_support()的下面)

```
<?php set_post_thumbnail_size( $width, $height, $crop ); ?>
//$width 是图片的宽度，可以直接填数字
//$height 是图片的高度,也可以直接填数字
//$crop 是否进行裁剪，默认是false不裁剪，如果填写true 你的图片将会裁剪成你设置的大小。
```

不过set\_post\_thumbnail_size()磊子在用的时候不起作用，不知道大家有没有试过。试过后如果可以使用的记得和磊子说下哈。

我这边使用的是直接规定缩略图大小(直接在主循环里面输出)

```
<?php
the_post_thumbnail('thumbnail');       // 缩略图(最大默认 宽150px高150px)
the_post_thumbnail('medium');          // 中等大小(最大默认 宽300px 高300px)
the_post_thumbnail('large');           // 大图 (最后默认宽1024px高1024px)
the_post_thumbnail('full');            // 原图
the_post_thumbnail( array(100,100) );  // 自己定义宽高
?>
```

```
这里需要多讲一下，设置默认缩略图大小是在wordpress后台 设置->媒体 里面。
```

[![](http://blog.haohtml.com/wp-content/uploads/2012/03/thumbnail-style.jpg)][4]
2.判断文章是否含有缩略图

```
<?php
has_post_thumbnail();

//用法是，通过if如果进行判断

if ( has_post_thumbnail() )
{
	//显示缩略图
} else {
	//没有缩略图( 这里可以放一张默认的图片 )
}

?>
```

3.创建新的缩略图大小(放在主题functions.php文件add\_theme\_support()下面)

我们看到上面设置的图片大小都是等比例缩小的。不管你怎么设置它都是按比列来进行缩小的。如果想设置宽高不等的。便可以使用下面这个函数。

```
<?php

 add_image_size( $name, $width, $height, $crop );

//这里的第一个参数$name是新创建缩略图的名称，其他的参数和上面说的是一样的

//使用方法

add_image_size('home-thumb','200','120');

?>
```

然后我们在显示的时候只需要在填上新的缩略图名称如：the\_post\_thumbnail(‘home-thumb’);这样就可以显示了。

4.获取缩略图ID号

```
<?php  get_post_thumbnail_id();  ?>  //放在主循环中
```

以上便是今天主要讲的内容，大家如果以后有需要的可以来看看。

 [1]: http://blog.haohtml.com/wp-content/uploads/2012/03/thumbnail-action.jpg
 [2]: http://blog.haohtml.com/wp-content/uploads/2012/03/thumbnail-select.jpg
 [3]: http://blog.haohtml.com/wp-content/uploads/2012/03/thumbnail-show.jpg
 [4]: http://blog.haohtml.com/wp-content/uploads/2012/03/thumbnail-style.jpg