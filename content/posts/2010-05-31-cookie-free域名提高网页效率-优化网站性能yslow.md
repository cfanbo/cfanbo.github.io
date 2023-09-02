---
title: cookie-free域名提高网页效率-优化网站性能(yslow)
author: admin
type: post
date: 2010-05-31T00:38:45+00:00
url: /archives/3739
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - 网页优化

---
YSlow给如何提高网页效率和优化网站性能提供了22条建议，其中有一条是关于域名的：Use cookie-free domains。

使用 cookie-free domains 有什么好处呢？当用户浏览器发送一个静态文件，如图片image、CSS样式表文件时会同时发送同一个域名（或二级域名）下的cookies，但是网站服 务器对发送过来的cookies完全不予理会，因此这些没用的cookies白白浪费了网站带宽，影响网站加载速度和网页性能表现。YSlow建议为了解 决这个问题，就可以通过使用 cookie-free domains 的方法来做优化，从而提高网页效率。

**使用二级域名作为cookie-free domains**

通俗地说，所谓的 cookie-free domains 就是在浏览器发送静态内容的请求时不会发送cookies 的域名。YSlow提示可以申请注册一个二级域名专门用来储存这些静态图片、JS、静态CSS文件。

在前面泛域名解析设置影响seo和Google PR值这里提到了www开头，形如www.haohtml.com的域名实际上也是属于二级域名。如果你的网站主域名是www开头的域名，建立一个二级域 名作为单独储存（hosting）静态图片、JS、CSS文件的cookie-free domains 是可行的；但是如果网站主域名用的是比较短的顶级域名，如远方博客用的是不带www的顶级域名haohtml.com，使用新创建的二级域名作为 cookie-free domains的方法是无效的。因为顶级域名haohtml.com会向所有被请求的静态文件二级域名服务器发送cookies。

即www.haohtml.com 和 blog.haohtml.com是互相独立的两个“二级域名”，不会造成域名污染， blog.haohtml.com 可以作为cookie-free 域名；但是需要做一些设置，比如下面介绍的Wordpress 博客设置wp-config.php文件的实例。

顶级域名haohtml.com 会向所有被请求的二级域名（子域名：www.haohtml.com和blog.haohtml.com）发送 cookies，blog.haohtml.com 也会被污染，不能当作cookie-free 域名。具体原因在下面Wordpress 博客cookie-free domains设置中有介绍。

**使用独立域名作为cookie-free domains**

那么使用顶级域名的博客应该如何使用 cookie-free domains？解决方法是使用另外一个独立域名。比如雅虎Yahoo！ 自身使用的是就是独立域名ymig.com来作为cookie-free domains的，YouTube使用的是ytimg.com 独立域名。

现在注册一个域名也很便宜的，godaddy 域名以.com .info .org .net 后缀的域名第一年购买都很便宜，第二年续费比较贵，这时候第二年可以再换一个新的。其他一些域名注册商也差不多这样。

**WordPress 博客cookie-free domains 设置**

在Wordpress 博客中，针对使用带www域名作为网站主域名，其他二级域名作为cookie-free domains 的情况，还要再另外设置Cookie的作用域就可以了。打开wp-config.php文件，设置COOKIE_DOMAIN：

所谓的COOKIE_DOMAIN，就是cookie-free domains相反的意思。看看Wordpress 对Set Cookie Domain 的介绍：

 为Wordpress设置的COOKIES Domain 可以进行一些特殊情况下的域名设置。比如使用二级域名存放静态内容。为了阻止Wordpress Cookies 在对每一个二级域名上的静态内容请求时被传送，我们可以只设置非静态域名为cookie domian。

The domain set in the cookies for WordPress can be specified for those with unusual domain setups. One reason is if subdomains are used to serve static content. To prevent WordPress cookies from being sent with each request to static content on your subdomain you can set the cookie domain to your non-static domain only.

设置COOKIE_DOMAIN就可以指定哪个二级域名需要传送cookies，其他的域名不发送cookies。所以如果我们的网站主域名用的是 顶级域名，COOKIE_DOMAIN就必须设置为顶级域名haohtml.com了，而顶级域名的设置会映射到各个子域名，所以即使另外添加二级域名作 为cookie-free domains 也无效了。这时只能另外注册一个独立的顶级域名。


下面以独立域名haohtmlimg.com ,为例演示Wordpress 博客（haohtml.com） cookie-free domains 设置步骤：


1. 图片用单独的cookie-free域名储存


首先进入haohtml.com网站空间控制面板新增绑定域名：haohtmlimg.com。


然后进入Wordpress 管理后台设置：控制面板–设置–杂项–文件的完整URL地址填写http://haohtmlimg.com/，设置如下图：


[![2015320T960](http://blog.haohtml.com/wp-content/uploads/2010/05/2015320T960.jpg)](http://blog.haohtml.com/wp-content/uploads/2010/05/2015320T960.jpg)

在发表文章时上传图片，前台显示的图片域名地址就是 cookie-free domains （haohtmlimg.com）了。


如果是使用图床（可以上传图片，然后获得图片链接地址的网站空间）就更简单了，将图片上传到图片空间的网站服务器，发表文章的时候将图片链接地址取 过来即可。


2. WordPress CSS文件、JS文件设置独立域名储存


和图片一样，把CSS和JS上传到绑定cookie-free 域名的空间，取链接URL地址。然后在Wordpress 主题文件里修改CSS和JS的url。


3. WordPress 表情图片


方法同上，上传后修改wp-includes/formatting.php文件。将


 $srcurl = apply_filters(‘smilies_src’, “$siteurl/wp-includes/images/smilies/$img”, $img, $siteurl);


将红色部分改为上传到cookie-free domains 的网站空间的表情图片的URL链接地址。


另外YSlow 关于Cookies 的另外一个网站速度提升建议，Cookie信息在网站服务器和浏览器之间传送的HTTP headers 中交换，因此要尽量缩小cookie 大小，单个cookie不要超过4K。