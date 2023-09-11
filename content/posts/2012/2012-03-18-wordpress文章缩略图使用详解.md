---
title: WordPress文章缩略图使用详解
author: admin
type: post
date: 2012-03-18T12:15:48+00:00
url: /archives/12603
IM_contentdowned:
 - 1
categories:
 - 其它

---
许多WordPress主题使用图片代表每篇文章，特别是一些杂志般布局的。图片可能只在首页显示，可能是独立的，或者放在摘要旁边。迄今为止，并没有标准化的方法来实现这个功能。很多主题需要用户冗长乏味的在自定义域中输入图片URL，图片通常需要手动裁切。从WordPress 2.9开始，主题作者可以轻松地开启缩略图选择界面，然后使用简单的模板标签调用图片。

[![](http://blog.haohtml.com/wp-content/uploads/2012/03/064359EQ7.jpg)](http://blog.haohtml.com/wp-content/uploads/2012/03/064359EQ7.jpg)

　　首先，在主题的`functions.php`中声明该主题支持缩略图功能，这将开启WP管理后台中的缩略图设置界面。

```
add_theme_support( 'post-thumbnails' );
```

上面的代码将在文章（Post）和页面（Page）两种内容模型中都开启缩略图选择界面，如果只想选择其一，可以添加参数：

```
add_theme_support( 'post-thumbnails', array( 'post' ) ); // Add it for posts
add_theme_support( 'post-thumbnails', array( 'page' ) ); // Add it for pages
```

添加所需的行到`functions.php`中即可。

下一步，指定缩略图尺寸，共有两个选项：等比例缩放和裁切。等比例缩放就是例如一张100×50图片，设定的大小为50×50，则此图片会被缩放成50×25。这种方式的好处是整张图片都能显示，缺点是生成的图片大小不一，有时是限宽的，有时是限高的。如果想把图片限定在某个宽度、高度不限的话，可以指定宽度，再指定一个绝不可能达到的高度，如99999。

```
set_post_thumbnail_size( 50, 50 ); // 50 pixels wide by 50 pixels tall, box resize mode
```

在第二个选项裁切模式中，图片将被裁切成匹配目标的纵横比，然后在精确地缩放至指定尺寸。好处是生成的图片大小一致，缺点是图片将被裁切（或从左右两边，或从上下）以适合目标尺寸的纵横比，被裁掉的部份无法显示在缩略图中。

```
set_post_thumbnail_size( 50, 50, true ); // 50 pixels wide by 50 pixels tall, hard crop mode
```

现在可以利用模板函数在主题中显示这些图片，这些函数应放到主循环中（**译者注：**作者根据2.9撰写此文，根据3.0默认主题的代码来看，不在主循环内也能使用这些函数调用缩略图）。

`　　has_post_thumbnail()`返回布尔值，表明当前文章是否有手动选的缩略图：

```
<?php
    if ( has_post_thumbnail() ) {
	// the current post has a thumbnail
    } else {
	the current post lacks a thumbnail
    }
?>
```

`　　the_post_thumbnail()`如果存在缩略图则输出缩略图：

```
<?php the_post_thumbnail(); ?>
```

这些是最基本用法，下面是一些高级用法。

假设你想在首页使用50×50硬裁切（hard-cropped）的缩略图，而在文章的永久链接页面使用400像素宽（不限高）的缩略图，指定额外的自定义尺寸即可，代码如下：

`　functions.php代码：`

```
add_theme_support( 'post-thumbnails' );

set_post_thumbnail_size( 50, 50, true ); // Normal post thumbnails

add_image_size( 'single-post-thumbnail', 400, 9999 ); // Permalink thumbnail size
```

`　home.php`或者`index.php`的代码，取决于主题结构：

```
<?php the_post_thumbnail(); ?>
```

`　single.php`代码：

```
<?php the_post_thumbnail( 'single-post-thumbnail' ); ?>
```

`　　set_post_thumbnail_size()`仅仅调用`add_image_size( 'post-thumbnail' )`——默认的文章缩略图“句柄”，但正如你所见可以添加额外的句柄来调用`add_image_size( $handle, $width, $height, {$hard_crop_switch} );`，使用的时候区分句柄即可`the_post_thumbnail( $handle );`

如果想要主题支持更早的WordPress版本，需要用`function_exists()`来阻止早期版本调用这些新函数：

```
if ( function_exists( 'add_theme_support' ) ) { // Added since 2.9
    add_theme_support( 'post-thumbnails' );
    set_post_thumbnail_size( 50, 50, true ); // Normal post thumbnails
    add_image_size( 'single-post-thumbnail', 400, 9999 ); // Permalink thumbnail size
}
```

警告：WordPress 2.9这个功能仅仅对新上传的图片有效，还不能对已存在图片调整大小，我正在考虑在未来版本中实现它（译者注：WordPress 3.0已经可以调整？）。如果你先前上传了缩略图，然后重新声明一个新的尺寸，那么在调用模板函数时将不能对这些先前上传的缩略图硬裁切，等比例缩放也将在浏览器中完成。作为临时的解决方案，Viper007Bond开发了一个插件回溯和创建丢失的缩略图尺寸：[Regenerate Thumbnails][1]。

 [1]: http://wordpress.org/extend/plugins/regenerate-thumbnails/ "http://wordpress.org/extend/plugins/regenerate-thumbnails/"