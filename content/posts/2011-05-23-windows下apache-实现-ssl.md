---
title: Windows下apache 实现 SSL
author: admin
type: post
date: 2011-05-23T03:22:51+00:00
url: /archives/9547
IM_data:
 - 'a:14:{s:65:"http://img1.51cto.com/attachment/200807/200807311217518814046.jpg";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/05/e549_200807311217518814046.jpg";s:65:"http://img1.51cto.com/attachment/200807/200807311217518872656.jpg";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/05/3e0b_200807311217518872656.jpg";s:65:"http://img1.51cto.com/attachment/200807/200807311217519020078.jpg";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/05/cb73_200807311217519020078.jpg";s:65:"http://img1.51cto.com/attachment/200807/200807311217519105328.jpg";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/05/dd73_200807311217519105328.jpg";s:65:"http://img1.51cto.com/attachment/200807/200807311217519171015.jpg";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/05/1c56_200807311217519171015.jpg";s:65:"http://img1.51cto.com/attachment/200807/200807311217519228281.jpg";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/05/73d8_200807311217519228281.jpg";s:65:"http://img1.51cto.com/attachment/200807/200807311217519274015.jpg";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/05/ad71_200807311217519274015.jpg";s:65:"http://img1.51cto.com/attachment/200807/200807311217519323187.jpg";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/05/7ba1_200807311217519323187.jpg";s:65:"http://img1.51cto.com/attachment/200807/200807311217519532484.jpg";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/05/a895_200807311217519532484.jpg";s:65:"http://img1.51cto.com/attachment/200807/200807311217519585546.jpg";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/05/df5e_200807311217519585546.jpg";s:65:"http://img1.51cto.com/attachment/200807/200807311217519637296.jpg";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/05/7d59_200807311217519637296.jpg";s:65:"http://img1.51cto.com/attachment/200807/200807311217519733000.jpg";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/05/b1c9_200807311217519733000.jpg";s:65:"http://img1.51cto.com/attachment/200807/200807311217519792625.jpg";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/05/dcb5_200807311217519792625.jpg";s:65:"http://img1.51cto.com/attachment/200807/200807311217519934937.jpg";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/05/81b5_200807311217519934937.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ssl

---

SSL：安全套接层，是netscape公司设计的主要用于web的安全传输协议。这种协议在WEB上获得了广泛的应用。通过证书认证来确保客户端和网站服务器之间的数据是安全，过程大致如下：


SSL客户端在TCP连接建立之后，发出一个消息给服务器，这个消息里面包含了自己可实现的算法列表和其它一些需要的消息，SSL的服务器端会回应一个数据包，这里面确定了这次通信所需要的算法，然后发过去自己的证书（里面包含了身份和自己的公钥）。Client在收到这个消息后会生成一个秘密消息，用SSL服务器的公钥加密后传过去，SSL服务器端用自己的私钥解密后，会话密钥协商成功，双方可以用同一份会话密钥来通信了。


如果对于一般的应用，管理员只需生成“证书请求”（后缀大多为.csr），它包含你的名字和公钥，然后把这份请求交给诸如verisign等有CA服务公司，你的证书请求经验证后，CA用它的私钥签名，形成正式的证书发还给你。管理员再在web server上导入这个证书就行了。如果你不想花那笔钱，或者想了解一下原理，可以自己做CA。从ca的角度讲，你需要CA的私钥和公钥。从想要证书的服 务器角度将，需要把服务器的证书请求交给CA.

如果你要自己做CA，别忘了客户端需要导入CA的证书（CA的证书是自签名的，导入它意味着你“信任”这个CA签署的证书）。而商业CA的一般不用，因为它们已经内置在你的浏览器中了。


实现HTTPS，经我个人测试，每个版本实现的细节都有所不同，但个人认为最为简单的还是apache_2.2.8-win32-x86-openssl-0.9.8g  那么我下面就以这个版本来介绍一下如何实现基于windows下的https.


**思路：**

1． 配置 apache 以支持 SSL


2． 为网站服务器生成私钥及申请文件


3． 安装CA 使用两种方法


4．通过CA为网站服务器签署证书


5．测试


**步骤1：配置 APACHE以支持SSL**

去掉两行前面的#


> LoadModule ssl_module modules/mod_ssl.so
>
>
> Include conf/extra/httpd-ssl.conf

**步骤2： 为网站服务器生成证书及私钥文件**

生成服务器的私钥


C:\Program Files\Apache Software Foundation\Apache2.2\bin>openssl genrsa -out server.key 1024


![Windows下apache 实现 SSL  - czy4411741 - 混混噩噩](http://img1.51cto.com/attachment/200807/200807311217518814046.jpg)

生成一个server.key


生成签署申请


C:\Program Files\Apache Software Foundation\Apache2.2\bin>openssl req -new –out server.csr -key server.key -config ..\conf\openssl.cnf


![Windows下apache 实现 SSL  - czy4411741 - 混混噩噩](http://img1.51cto.com/attachment/200807/200807311217518872656.jpg)

此时生成签署文件  SERVER.CSR


**步骤3：**

CA方面：


应该是一个专门的CA机构，我们这里就自己在同一台机器搭建一个企业内部CA。


这里可以直接使用商业CA，但要交纳一定的费用，我们来自己动手搭建一个企业内部CA。


我们这里介绍两种方法，一种是使用OPENSSL 另一种是使用WNDOWS系统自带的 CA服务。


我们先看第一种方法，使用OPENSSL


生成CA私钥


> C:\Program Files\Apache Software Foundation\Apache2.2\bin>openssl genrsa  -out ca.key 1024

![Windows下apache 实现 SSL  - czy4411741 - 混混噩噩](http://img1.51cto.com/attachment/200807/200807311217519020078.jpg)

多出CA.key文件


利用CA的私钥产生CA的自签署证书


> C:\Program Files\Apache Software Foundation\Apache2.2\bin>openssl req  -new -x509 -days 365 -key ca.key -out ca.crt  -config ..\conf\openssl.cnf

![Windows下apache 实现 SSL  - czy4411741 - 混混噩噩](http://img1.51cto.com/attachment/200807/200807311217519105328.jpg)

此时生成了一个自己的证书文件，CA就可以工作了，等着生意上门了。


下面准备为网站服务器签署证书


> C:\Program Files\Apache Software Foundation\Apache2.2\bin>openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key -config ..\conf\openssl.cnf

但，此时会报错：


![Windows下apache 实现 SSL  - czy4411741 - 混混噩噩](http://img1.51cto.com/attachment/200807/200807311217519171015.jpg)

所以我们需要先创建以下文件结构用于存放相应文件：


![Windows下apache 实现 SSL  - czy4411741 - 混混噩噩](http://img1.51cto.com/attachment/200807/200807311217519228281.jpg)

再执行一遍，即可生成  server.crt文件


![Windows下apache 实现 SSL  - czy4411741 - 混混噩噩](http://img1.51cto.com/attachment/200807/200807311217519274015.jpg)

然后将  server.crt    server.key复制到  conf文件夹下

重新启动 APACHE即可！


![Windows下apache 实现 SSL  - czy4411741 - 混混噩噩](http://img1.51cto.com/attachment/200807/200807311217519323187.jpg)

但要在IE中导入CA的证书，否则会报告证书不可信任！


实验终于OK！！


============= FOR IIS ====================================================


当然也可以使用WINDOWS的证书服务来为网站服务器签署证书：下面我们就来看一看：


安装成功后会在默认站点下生成certsr虑拟目录


下面我们开始签署过程


先停止APACHE，因为80口在占用。开启IIS的默认网站


![Windows下apache 实现 SSL  - czy4411741 - 混混噩噩](http://img1.51cto.com/attachment/200807/200807311217519532484.jpg)

然后选择高级证书申请


![Windows下apache 实现 SSL  - czy4411741 - 混混噩噩](http://img1.51cto.com/attachment/200807/200807311217519585546.jpg)

![Windows下apache 实现 SSL  - czy4411741 - 混混噩噩](http://img1.51cto.com/attachment/200807/200807311217519637296.jpg)

然后开始颁发即可：


![Windows下apache 实现 SSL  - czy4411741 - 混混噩噩](http://img1.51cto.com/attachment/200807/200807311217519733000.jpg)

![Windows下apache 实现 SSL  - czy4411741 - 混混噩噩](http://img1.51cto.com/attachment/200807/200807311217519792625.jpg)

然后再将  server.key也复制到 conf文件夹下。


重新启动 APACHE，即可！


**步骤4： 重新启动 APACHE即可！**

![Windows下apache 实现 SSL  - czy4411741 - 混混噩噩](http://img1.51cto.com/attachment/200807/200807311217519934937.jpg)

到此,实验终于结束了