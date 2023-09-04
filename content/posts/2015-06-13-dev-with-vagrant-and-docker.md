---
title: Dev with Vagrant and Docker
author: admin
type: post
date: 2015-06-13T15:39:52+00:00
url: /archives/15740
categories:
 - 程序开发
tags:
 - docker
 - Vagrant

---
## 前言

为了在团队里搭建统一的本地开发环境，最近花了点时间用了下vagrant和docker，在此做个记录， 这也算一个DevOps的实践。

* * *

## Vagrant介绍

[Vagrant][1] 是一款用来构建虚拟开发环境的工具，非常适合 php/python/ruby/java 这类语言开发 web 应用，“代码在我机子上运行没有问题”这种说辞将成为历史。

我们可以通过 Vagrant 封装一个 Linux 的开发环境，分发给团队成员。成员可以在自己喜欢的桌面系统（Mac/Windows/Linux）上开发程序，代码却能统一在封装好的环境里运行，非常霸气。

_以上介绍直接抄自[网络][2]，我觉得介绍的很到位。_

**「注意点：」**

vagrant up命令执行后，如果看到下面的错误信息，则需要安装另外一个工具：

> [default] The guest additions on this VM do not match the installed version of
> VirtualBox! In most cases this is fine, but in rare cases it can
> prevent things such as shared folders from working properly. If you see
> shared folder errors, please make sure the guest additions within the
> virtual machine match the version of VirtualBox you have installed on
> your host and reload your VM.
>
> Guest Additions Version: 4.1.18
> VirtualBox Version: 4.3

这是因为Guest Additions 和 VirtualBox的版本不一致所导致的，所以需要安装另外一个工具，保持他们的版本自动同步：

```
$  vagrant plugin install vagrant-vbguest
```



然后重新vagrant up一下就好了（当然需要先vagrant destroy掉之前的）

## Docker介绍

[Docker][3]是一种增加了高级API的LinuX Container（LXC）技术，提供了能够独立运行Unix进程的轻量级虚拟化解决方案。它提供了一种在安全、可重复的环境中自动部署软件的方式。Docker使用标准化容器的概念，能够容纳软件组件及其依赖关系——二进制文件、类库、配置文件、脚本、Virtualenv、jar包、gem包、原始码等——而且可以在任何支持cgroups的64位（针对x64）Linux内核上运行。

Linux Container，是一个轻量级系统虚拟化机制。有时被人称为：“chroot on steroids”，你也可以拿chroot来理解这个概念。而Docker是用go语言实现的Linux Container包装。更详细的介绍，可以自行搜索。

## Github 示例

我在github创建了一个[示例][4]，也可以直接拿来当开发环境，只限于Ruby。

## 详细

关于Vagrant的安装、命令的意义，网上多的是，这里就不做重复介绍了。 我只把github示例做个讲解吧。

**Vagrantfile**

```
# -*- mode: ruby -*-

# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu"
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"

  # 1024 < port < 49151

  #shipyard

  config.vm.network :forwarded_port, guest: 8005, host: 8005

  #apps

  config.vm.network :forwarded_port, guest: 3000, host: 3000
  #elasticsearch

  config.vm.network :forwarded_port, guest: 9200, host: 9200
  config.vm.network :forwarded_port, guest: 9300, host: 9300

  #memcached

  config.vm.network :forwarded_port, guest: 11211, host: 11211

  #mysql

  config.vm.network :forwarded_port, guest: 49153, host: 3306

  #share folder

  config.vm.synced_folder "./vagrant", "/vagrant_data"

  config.vm.provider :virtualbox do |vb|
    vb.gui = false
    #memory

    vb.customize ["modifyvm", :id, "--memory", "1024"]
  end

  config.vm.provision :puppet do |puppet|
    puppet.manifests_path = 'puppet/manifests'
    # puppet.module_path    = 'puppet/modules'

  end

  config.vm.provision "docker" do |d|
    d.pull_images "blackanger/my-mysql-server"
    # d.pull_images "blackanger/elasticsearch"

    # d.pull_images "blackanger/memcached"

    # build

  end

  config.vm.provision :shell, path: "bootstrap.sh"
end
```



基本的架构是这样的：

> Vagrant <-> box <-> VirtualBox[ docker images[containers] ]

Vagrant操作的VirtualBox里面运行着ubuntu， 然后又在ubuntu里安装了docker， 我们可以使用docker image来运行contianers。大家可以去Docker的官网在线体验一下docker的各个命令。

为什么用Vagrant + Docker呢？ 因为我不想在本地开多个Vagrant。

**一切从Vagrantfile开始：**

```
vagrant up
```



当这个命令被执行之后， vagrant会按照Vagrantfile文件所配置的来执行命令。

```
config.vm.network :forwarded_port, guest: 22, host: 2222
```



这个命令会指定本地系统和VirtualBox系统的端口映射，也就是说，我在本地系统访问22端口，就是去访问VirtualBox的2222端口。这样才能让我们和VirtualBox通信。

其他配置，大家看vagrant init生成的Vagrantfile默认说明就清楚了。

```
config.vm.provision :puppet do |puppet|
    puppet.manifests_path = 'puppet/manifests'
end
```



vagrant支持puppet，来帮你自动安装好VirtualBox里需要的各种lib。 这个例子中，我安装了sqlite、rvm。

```
config.vm.provision "docker" do |d|
  d.pull_images "blackanger/my-mysql-server"
end
```



首先，这条命令会帮你在VitualBox里安装docker。 然后 pull_images 命令，会把你上传到index.docker.io的配置好的image pull下来。

```
config.vm.provision :shell, path: "bootstrap.sh"
```



最后，执行一个shell脚本， 把VirtualBox里的docker images都run出来相应的Contianer供我们使用。

```
config.vm.synced_folder "./vagrant", "/vagrant_data"
```



这是设定VirtualBox和本地系统共享的目录， 我在vagrant目录下面放了apps目录， 然后把本地系统开发的Ruby项目做一个link连到vagrant/apps/ 目录下面 (link不工作，可以把开发的项目放到apps目录下)， 就可以直接在VirtualBox里跑我们的Ruby应用了，而且还可以使用我们docker run出来的Contianer。比如我例子里运行的mysql docker container， 就可以在Rails项目里配置好直接使用，感觉就像在vagrant上用docker来做的本地云服务。

**Docker的使用事项**

```
- pull images的时候必须要翻墙。
- 你可以把一个container 的 ID commit到一个image中去，
  然后push到index.docker.io的image仓库中，可以随时pull下来使用。
 （有点和git的用法相似）
- 也可以使用
      sudo docker build -t  <username>/<imagesname> .
  命令从一个Dockerfile文件来构建image。
```



## 总结

还有很多的vagrant和docker命令我都没有说，这些只要去看help就明白了。关于其他使用细节，欢迎大家一起交流。



[1]: http://vagrantup.com/
[2]: http://blog.segmentfault.com/fenbox/1190000000264347
[3]: https://www.docker.io/%E2%80%8E
[4]: https://github.com/Ruby-Study/dh_env