---
title: Nginx常用Rewrite(伪静态规则)
author: admin
type: post
date: 2010-09-09T02:09:23+00:00
url: /archives/5634
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - discuz
 - ecshop
 - nginx
 - phpcms
 - rewrite
 - sablog

---
信现在大部分用Linux VPS的朋友都在使用这个迅速传播的 [Nginx](http://nginx.me/)，今天就整理一下最常见的PHP程序的Rewrite(伪静态规则)。

WordPress：

location / {
index index.html index.php;
if (-f $request_filename/index.html){
rewrite (.*) $1/index.html break;
}
if (-f $request_filename/index.php){
rewrite (.*) $1/index.php;
}
if (!-f $request_filename){
rewrite (.*) /index.php;
}
}

PHPCMS：

location / {
###以下为PHPCMS 伪静态化rewrite规则
rewrite ^(.*)show-([0-9]+)-([0-9]+)\.html$ $1/show.php?itemid=$2&page=$3;
rewrite ^(.*)list-([0-9]+)-([0-9]+)\.html$ $1/list.php?catid=$2&page=$3;
rewrite ^(.*)show-([0-9]+)\.html$ $1/show.php?specialid=$2;

####以下为PHPWind 伪静态化rewrite规则
rewrite ^(.\*)-htm-(.\*)$ $1.php?$2 last;
rewrite ^(.*)/simple/([a-z0-9\_]+\.html)$ $1/simple/index.php?$2 last;
}

ECSHOP：

if (!-e $request_filename)
{
rewrite “^/index\.html” /index.php last;
rewrite “^/category$” /index.php last;
rewrite “^/feed-c([0-9]+)\.xml$” /feed.php?cat=$1 last;
rewrite “^/feed-b([0-9]+)\.xml$” /feed.php?brand=$1 last;
rewrite “^/feed\.xml$” /feed.php last;
rewrite “^/category-([0-9]+)-b([0-9]+)-min([0-9]+)-max([0-9]+)-attr([^-]\*)-([0-9]+)-(.+)-([a-zA-Z]+)(.\*)\.html$” /category.php?id=$1&brand=$2&price\_min=$3&price\_max=$4&filter_attr=$5&page=$6&sort=$7&order=$8 last;
rewrite “^/category-([0-9]+)-b([0-9]+)-min([0-9]+)-max([0-9]+)-attr([^-]\*)(.\*)\.html$” /category.php?id=$1&brand=$2&price\_min=$3&price\_max=$4&filter_attr=$5 last;
rewrite “^/category-([0-9]+)-b([0-9]+)-([0-9]+)-(.+)-([a-zA-Z]+)(.*)\.html$” /category.php?id=$1&brand=$2&page=$3&sort=$4&order=$5 last;
rewrite “^/category-([0-9]+)-b([0-9]+)-([0-9]+)(.*)\.html$” /category.php?id=$1&brand=$2&page=$3 last;
rewrite “^/category-([0-9]+)-b([0-9]+)(.*)\.html$” /category.php?id=$1&brand=$2 last;
rewrite “^/category-([0-9]+)(.*)\.html$” /category.php?id=$1 last;
rewrite “^/goods-([0-9]+)(.*)\.html” /goods.php?id=$1 last;
rewrite “^/article\_cat-([0-9]+)-([0-9]+)-(.+)-([a-zA-Z]+)(.*)\.html$” /article\_cat.php?id=$1&page=$2&sort=$3&order=$4 last;
rewrite “^/article\_cat-([0-9]+)-([0-9]+)(.*)\.html$” /article\_cat.php?id=$1&page=$2 last;
rewrite “^/article\_cat-([0-9]+)(.*)\.html$” /article\_cat.php?id=$1 last;
rewrite “^/article-([0-9]+)(.*)\.html$” /article.php?id=$1 last;
rewrite “^/brand-([0-9]+)-c([0-9]+)-([0-9]+)-(.+)-([a-zA-Z]+)\.html” /brand.php?id=$1&cat=$2&page=$3&sort=$4&order=$5 last;
rewrite “^/brand-([0-9]+)-c([0-9]+)-([0-9]+)(.*)\.html” /brand.php?id=$1&cat=$2&page=$3 last;
rewrite “^/brand-([0-9]+)-c([0-9]+)(.*)\.html” /brand.php?id=$1&cat=$2 last;
rewrite “^/brand-([0-9]+)(.*)\.html” /brand.php?id=$1 last;
rewrite “^/tag-(.*)\.html” /search.php?keywords=$1 last;
rewrite “^/snatch-([0-9]+)\.html$” /snatch.php?id=$1 last;
rewrite “^/group\_buy-([0-9]+)\.html$” /group\_buy.php?act=view&id=$1 last;
rewrite “^/auction-([0-9]+)\.html$” /auction.php?act=view&id=$1 last;
rewrite “^/exchange-id([0-9]+)(.*)\.html$” /exchange.php?id=$1&act=view last;
rewrite “^/exchange-([0-9]+)-min([0-9]+)-max([0-9]+)-([0-9]+)-(.+)-([a-zA-Z]+)(.*)\.html$” /exchange.php?cat\_id=$1&integral\_min=$2&integral_max=$3&page=$4&sort=$5&order=$6 last;
rewrite ^/exchange-([0-9]+)-([0-9]+)-(.+)-([a-zA-Z]+)(.*)\.html$” /exchange.php?cat_id=$1&page=$2&sort=$3&order=$4 last;
rewrite “^/exchange-([0-9]+)-([0-9]+)(.*)\.html$” /exchange.php?cat_id=$1&page=$2 last;
rewrite “^/exchange-([0-9]+)(.*)\.html$” /exchange.php?cat_id=$1 last;
}

SHOPEX:

location / {
if (!-e $request_filename) {
rewrite ^/(.+\.(html|xml|json|htm|php|jsp|asp|shtml))$ /index.php?$1 last;
}
}

SaBlog 2.0：(感谢 [追寻36[正冰]博客](http://blog.is36.cn/) 提供)

\# 只带月份的归档
rewrite “^/date/([0-9]{6})/?([0-9]+)?/?$” /index.php?action=article&setdate=$1&page=$2 last;
\# 无分类翻页
rewrite ^/page/([0-9]+)?/?$ /index.php?action=article&page=$1 last;
\# 分类
rewrite ^/category/([0-9]+)/?([0-9]+)?/?$ /index.php?action=article&cid=$1&page=$2 last;
rewrite ^/category/([^/]+)/?([0-9]+)?/?$ /index.php?action=article&curl=$1&page=$2 last;
\# 归档、高级搜索
rewrite ^/(archives|search|article|links)/?$ /index.php?action=$1 last;
\# 全部评论、标签列表、引用列表 带分页
rewrite ^/(comments|tagslist|trackbacks|article)/?([0-9]+)?/?$ /index.php?action=$1&page=$2 last;
\# tags
rewrite ^/tag/([^/]+)/?([0-9]+)?/?$ /index.php?action=article&item=$1&page=$2 last;
\# 文章
rewrite ^/archives/([0-9]+)/?([0-9]+)?/?$ /index.php?action=show&id=$1&page=$2 last;
\# RSS rewrite ^/rss/([0-9]+)?/?$ /rss.php?cid=$1 last;
rewrite ^/rss/([^/]+)/?$ /rss.php?url=$1 last;
\# 用户 rewrite ^/uid/([0-9]+)/?([0-9]+)?/?$ /index.php?action=article&uid=$1&page=$2 last;
rewrite ^/user/([^/]+)/?([0-9]+)?/?$ /index.php?action=article&user=$1&page=$2 last;
\# 地图文件
rewrite sitemap.xml sitemap.php last;
\# 自定义链接
rewrite ^(.*)/([0-9a-zA-Z\-\_]+)/?([0-9]+)?/?$ $1/index.php?action=show&alias=$2&page=$3 last;

Discuz 7：

rewrite ^/archiver/((fid|tid)-[\w\-]+\.html)$ /archiver/index.php?$1 last;
rewrite ^/forum-([0-9]+)-([0-9]+)\.html$ /forumdisplay.php?fid=$1&page=$2 last;
rewrite ^/thread-([0-9]+)-([0-9]+)-([0-9]+)\.html$ /viewthread.php?tid=$1&extra=page\%3D$3&page=$2 last;
rewrite ^/space-(username|uid)-(.+)\.html$ /space.php?$1=$2 last;
rewrite ^/tag-(.+)\.html$ /tag.php?name=$1 last;

Typecho：

location / {
index index.html index.php;
if (-f $request_filename/index.html){
rewrite (.*) $1/index.html break;
}
if (-f $request_filename/index.php){
rewrite (.*) $1/index.php;
}
if (!-f $request_filename){
rewrite (.*) /index.php;
}
}

今天暂时整理这些，以后不定期更新。