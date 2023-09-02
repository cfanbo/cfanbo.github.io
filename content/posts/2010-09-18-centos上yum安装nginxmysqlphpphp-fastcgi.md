---
title: CentOS上yum安装nginx+mysql+php+php-fastcgi
author: admin
type: post
date: 2010-09-18T14:18:46+00:00
url: /archives/5750
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - nginx

---
**一、更改yum源为网易的源加快速度**

vi /etc/yum.repos.d/CentOS-Base.repo
更改内容如下

01. # CentOS-Base.repo

02. #

03. # This file uses a new mirrorlist system developed by Lance Davis for CentOS.

04. # The mirror system uses the connecting IP address of the client and the

05. # update status of each mirror to pick mirrors that are updated to and

06. # geographically close to the client. You should use this for CentOS updates

07. # unless you are manually picking other mirrors.

08. #

09. # If the mirrorlist= does not work for you, as a fall back you can try the

10. # remarked out baseurl= line instead.

11. #

12. #

13. [base]

14. name=CentOS-$releasever – Base

15. #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os

16. #baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/

17. baseurl=http://mirrors.163.com/centos/$releasever/os/$basearch/

18. gpgcheck=1

19. gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-5

20. #released updates

21. [updates]

22. name=CentOS-$releasever – Updates

23. #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates

24. #baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/

25. baseurl=http://mirrors.163.com/centos/$releasever/updates/$basearch/

26. gpgcheck=1

27. gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-5

28. #packages used/produced in the build but not released

29. [addons]

30. name=CentOS-$releasever – Addons

31. #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=addons

32. #baseurl=http://mirror.centos.org/centos/$releasever/addons/$basearch/

33. baseurl=http://mirrors.163.com/centos/$releasever/addons/$basearch/

34. gpgcheck=1

35. gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-5

36. #additional packages that may be useful

37. [extras]

38. name=CentOS-$releasever – Extras

39. #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras

40. #baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/

41. baseurl=http://mirrors.163.com/centos/$releasever/extras/$basearch/

42. gpgcheck=1

43. gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-5

44. #additional packages that extend functionality of existing packages

45. [centosplus]

46. name=CentOS-$releasever – Plus

47. #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus

48. #baseurl=http://mirror.centos.org/centos/$releasever/centosplus/$basearch/

49. baseurl=http://mirrors.163.com/centos/$releasever/centosplus/$basearch/

50. gpgcheck=1

51. enabled=0

52. gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-5


**二、update yum**

1. yum -y update

**三、利用CentOS Linux系统自带的yum命令安装、升级所需的程序库**

1. LANG=C

2. yum -y install gcc gcc-c++ autoconf libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers


**四、安装php和mysql**

1. yum -y install php mysql mysql-server mysql-devel php-mysql php-cgi php-mbstring php-gd php-fastcgi


**五、安装nginx**
由于centos没有默认的nginx软件包，需要启用REHL的附件包

1. rpm -Uvh http://download.fedora.redhat.com/pub/epel/5/i386/epel-release-5-4.noarch.rpm

2. yum -y install nginx


设置开机启动

1. chkconfig nginx on


**六、安装spawn-fcgi来运行php-cgi
**

1. yum install spawn-fcgi


**七、下载spawn-fcgi 的启动脚本**

1. wget http://bash.cyberciti.biz/dl/419.sh.zip

2. unzip 419.sh.zip

3. mv 419.sh /etc/init.d/php_cgi

4. chmod +x /etc/init.d/php_cgi


启动php_cgi

1. /etc/init.d/php_cgi start


查看进程

1. netstat -tulpn | grep :9000


若出现如下代表一切正常

1. tcp 0 0 127.0.0.1:9000 0.0.0.0:* LISTEN 4352/php-cgi


**八、配置nginx(详细配置见nginx.conf详细说明)**

1. location ~ \.php$ {

2. root html;

3. fastcgi_pass 127.0.0.1:9000;

4. fastcgi_index index.php;

5. fastcgi_param SCRIPT_FILENAME /usr/share/nginx/html$fastcgi_script_name;

6. include fastcgi_params;

7. }


**九、查看phpinfo**
编写脚本

1. phpinfo();


**十、安装phpmyadmin
** 修改/var/lib/php/session的权限和nginx和php_cgi一致

1. chown -R www.www /var/lib/php/session


FPM安装方式： [CentOS下安装lnmp(Nginx+PHP+MySQL)fpm](http://blog.haohtml.com/index.php/archives/5732)