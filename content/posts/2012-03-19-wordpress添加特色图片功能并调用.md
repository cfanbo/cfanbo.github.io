---
title: WordPress添加“特色图片”功能并调用
author: admin
type: post
date: 2012-03-19T08:54:21+00:00
url: /archives/12632
IM_contentdowned:
 - 1
categories:
 - 其它

---
我们在使用wordpress建博客的时候，是不是会发现有些主题中有“设为特色图像”功能，设置后会在首页或者文章的左上角，右上角加上特色的文章LOGO图片，还是比较酷的。但是有些主题是没有的，这是如何设置的呢？如果你的主题没有，但就想加上这个功能，如何设置？

第一步，在你的改款主题的functions.php加入如下代码：

```
add_theme_support( 'post-thumbnails' );
```

第二步，在你的首页文件index.php模板内容位置加入：

```
<?php if ( has_post_thumbnail() ) {
the_post_thumbnail(); ?>
<?php } else {?>
<img src="<?php bloginfo('template_url'); ?>/images/xxx.jpg" />
<?php } ?>
```

注：XXX.JPG为你在没有特色图片的时候显示的默认图片。

第三步，完毕，在添加文章的时候添加特色图即可显示了。