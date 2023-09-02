---
title: Nagios监控Mysql服务器
author: admin
type: post
date: 2011-08-09T08:20:56+00:00
url: /archives/10953
IM_data:
 - 'a:1:{s:60:"http://images.51cto.com/files/uploadimg/20110323/1007290.jpg";s:67:"http://blog.haohtml.com/wp-content/uploads/2011/08/64ab_1007290.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - nagios

---
**Nagios-监控Mysql**服务器

监控Mysql需要在nagios和Mysql服务器这两个部分做处理:Mysql服务器安装nrpe、创建Mysql监控用户;配置nagios及用htpasswd创建浏览器验证帐号。下面分步描述。

**一、 在Mysql服务器安装nrpe**

这个操作与nagios服务器安装nrpe基本相同，唯一不同的是nrpe.cfg文件server_address,把它改成Mysql服务器的ip地址即可。检查无误后启动nrpe服务.

**二、创建Mysql访问用户nagios**

这个账号仅仅是nagios监控程序用来访问Mysql数据库所用，与其它帐号毫无关系。为了安全起见，nagios这个账号的权限应该特别低，仅仅有数据库的select权限即可。再进一步，我们创建一个空的数据库nagdb，然后让nagios账号访问这个空库，就可以通过check_Mysql插件测试和监控Mysql数据库。

一切正常以后，Mysql服务器这边的配置和测试就算完成了。

**三、nagios服务器上的操作**

即在nagios配置文件后面追加内容。

(一)、主机配置文件追加Mysql主机定义，联系组contactgroups 的值为sagroup,dbgroup,具体步骤参照前面的操作。

(二)、联系人配置文件(contacts.cfg)追加数据库管理员定义(dba1)，具体步骤参照前面的操作。

(三)、联系组配置文件(contactgroups.cfg)追加数据库管理员组定义(dbgroup)，其成员为联系人配置文件(contacts.cfg)定义的数据库管理员(dba1)。

(四)、服务配置文件(services.cfg)追加Mysql服务监控，除了Mysql服务监控而外，其他几个对象都于前面的类似，只不过联系组多了一个dbgroup。这里列出Mysql服务这个定义：

 1. define service {
 2. host_name nagios-server
 3. service\_description check\_Mysql
 4. check_period 24×7
 5. max\_check\_attempts 4
 6. normal\_check\_interval 3
 7. retry\_check\_interval 2
 8. contact_groups sagroup,dbgroup
 9. notification_interval 10
 10. notification_period 24×7
 11. notification_options w,u,c,r
 12. check\_command check\_Mysql
 13. }

**(五)、命令配置文件**(command.cfg)追加检查Mysql的定义，其追加内容为：

 1. define command {
 2. command\_name check\_Mysql
 3. command\_line $USER1$/check\_Mysql -H $HOSTADDRESS$ -u nagios -d nagdb
 4. }

**(六)、检查并启动nagios**

 1. cd /usr/local/nagios
 2. bin/nagios -v etc/nagios.cfg
 3. bin/nagios -d etc/nagios.cfg

**(七)增加apache验证帐号**

 1. /usr/local/apache/bin/htpasswd /usr/local/nagios/etc/htpasswd db1

输入两次密码后，从别的计算机的浏览器地址栏输入 http://59.26.240.63/nagios ,再输入用户名db1及刚才设定的密码，进入页面后，点击左上方的链接”Service Detail”,就可以看到Mysql服务器当前的运行状态(db1用户只能看到Mysql服务器状态，而管理员sery账号则可以看所有被监控对象的状态)。Nagios监控Mysql服务器OK！

对于一个网站来说，外部用户能够看到就是该网站的页面。网站页面能否被正常访问，以及显示是否正常势必会成为网站整体水平最直接的外在表现。

那么，如何才能在第一时间检测到网页是否正常，并且给相应的技术人员发出报警来及时解决问题，而不是等接到用户抱怨的电话后才在慌忙中仓促的解决问题呢?解决这个问题的关键就是要在第一时间发现问题，发现那些不能显示的网页或是显示不正常的网页，并及时发出报警。当然我们可以通过人工的方法去监测，但对于一些大型的、复杂的网站来说就不是很合适了，我们可以使用监控软件来解决这个问题。我所使用的就是Nagios软件，它提供的插件(Plugins)中有相应的命令可以完成对网页的监控。

======================================

**方式二、通过NRPE监控网页**



[![Nagios/监控](http://images.51cto.com/files/uploadimg/20110323/1007290.jpg)](http://images.51cto.com/files/uploadimg/20110323/1007290.jpg)

方式一Linux下监控网页-Nagios的配置十分简单，只需要在Nagios的配置文件里添加一个服务即可。

**配置内容如下**

修改./etc/objects/commands.cfg，增加如下内容。

 1. #’check_http‘check web page
 2. define command{
 3. command\_name check\_webpage
 4. command\_line $USER1$/check\_http $ARG1$
 5. }

修改./etc/objects/localhost.cfg，增加如下内容。

 1. define host{
 2. uselinux-server
 3. host\_nameweb\_pages
 4. alias web_pages
 5. address 127.0.0.1
 6. }
 7.  #the check web pages on the remote host.
 8.  define service{
 9.  usegeneric-service
 10. host\_name web\_pages;主机名，为了便于显示可以定义一个虚拟的host
 11.  service_description web page1
 12.  check\_command check\_webpage!-H www.testhost.test -u /index.html
 13.  }
 方式二的配置方法略复杂一些，需要修改两台主机的配置文件。

修改NRPE的配置文件，增加如下内容。

 1. #check webpage
 2. command[check\_webpage]=/usr/local/nagios//libexec/check\_http -H www.testhost.test -u /index.html

修改Nagios配置文件，增加如下内容。

 1. #the check_apache on the remote host.
 2. define service{
 3. usegeneric-service
 4. host_namehostname
 5. service_description web page
 6. check\_command check\_nrpe! check_webpage
 7. }

以上仅仅是举个简单的例子来说明，当然实际环境要更复杂、页面要更多，可以通过增加服务(service)的方式将其一一纳入监控范围。