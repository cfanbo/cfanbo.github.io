---
title: 'centos安装redmine项目管理系统[教程]'
author: admin
type: post
date: 2012-08-12T17:24:16+00:00
url: /archives/13282
categories:
 - 程序开发
tags:
 - rebyGems
 - redmine
 - ruby

---
这里操作系统为Linux Centos5,参考文档： [http://www.redmine.org/projects/redmine/wiki/HowTo_install_Redmine_on_CentOS_5](http://www.redmine.org/projects/redmine/wiki/HowTo_install_Redmine_on_CentOS_5)

### 另外网上也有一键安装的软件，官方网站为：

### Ruby & Ruby on Rails & Rack

The required Ruby and Ruby on Rails versions for a given Redmine version is:

 Redmine version

 Supported Ruby versions

 Required Rails version

 Supported RubyGems versions

 current trunk

 ruby 1.8.7, 1.9.2, 1.9.3, jruby-1.6.7

 Rails 3.2.6

 RubyGems <= 1.8

 2.0.3

 ruby 1.8.7, 1.9.2, 1.9.3, jruby-1.6.7

 Rails 3.2.6

 RubyGems <= 1.8

 2.0.2

 ruby 1.8.7, 1.9.2, 1.9.3, jruby-1.6.7

 Rails 3.2.5

 RubyGems <= 1.8

 2.0.0, 2.0.1

 ruby 1.8.7, 1.9.2, 1.9.3, jruby-1.6.7

 Rails 3.2.3

 RubyGems <= 1.8

 1.4.x

 ruby 1.8.7, 1.9.2, 1.9.3, jruby-1.6.7

 Rails 2.3.14

 RubyGems <= 1.8


这里我们已经安装好了nginx,mysql环境了。

**一. 安装依赖包**

[shell]yum -y install zlib-devel curl-devel openssl-devel apr-devel apr-util-devel[/shell]

在做Ruby on rail开发环境的时候，发现ruby有了yaml库需求，如果不进行前置安装yaml库，那么在进行接下来的rubygems和rails的时候就会出现如下错误：“It seems your ruby installation is missing psych (for YAML output). To eliminate this warning, please install libyaml and reinstall your ruby.”
注意：请勿使用yum去更新libyaml-devel和libyaml

安装libyaml库

[shell]wget -c http://pyyaml.org/download/libyaml/yaml-0.1.4.tar.gz
tar xzvf yaml-0.1.4.tar.gz
cd yaml-0.1.4
./configure –prefix=/usr/local #注意此处勿改路径！否则库文件无法写入正确目录
make&&make install
cd ../[/shell]

**二. 安装ruby**

这里安装最新版本

[shell]wget http://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.3-p194.tar.gz
tar zxvf ruby-1.9.3-p194.tar.gz
cd ruby-1.9.3-p194
./configure –prefix=/usr/local –enable-shared –disable-install-doc –with-opt-dir=/usr/local/lib
make
make install
cd ../[/shell]

检查ruby版本号

[shell]ruby -v[/shell]

**三. 安装rebyGems**

[shell]wget http://production.cf.rubygems.org/rubygems/rubygems-1.8.24.tgz
tar zxvf rubygems-1.8.24.tgz
cd rubygems-1.8.24
ruby setup.rb
gem -v
cd ../[/shell]

查看更多帮助

[shell]ruby setup.rb –help[/shell]

更新版本号，这里直接跳过了。因为安装的已经是最新版本的了。有时候新版本并不一定稳定和兼容的，呵呵。

[shell]gem update –system[/shell]

或者：

[shell]$ gem install rubygems-update # again, might need to be admin/root
$ update_rubygems # … here too[/shell]

详细请参考： [http://rubygems.org/pages/download](http://rubygems.org/pages/download)

－－－－－－－－－－－－－－－－

**四. 下载redmine2**
打开下载列表页，http://rubyforge.org/frs/?group_id=1850，会显示所有版本的redmine。这里下载redmine 2.0.3版本.(目前已有新版本,请自行下载)

[shell]wget http://rubyforge.org/frs/download.php/76259/redmine-2.0.3.tar.gz
tar zxvf redmine-2.0.3.tar.gz
mv redmine-2.0.3 redmine
cd /data/wwwroot/redmine

gem install bundler
bundle install –without development test rmagick postgresql sqlite[/shell]

**五. 初始化数据库**
1)在phpmyadmin里创建redmine数据库和数据库账户和密码,注意如果密码为数字类型的话，需要用引号括住才可以的。不然会提示“

> rake aborted!
> can’t convert Fixnum into String”

错误。

[shell]create database redmine character set utf8;
grant all privileges on redmine.* to ‘redmine’@’localhost’ identified by ‘my_password’;[/shell]

2)修改数据库配置文件，这里使用的是mysql数据库,由于ruby的版本为1.9。所有adapter要为mysql2, 如果版本为1.8的话，由直接写mysql即可。

> cp config/database.yml.example config/database.yml
>
> vi config/database.yml
> production:
> adapter: mysql2
> database: redmine
> host: localhost
> username: redmine
> password: my_password

**六。生成会员存储密码**

[shell]rake generate\_secret\_token[/shell]

**七。初始化数据库**

[shell]RAILS_ENV=production rake db:migrate
RAILS\_ENV=production rake redmine:load\_default_data[/shell]

**八. 配置redmine**
配合的配置文件为 configuration.yml

[shell]cp config/configuration.yml.example config/configuration.yml[/shell]

**九. 目录权限**

[shell]addgroup redmine
adduser redmine -g redmine
chown -R redmine:redmine files log tmp public
chmod -R 755 files log tmp public[/shell]

> 一般可以设置为www:www用户。

**十. 测试WEBrick web server**

[shell]ruby script/rails server webrick -e production[/shell]

到这里已经全部安装完成。在浏览器里打开ip:3000即可看到redmine的界面。如果看不到界面，请检查防火墙问题。只需要将3000端口开放就可以了。或者直接将防火墙关闭也可以。

如果在局域网用ip地址访问的话，会发现特别的慢，这是由于Redmine自带的WebrickWeb发布的问题，需要使用Mongrel组件来替换Webrick.解决办法请参考：
但这样只是以独立的方式启动redmine的服务器，在后台执行，有些不足，因为客户端的访问日志会在终端上直接显示。并且你退出终端时，服务器进程也会跟着关闭，如果希望Redmine作为服务运行，加上-d参数即可：

[shell]ruby script/rails server webrick -e production -d[/shell]



[shell]vi redmine_start.sh 把脚本加入到 rc.local
#!/bin/bash
/data/wwwroot/redmine/script/rails server webrick -e production -d[/shell]

初始化用户名和密码全为admin.默认语言为english,在settings->display->Default language 里修改成“**简体中文**”就可以了。不现用户可以选择使用不同的显示语言，如果要修改自己的显示语言的话，只需要在个人账户里修改就可以了。

**Redmine里邮件配置：**

vi config/configuration.yml

> default:
> \# Outgoing emails configuration (see examples above)
> email_delivery:
> delivery_method: :smtp
> smtp_settings:
> address: smtp.qq.com
> port: 25
> domain: qq.com
> authentication: :login
> user_name: “123456@qq.com”
> password: “blog.haohtml.com”

保存即可。记得如果服务已经启用过的话，先 kill -9 进程号 杀掉，再启用服务。

对于准备开始使用redmine管理项目的时候，一定要注意项目信息的安全性。一般情况下项目信息只允许项目开发组相关人员才有查看操作权限，游客是不允许查看的。

[![redmine_purview](/wp-content/uploads/2012/08/redmine_purview.png?t=1)][1]
**相关教程：**
[redmine与git组合配置：http://blog.haohtml.com/archives/14771][2]
[Redmine局域网访问缓慢问题解决：http://blog.haohtml.com/archives/13272][3]



 [1]: /wp-content/uploads/2012/08/redmine_purview.png
 [2]: http://blog.haohtml.com/archives/14771
 [3]: http://blog.haohtml.com/archives/13272