---
title: Nginx下WordPress的永久链接实现
author: admin
type: post
date: 2009-12-10T01:48:49+00:00
url: /archives/2685
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx

---

经过多番测试，终于在nginx下实现了rewrite的功能，WrodPress的永久链接终于生效了。

其实也是很简单的方法，修改nginx.conf文件，加入以下内容：

`location / {

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

`

重启nginx就可以了。

#killall nginx

#nginx