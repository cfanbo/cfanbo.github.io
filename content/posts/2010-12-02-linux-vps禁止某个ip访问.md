---
title: Linux VPS禁止某个IP访问
author: admin
type: post
date: 2010-12-02T09:00:58+00:00
url: /archives/6817
IM_data:
 - 'a:1:{s:75:"http://www.vpser.net/wp-includes/js/tinymce/plugins/wordpress/img/trans.gif";s:65:"http://blog.haohtml.com/wp-content/uploads/2011/03/e87e_trans.gif";}'
IM_contentdowned:
 - 1
categories:
 - 服务器

---
今天在查看 [VPS侦探](http://www.vpser.net/) 的 [VPS](http://www.diavps.cn/client/aff.php?aff=002) 的SSH登录记录吓了一跳，居然与几个IP连续登录SSH字典猜root密码，我很生气，后果很严重，GFW掉他们，现公布他们的名单：

62.75.214.93  gera125.server4you.de  德国/德国鬼子

203.215.252.189  香港特别行政区/无语。。。。

219.143.200.169  北京市电信 /在党中央还做坏事。。。。

60.12.193.134  浙江省湖州市网通  /

c953dc2c.virtua.com.br  201.83.220.44 巴西 /就你最多。。。。

其中几个还搭建了Nginx的环境，都没做站。

/etc/hosts.allow和/etc/hosts.deny两个文件是控制远程访问设置的，通过他可以允许或者拒绝某个ip或者ip段的客户访问linux的某项服务。![](http://www.vpser.net/wp-includes/js/tinymce/plugins/wordpress/img/trans.gif)

如果请求访问的主机名或IP不包含在/etc/hosts.allow中，那么tcpd进程就检查/etc/hosts.deny。看请求访问的主机名或IP有没有包含在hosts.deny文件中。如果包含，那么访问就被拒绝；如果既不包含在/etc/hosts.allow中，又不包含在/etc/hosts.deny中，那么此访问也被允许。

:[:::…]

daemon list     服务进程名列表，如telnet的服务进程名为in.telnetd
client list     访问控制的客户端列表，可以写域名、主机名或网段，如.trubolinux.com.cn或者192.168.1.
option          可选选项，这里可以是某些命令，也可以是指定的日志文件

例子：hosts.allow
in.telnetd:.vpser.net
vsftpd:192.168.0.
sshd:192.168.0.0/255.255.255.0

/etc/hosts.allow里第一行vpser.net表示，只有vpser.net这个域里的主机允许访问TELNET服务，注意vpser.net前面的那个点(.)。
/etc/hosts.allow里第二行表示，只有192.168.0这个网段的用户允许访问FTP服务，注意0后面的点(.)。
/etc/hosts.allow里第三行表示，只有192.168.0这个网段的用户允许访问SSH服务，注意这里不能写为192.168.0.0/24。虽然在CISCO路由器种这两中写法是等同的。

在/etc/hosts.deny里加上：

sshd:62.75.214.93
sshd:203.215.252.189
sshd:219.143.200.169
sshd:60.12.193.134
sshd:201.83.220.44
sshd:c953dc2c.virtua.com.br
sshd:gera125.server4you.de

把他们访问SSH的全部给拒绝了，Linux的GFW也很强。

本文系： [VPS侦探](http://www.vpser.net/) 原创文章，转载请注明出处。