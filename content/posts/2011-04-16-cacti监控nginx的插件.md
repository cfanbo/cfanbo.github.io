---
title: 如何安装cacti监控nginx插件
author: admin
type: post
date: 2011-04-16T17:26:33+00:00
url: /archives/9288
IM_contentdowned:
 - 1
IM_data:
 - 'a:1:{s:72:"http://forums.cacti.net/styles/subsilver2/imageset/icon_topic_attach.gif";s:77:"http://blog.haohtml.com/wp-content/uploads/2011/04/9fe7_icon_topic_attach.gif";}'
categories:
 - 程序开发
tags:
 - CACTI
 - nginx

---
Scripts and templates for nginx.

Nginx –

Provide graphing nginx clients statistics (active, reading, writing, waiting) and nginx socket statistics (accepts, handled, requests). It’s a formal devision used only for graphs usability.

For use do next steps:

1. Enable nginx http\_stub\_status_module at configure stage (if requared).

2. Enable stub status. Add to nginx.conf (in any server context):

>

> location /nginx_status {
>

>
>

> stub_status on;

# disable access_log if requared

access_log   off;

#allow XX.YY.AA.ZZ;

#allow YY.ZZ.JJ.CC;

#deny all;

}
>

Restart nginx.

**3.**

>

> cp get_nginx_clients_status.pl /scripts/

cp get_nginx_socket_status.pl /scripts/

chmod 0755 /scripts/get_nginx_socket_status.pl

chmod 0755 /scripts/get_nginx_clients_status.pl
>

4. Check that it’s work. Run

>

> get_nginx_clients_status.pl http://nginx.server.tld/nginx_status
>

and see that returned the same string:

>

> nginx_accepts:113869 nginx_handled:113869 nginx_requests:122594
>

5. Import to cacti cacti_graph_template_nginx_clients_stat.xml and cacti_graph_template_nginx_sockets_stat.xml.

6. Add nginx graphs to your hosts.

P.S. Sorry for my english

**Attachments:**![](http://forums.cacti.net/styles/subsilver2/imageset/icon_topic_attach.gif)[cacti-nginx.tar.gz](http://forums.cacti.net/download/file.php?id=12676) [5.15 KiB]


官方地址: [http://forums.cacti.net/about26458.html](http://forums.cacti.net/about26458.html) 注意：nginx_sockets这个模板如果只有图，但没有数据的话，可能是 perl脚本问题，手动调整一下就可以了．

说明:如果” no (LWP::UserAgent not found)”错误,请参考解决办法:

下面来安装php-fpm监控插件
默认情况下此功能未开启，编辑php-fpm.conf文件，开启pm.status_path = phpfpm-status　选项,退出保存!

get_php_fpm_status.pl copy to ../scripts directory

cacti_graph_template_php-fpm_pool_status.xml import to cactiScript check url http://server:port/phpfpm-status

[shell]location /phpfpm-status {

allow 10.0.1.58;

deny all;

include fastcgi_params;

fastcgi_pass 127.0.0.1:9000;

fastcgi_param SCRIPT_FILENAME $fastcgi_script_name;

}[/shell]


Thanks for .xml and idea for **Alexander Moskalenko**

http://forum.nginx.org/read.php?25,167900,167900#msg-167900


**Attachments:(php5.2.X)**![](http://forums.cacti.net/styles/subsilver2/imageset/icon_topic_attach.gif)[cacti-phpfpmstatus.zip](http://forums.cacti.net/download/file.php?id=22746) [3.58 KiB]
 **Attachments:[php5.3.x]**![](http://forums.cacti.net/styles/subsilver2/imageset/icon_topic_attach.gif)[get_php_fpm_status.pl](http://forums.cacti.net/download/file.php?id=24186) [1.29 KiB]


官方地址: