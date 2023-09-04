---
title: 'Apache和Nginx下禁止访问*.txt文件'
author: admin
type: post
date: 2009-03-16T14:41:53+00:00
excerpt: |
 大家是否测试Apache做了目录禁止浏览后，目录下面的txt文件还是可以显示里面的内容的。（我的是这样的）
 例如：http://www.domain.com/test/此访问会报403错误，但是如果test下有很多txt，你访问该txt时；
 例如：http://www.domain.com/test/a.txt，此时a.txt里的内容会全部暴露在外面了（有时这个txt是很机密的文件），这样以来问题就来了。
 同样：我在Nginx配置后后也存在这样的问题，Apache下此问题的解决多谢NetSeek帮助。
 如下是关于Apache和Nginx 限制该类事情办法：
 Apache：解决办法；

 Options -Indexes FollowSymLinks
 AllowOverride All

 Order allow,deny
 Deny from all

url: /archives/1090
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - nginx

---

大家是否测试Apache做了目录禁止浏览后，目录下面的txt文件还是可以显示里面的内容的。（我的是这样的）

例如： [http://www.domain.com/test/](http://www.domain.com/test/) 此访问会报403错误，但是如果test下有很多txt，你访问该txt时；

例如： [http://www.domain.com/test/a.txt](http://www.domain.com/test/a.txt)，此时a.txt里的内容会全部暴露在外面了（有时这个txt是很机密的文件），这样以来问题就来了。

同样：我在Nginx配置后后也存在这样的问题，Apache下此问题的解决多谢NetSeek帮助。

如下是关于Apache和Nginx 限制该类事情办法：

Apache：解决办法；

```
Options -Indexes   FollowSymLinks

AllowOverride All

         Order allow,deny

         Deny from all

Nginx：解决办法；

location ~* .(txt|doc)$ {

                if (-f $request_filename) {

                   root /home/domain/public_html/test;

                   break;


                   }
}
```




Nginx下请大家注意标点符号的使用，不要漏掉后面的“;”！

转自： [http://www.linuxtone.org/redirect.php?tid=894](http://www.linuxtone.org/redirect.php?tid=894)