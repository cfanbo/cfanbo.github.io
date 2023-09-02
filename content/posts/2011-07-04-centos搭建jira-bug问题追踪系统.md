---
title: centos搭建jira bug问题追踪系统
author: admin
type: post
date: 2011-07-04T03:12:28+00:00
url: /archives/10211
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - centos
 - jira

---
**一. 安装jdk**
参考:

**二. 建立JIRA数据库**

> mysql>create database jiradb character set utf8;
> mysql>grant all on jiradb.* to \`jira\`@\`localhost\` identified by ‘jira’;

**三.JIRA 安装**

> wget http://wpc.29c4.edgecastcdn.net/8029C4/downloads/software/jira/downloads/atlassian-jira-enterprise-4.2.4-b591-standalone.tar.gz
> tar zxvf atlassian-jira-enterprise-4.2.4-b591-standalone.tar.gz
> mv atlassian-jira-enterprise-4.2.4-b591-standalone /usr/local/jira

创建jira.home文件夹

> mkdir -p /usr/local/jira_home

修改vi /usr/local/jira/atlassian-jira/WEB-INF/classes/jira-application.properties 文件,指定jira.home = 路径.要使用绝对路径.

> jira.home = /usr/local/jira_home

注：jira.home文件夹不可以设置为jira根目录及其子目录，jira动态运行时使用和产生的文件都会放在这

修改vi /usr/local/jira/conf/server.xml文件,修改成如下几项:

>  driverClassName=”com.mysql.jdbc.Driver”
> username=”jira”
> password=”jira”
> url=”jdbc:mysql://localhost/jiradb?autoReconnect=true&useUnicode=true&characterEncoding=UTF8”
> maxActive=”20″
> validationQuery=”select 1″

上面username和password是jira使用的mysql数据库用户名和密码.

为了避免与自带的tomcat与原来的tomcat冲突，可以把server.xml里的8080端口改成 8081，删除以下两行

> minEvictableIdleTimeMillis= “4000”
>  timeBetweenEvictionRunsMillis=”5000″

修改vi /usr/local/jira/atlassian-jira/WEB-INF/classes/entityengine.xml文件

查找datasource name

将其中的hsql改成mysql 数据库类型

> hsql”
> 改为：
> mysql“

删除 schema-name=”PUBLIC”,此项只能适应于hsql.

**四、破解**

1.用JiraLicenseStoreImpl.class文件覆盖/usr/local/jira/atlassian-jira/WEB-INF/classes/com/atlassian/jira/license/JiraLicenseStoreImpl.class文件

> cd /usr/local/jira/atlassian-jira/WEB-INF/classes/com/atlassian/jira/license
> mv JiraLicenseStoreImpl.class JiraLicenseStoreImpl.class.bak
> wget 2.用atlassian-extras-2.2.2.jar文件覆盖/usr/local/jira/atlassian-jira/WEB-INF/lib/atlassian-extras-2.2.2.jar包

> cd /usr/local/jira/atlassian-jira/WEB-INF/lib/
> mv atlassian-extras-2.2.2.jar atlassian-extras-2.2.2.jar.bak
> wget

**五.安装汉化包**

停止JIRA,将中文语言包language\_zh\_CN.jar拷贝至/usr/local/jira/atlassian-jira/WEB-INF/lib/目录下,系统默认的改名备份

> /usr/local/jira/bin/shutdown.sh
> cd  /usr/local/jira/atlassian-jira/WEB-INF/lib/
> mv language\_zh\_CN.jar language\_zh\_CN.jar.bak
> wget [http://blog.haohtml.com/down/linux/jira/language\_zh\_CN.jar][1]
> /usr/local/jira/bin/startup.sh

注意:运行或者停止JIRA服务命令有

> /usr/local/jira/bin/catalina.sh start[stop]
> 或者
> /usr/local/jira/bin/startup.sh
> /usr/local/jira/bin/shutdown.sh

**六.WEB 配置 JIRA**

浏览器输入http://localhost:8081就看到jira的主页面了。在首页会看到你的ServerID，比如ServerID为B5EU-IZVX-K1SZ-39HC,那么拷贝如下licence，经过破解之后，可以直接填入如下明文key了：

> #Sun Oct 25 00:50:34 CDT 2009
> Description=JIRA: longmaster
> CreationDate=2010-02-22
> ContactName=zzhcool@126.com
> jira.LicenseEdition=ENTERPRISE
> ContactEMail=zzhcool@126.com
> Evaluation=false
> jira.LicenseTypeName=COMMERCIAL
> jira.active=true
> licenseVersion=2
> MaintenanceExpiryDate=2099-10-24
> Organisation=zzh
> jira.NumberOfUsers=-1
> ServerID=B5EU-IZVX-K1SZ-39HC
> LicenseID=LID
> LicenseExpiryDate=2099-10-24
> PurchaseDate=2010-10-25

ok，你的jira的过期时间就是2099年了.

如果JIRA locked报错,删掉/usr/local/jira_home/.jira-home.lock文件,重启jira即可

 [1]: http://blog.haohtml.com/down/linux/jira/language_zh_CN.jar