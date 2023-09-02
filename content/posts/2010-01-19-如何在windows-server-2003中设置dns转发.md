---
title: 如何在Windows server 2003中设置DNS转发
author: admin
type: post
date: 2010-01-19T09:25:29+00:00
excerpt: |
 发技术是用于解决那些在本地DNS服务器解决不了的查询工作。这个过程涉及到一个DNS服务器与其他DNS服务器直接通信的问题。当本地DNS与外部 DNS服务器直接进行通信时，将简化存储在DNS服务器缓冲区中非本地站点的信息。这样将可以使得DNS服务器在解决本地查询工作时的效率会更高。

 在Windows 2003之前发行的版本中，对于所有不能解决的查询工作可以指定转发给多个服务器。如果其中一个服务器不可用，这个请求就可以转发到列表中的下一个服务器。这在Windows2003中称为标准转发。
url: /archives/2838
IM_data:
 - 'a:5:{s:74:"http://www.zdnet.com.cn/i/techupdate/images/200401/r00220031203lel01_a.gif";s:79:"http://blog.haohtml.com/wp-content/uploads/2011/03/cce9_r00220031203lel01_a.gif";s:74:"http://www.zdnet.com.cn/i/techupdate/images/200401/r00220031203lel01_b.gif";s:79:"http://blog.haohtml.com/wp-content/uploads/2011/03/e0d8_r00220031203lel01_b.gif";s:74:"http://www.zdnet.com.cn/i/techupdate/images/200401/r00220031203lel01_c.gif";s:79:"http://blog.haohtml.com/wp-content/uploads/2011/03/6b0a_r00220031203lel01_c.gif";s:74:"http://www.zdnet.com.cn/i/techupdate/images/200401/r00220031203lel01_d.gif";s:79:"http://blog.haohtml.com/wp-content/uploads/2011/03/8d28_r00220031203lel01_d.gif";s:74:"http://www.zdnet.com.cn/i/techupdate/images/200401/r00220031203lel01_e.gif";s:79:"http://blog.haohtml.com/wp-content/uploads/2011/03/61be_r00220031203lel01_e.gif";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - dns

---
发技术是用于解决那些在本地DNS服务器解决不了的查询工作。这个过程涉及到一个DNS服务器与其他DNS服务器直接通信的问题。当本地DNS与外部 DNS服务器直接进行通信时，将简化存储在DNS服务器缓冲区中非本地站点的信息。这样将可以使得DNS服务器在解决本地查询工作时的效率会更高。

在Windows 2003之前发行的版本中，对于所有不能解决的查询工作可以指定转发给多个服务器。如果其中一个服务器不可用，这个请求就可以转发到列表中的下一个服务器。这在Windows2003中称为标准转发。

条件转发的定义如下：对于一个特定域的DNS查询信息将向何处转发。这是Windows 2003 DNS 的一个可用的新特征。这种设计的目的是为了实现在多个DNS分区环境中工作，在这种情况下，一个名字空间（例如prep.com）中的系统能够和另一个名 字空间（例如test.com）中的系统进行通信。当然，这也可以作为企业内部互联网（两个公司合并的情况）或者互联网中特定场合（与一个商业伙伴共享数 据）的解决方案。

**设置标准转发**
1、点击“开始”菜单中的“管理工具”中的“DNS”，访问DNS服务。DNS管理工具的主屏幕如图A所示。

![](http://www.zdnet.com.cn/i/techupdate/images/200401/r00220031203lel01_a.gif)

1、在左面板的树层次上选定你的DNS服务器的名字。
2、单击鼠标右键，然后选择“属性”。DNS属性的对话框如图B所示。

![](http://www.zdnet.com.cn/i/techupdate/images/200401/r00220031203lel01_b.gif)

注意：在DNS域名中的“所有其他DNS域名”也都被选定了。当选定这个标准时，所有被添加为转发器的服务器也将用于标准转发（或者也可以用于没有定义条件转发但是包含此域信息的所有其他查询）

1、在 “选定的域转发器IP地址列表”中输入服务器的IP地址，然后点击“添加”，对应这个IP地址的服务器将作为转发DNS查询的服务器。用于转发的服务器将按照添加的先后顺序存放在列表中。（图C）。

![](http://www.zdnet.com.cn/i/techupdate/images/200401/r00220031203lel01_c.gif)

可以选定列表中的IP地址并点击“上移”或者“下移”来适当的调整整个列表顺序。

1、点击DNS属性对话框下面的“确定”按钮结束设置。

**设置条件转发**
在这种情况下，假设两个公司——每个公司都有自己的网络——合并了。他们想要保持各自的网络配置，但是每个网络的域名服务器只能解析自己内部的网络。而有 一些应用程序和行为要求每个网络的域名服务器能够解析另一个网络的域名。这就是条件转发的最完美的情形。

1、点击DNS服务器属性的“转发”标签（首先执行上一节中的步骤1到步骤3）。
2、点击在DNS域名列表附近的“新列表”。将出现新的转发器的对话框，如图D所示。

![](http://www.zdnet.com.cn/i/techupdate/images/200401/r00220031203lel01_d.gif)

1、在“DNS域”中输入域名服务器，需要转发的查询将被转发到这个服务器，点击“确定”按钮。这个域名就被添加到“转发器”标签下的DNS域名列表中（图E）。

![](http://www.zdnet.com.cn/i/techupdate/images/200401/r00220031203lel01_e.gif)

1、为维护本地域名的内部DNS服务器添加IP地址。对于标准转发，在一个域中可以添加多个转发器。
2、点击DNS属性对话框下的“确定” 按钮结束设置。

**设置条件转发的其他提示和技巧**

条 件转发只能用于那些DNS服务器中没有一级或者二级区（zones）的域。这意味着条件转发可以用于具有与本地维护级别相同的区或者级别更高的区的域中。 例如，可以对test.com、corp.test.com或者prep.com设置条件转发，而example.test.com是在DNS服务器上维 护的本地分区。无论如何，在这种情况下，你都无法对one.example.test.com 和example.test.com 设置条件转发。

对于容错性，建议设置条件转发时，每个条件转发都定义多个服务器作为转发器。如果其中一个服务器不可用，那么还有另一个服务器可能可用。

条件转发可以替代以前版本的Windows DNS服务中的二级区（secondary zones），这个第二分区用于解决其他名字空间中的查询问题。