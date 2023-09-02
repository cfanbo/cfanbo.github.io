---
title: nginx rewrite规则和参考
author: admin
type: post
date: 2010-09-09T01:12:40+00:00
url: /archives/5631
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx
 - rewrite

---
推荐参考地址：
Mailing list ARChives 官方讨论区

Nginx 常见应用技术指南[Nginx Tips]

**本日志内容来自互联网和平日使用经验，整理一下方便日后参考。**

**正则表达式匹配，其中：**

1. * ~ 为区分大小写匹配

2. * ~* 为不区分大小写匹配

3. * !~和!~*分别为区分大小写不匹配及不区分大小写不匹配


**文件及目录匹配，其中：**

1. * -f和!-f用来判断是否存在文件

2. * -d和!-d用来判断是否存在目录

3. * -e和!-e用来判断是否存在文件或目录

4. * -x和!-x用来判断文件是否可执行

**flag标记有：**

1. * last 相当于Apache里的[L]标记，表示完成rewrite

2. * break 终止匹配, 不再匹配后面的规则

3. * redirect 返回302临时重定向 地址栏会显示跳转后的地址

4. * permanent 返回301永久重定向 地址栏会显示跳转后的地址


**一些可用的全局变量有，可以用做条件判断(待补全)**

01. $args

02. $content_length

03. $content_type

04. $document_root

05. $document_uri

06. $host

07. $http_user_agent

08. $http_cookie

09. $limit_rate

10. $request_body_file

11. $request_method

12. $remote_addr

13. $remote_port

14. $remote_user

15. $request_filename

16. $request_uri

17. $query_string

18. $scheme

19. $server_protocol

20. $server_addr

21. $server_name

22. $server_port

23. $uri


**结合QeePHP的例子**

1. if (!-d $request_filename) {

2. rewrite ^/([a-z-A-Z]+)/([a-z-A-Z]+)/?(.*)$ /index.php?namespace=user&controller=$1&action=$2&$3 last;

3. rewrite ^/([a-z-A-Z]+)/?$ /index.php?namespace=user&controller=$1 last;

4. break;


**多目录转成参数**
abc.domian.com/sort/2 => abc.domian.com/index.php?act=sort&name=abc&id=2

1. if ($host ~* (.*)\.domain\.com) {

2. set $sub_name $1;

3. rewrite ^/sort\/(\d+)\/?$ /index.php?act=sort&cid=$sub_name&id=$1 last;

4. }


**目录对换**
/123456/xxxx -> /xxxx?id=123456

1. rewrite ^/(\d+)/(.+)/ /$2?id=$1 last;


**例如下面设定nginx在用户使用ie的使用重定向到/nginx-ie目录下：**

1. if ($http_user_agent ~ MSIE) {

2. rewrite ^(.*)$ /nginx-ie/$1 break;

3. }


**目录自动加“/”**

1. if (-d $request_filename){

2. rewrite ^/(.*)([^/])$ http://$host/$1$2/ permanent;

3. }


**禁止htaccess**

1. location ~/\.ht {

2. deny all;

3. }


**禁止多个目录**

1. location ~ ^/(cron|templates)/ {

2. deny all;

3. break;

4. }


**禁止以/data开头的文件**
可以禁止/data/下多级目录下.log.txt等请求;

1. location ~ ^/data {

2. deny all;

3. }


**禁止单个目录**
不能禁止.log.txt能请求

1. location /searchword/cron/ {

2. deny all;

3. }


**禁止单个文件**

1. location ~ /data/sql/data.sql {

2. deny all;

3. }


**给favicon.ico和robots.txt设置过期时间;**
这里为favicon.ico为99天,robots.txt为7天并不记录404错误日志

01. location ~(favicon.ico) {

02. log_not_found off;

03. expires 99d;

04. break;

05. }

07. location ~(robots.txt) {

08. log_not_found off;

09. expires 7d;

10. break;

11. }


**设定某个文件的过期时间;这里为600秒，并不记录访问日志**

1. location ^~ /html/scripts/loadhead_1.js {

2. access_log   off;

3. root /opt/lampp/htdocs/web;

4. expires 600;

5. break;

6. }


**文件反盗链并设置过期时间**
这里的return 412 为自定义的http状态码，默认为403，方便找出正确的盗链的请求
“rewrite ^/ http://leech.c1gstudio.com/leech.gif;”显示一张防盗链图片
“access_log off;”不记录访问日志，减轻压力
“expires 3d”所有文件3天的浏览器缓存

01. location ~* ^.+\.(jpg|jpeg|gif|png|swf|rar|zip|css|js)$ {

02. valid_referers none blocked *.c1gstudio.com *.c1gstudio.net localhost 208.97.167.194;

03. if ($invalid_referer) {

04. rewrite ^/ http://leech.c1gstudio.com/leech.gif;

05. return 412;

06. break;

07. }

08. access_log   off;

09. root /opt/lampp/htdocs/web;

10. expires 3d;

11. break;

12. }


**只充许固定ip访问网站，并加上密码**

1. root  /opt/htdocs/www;

2. allow   208.97.167.194;

3. allow   222.33.1.2;

4. allow   231.152.49.4;

5. deny    all;

6. auth_basic “C1G_ADMIN”;

7. auth_basic_user_file htpasswd;


**将多级目录下的文件转成一个文件，增强seo效果**
/job-123-456-789.html 指向/job/123/456/789.html

1. rewrite ^/job-([0-9]+)-([0-9]+)-([0-9]+)\.html$ /job/$1/$2/jobshow_$3.html last;


**将根目录下某个文件夹指向2级目录**
如/**shanghai**job/ 指向 /area/**shanghai**/
如果你将last改成permanent，那么浏览器地址栏显是/location/shanghai/

1. rewrite ^/([0-9a-z]+)job/(.*)$ /area/$1/$2 last;


上面例子有个问题是访问/shanghai 时将不会匹配

1. rewrite ^/([0-9a-z]+)job$ /area/$1/ last;

2. rewrite ^/([0-9a-z]+)job/(.*)$ /area/$1/$2 last;


这样/shanghai 也可以访问了，但页面中的相对链接无法使用，
如./list\_1.html真实地址是/area/shanghia/list\_1.html会变成/list_1.html,导至无法访问。

那我加上自动跳转也是不行咯
(-d $request_filename)它有个条件是必需为真实目录，而我的rewrite不是的，所以没有效果

1. if (-d $request_filename){

2. rewrite ^/(.*)([^/])$ http://$host/$1$2/ permanent;

3. }


知道原因后就好办了，让我手动跳转吧

1. rewrite ^/([0-9a-z]+)job$ /$1job/ permanent;

2. rewrite ^/([0-9a-z]+)job/(.*)$ /area/$1/$2 last;


**文件和目录不存在的时候重定向：**

1. if (!-e $request_filename) {

2. proxy_pass http://127.0.0.1;

3. }


**域名跳转**

1. server

2. {

3. listen       80;

4. server_name  jump.c1gstudio.com;

5. index index.html index.htm index.php;

6. root  /opt/lampp/htdocs/www;

7. rewrite ^/ http://www.c1gstudio.com/;

8. access_log  off;

9. }


**多域名转向**

1. server_name  www.c1gstudio.com www.c1gstudio.net;

2. index index.html index.htm index.php;

3. root  /opt/lampp/htdocs;

4. if ($host ~ “c1gstudio\.net”) {

5. rewrite ^(.*) http://www.c1gstudio.com$1 permanent;

6. }


**三级域名跳转**

1. if ($http_host ~* “^(.*)\.i\.c1gstudio\.com$”) {

2. rewrite ^(.*) http://top.yingjiesheng.com$1;

3. break;

4. }


**域名镜向**

1. server

2. {

3. listen       80;

4. server_name  mirror.c1gstudio.com;

5. index index.html index.htm index.php;

6. root  /opt/lampp/htdocs/www;

7. rewrite ^/(.*) http://www.c1gstudio.com/$1 last;

8. access_log  off;

9. }


**某个子目录作镜向**

1. location ^~ /zhaopinhui {

2. rewrite ^.+ http://zph.c1gstudio.com/ last;

3. break;

4. }


**discuz ucenter home (uchome) rewrite**

1. rewrite ^/(space|network)-(.+)\.html$ /$1.php?rewrite=$2 last;

2. rewrite ^/(space|network)\.html$ /$1.php last;

3. rewrite ^/([0-9]+)$ /space.php?uid=$1 last;


**discuz 7 rewrite**

1. rewrite ^(.*)/archiver/((fid|tid)-[\w\-]+\.html)$ $1/archiver/index.php?$2 last;

2. rewrite ^(.*)/forum-([0-9]+)-([0-9]+)\.html$ $1/forumdisplay.php?fid=$2&page=$3 last;

3. rewrite ^(.*)/thread-([0-9]+)-([0-9]+)-([0-9]+)\.html$ $1/viewthread.php?tid=$2&extra=page\%3D$4&page=$3 last;

4. rewrite ^(.*)/profile-(username|uid)-(.+)\.html$ $1/viewpro.php?$2=$3 last;

5. rewrite ^(.*)/space-(username|uid)-(.+)\.html$ $1/space.php?$2=$3 last;

6. rewrite ^(.*)/tag-(.+)\.html$ $1/tag.php?name=$2 last;


**给discuz某版块单独配置域名**

1. server_name  bbs.c1gstudio.com news.c1gstudio.com;

3. location = / {

4. if ($http_host ~ news\.c1gstudio.com$) {

5. rewrite ^.+ http://news.c1gstudio.com/forum-831-1.html last;

6. break;

7. }

8. }


**discuz ucenter 头像 rewrite 优化**

01. location ^~ /ucenter {

02. location ~ .*\.php?$

03. {

04. #fastcgi_pass  unix:/tmp/php-cgi.sock;

05. fastcgi_pass  127.0.0.1:9000;

06. fastcgi_index index.php;

07. include fcgi.conf;

08. }

10. location /ucenter/data/avatar {

11. log_not_found off;

12. access_log   off;

13. location ~ /(.*)_big\.jpg$ {

14. error_page 404 /ucenter/images/noavatar_big.gif;

15. }

16. location ~ /(.*)_middle\.jpg$ {

17. error_page 404 /ucenter/images/noavatar_middle.gif;

18. }

19. location ~ /(.*)_small\.jpg$ {

20. error_page 404 /ucenter/images/noavatar_small.gif;

21. }

22. expires 300;

23. break;

24. }

25. }


**jspace rewrite**

01. location ~ .*\.php?$

02. {

03. #fastcgi_pass  unix:/tmp/php-cgi.sock;

04. fastcgi_pass  127.0.0.1:9000;

05. fastcgi_index index.php;

06. include fcgi.conf;

07. }

09. location ~* ^/index.php/

10. {

11. rewrite ^/index.php/(.*) /index.php?$1 break;

12. fastcgi_pass  127.0.0.1:9000;

13. fastcgi_index index.php;

14. include fcgi.conf;

15. }


**wordpress rewrite**

01. location / {

02. index index.html index.php;

03. if (-f $request_filename/index.html){

04. rewrite (.*) $1/index.html break;

05. }

06. if (-f $request_filename/index.php){

07. rewrite (.*) $1/index.php;

08. }

09. if  (!-e $request_filename)

10. {

11. rewrite (.*) /index.php;

12. }

13. }