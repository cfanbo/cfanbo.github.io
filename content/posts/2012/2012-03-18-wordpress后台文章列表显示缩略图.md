---
title: wordpress后台文章列表显示缩略图
author: admin
type: post
date: 2012-03-18T12:29:39+00:00
url: /archives/12607
IM_data:
 - 'a:1:{s:68:"http://www.favortt.com/wp-content/uploads/2011/12/thumbnail-list.png";s:68:"http://www.favortt.com/wp-content/uploads/2011/12/thumbnail-list.png";}'
IM_contentdowned:
 - 1
categories:
 - 数据库

---
大家都知道我们在wordpress后台添加文章或页面时如果你启用了缩略图功能，那么会在添加时有个特色图像的设置。具体的大家可以看下我的教程( [点击查看](http://www.favortt.com/wordpress-theme-thumbnails.html "WordPress主题(模板)修改教程(十一):使用文章缩略图功能"))，当我们添加好后。如果你在wordpress后台需要看某个文章或页面的缩略图是什么的时候，还得单击编辑才能看到。这样是不是很麻烦呢？如果我们直接把缩略图显示在文章或者页面的列表上面，这样的话就一目了然了。如下面效果图:

[![](http://blog.haohtml.com/wp-content/uploads/2012/03/thumbnail-list.png)][1]

今天磊子就把这个功能的实现，分享给大家，我们需要用到的是wordpress插件API里面的函数方法。看下面代码：

```
<?php
add_filter('manage_posts_columns', 'lei_add_thumb_col');
function lei_add_thumb_col($cols) {
	$cols['thumbnail'] = __('Thumbnail');
	return $cols;
}

//__('Thumbnail')是显示的文字标题，也可以改成__('缩略图')。

//通过manage_posts_columns方法将文字标题显示在文章列表上

add_action('manage_posts_custom_column', 'lei_get_thumb_show');
function lei_get_thumb_show($column_name ) {
	if ( $column_name  == 'thumbnail'  ) {
		echo get_the_post_thumbnail(get_the_ID(),array(100,100));
	}
}
?>

//get_the_post_thumbnail获取缩略图以及设置它的大小为宽100，高100
//通过manage_posts_custom_column方法将缩略图显示在列表上面
```

将上面两段代码放在你所用主题的functions.php里面,就可以在文章列表上面显示缩略图了。那么显示页面的缩略图，和文章的方法是一样的。只需要将manage_posts_columns和manage_posts_custom_column中间的posts改成manage_pages_columns和manage_pages_custom_column即可，是不是很方便也很简单呢！具体代码给大家贴出来，方便大家使用.

```
<?php
add_filter('manage_pages_columns', 'lei_add_page_thumb_col');
function lei_add_page_thumb_col($cols) {
	$cols['thumbnail'] = __('Thumbnail');
	return $cols;
}
//__('Thumbnail')是显示的文字标题，也可以改成__('缩略图')。
//通过manage_posts_columns方法将文字标题显示在文章列表上

add_action('manage_pages_custom_column', 'lei_get_page_thumb_show');
function lei_get_page_thumb_show($column_name ) {
	if ( $column_name  == 'thumbnail'  ) {
		echo get_the_post_thumbnail(get_the_ID(),array(100,100));
	}
}
```

 [1]: http://blog.haohtml.com/wp-content/uploads/2012/03/thumbnail-list.png