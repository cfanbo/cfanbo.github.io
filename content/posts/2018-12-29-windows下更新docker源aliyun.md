---
title: windows下更新docker源(aliyun)
author: admin
type: post
date: 2018-12-29T07:02:52+00:00
url: /archives/18672
categories:
 - 服务器
tags:
 - docker

---
每个aliyun账号都有一个专属的镜像源

我这里安装的是 Docker Toolbox 软件，更新docker源有两种情况，一种是你还没有创建过Docker Machine，另一种是你已经创建过了Docker Machine。

## 一、未安装过 

#### 创建一台安装有Docker环境的Linux虚拟机，指定机器名称为default，同时配置Docker加速器地址。 

$ docker-machine create –engine-registry-mirror=https://xxxx.mirror.aliyuncs.com -d virtualbox default

#### 查看机器的环境配置，并配置到本地。然后通过Docker客户端访问Docker服务。 

$ docker-machine env default
$ eval “$(docker-machine env default)”
$ docker info

这里 xxxx 是您的专有加速器地址

##
二、已安装过 

登录已创建的Docker VM
$ docker-machine ssh default
$ sudo vi /var/lib/boot2docker/profile

#### 在EXTRA_ARGS中添加 

–registry-mirror=https://xxxx.mirror.aliyuncs.com

```
EXTRA_ARGS='
--label provider=virtualbox
--registry-mirror=https://fgxrbz510.mirror.aliyuncs.com
'
CACERT=/var/lib/boot2docker/ca.pem
DOCKER_HOST='-H tcp://0.0.0.0:2376'
DOCKER_STORAGE=aufs
DOCKER_TLS=auto
SERVERKEY=/var/lib/boot2docker/server-key.pem
SERVERCERT=/var/lib/boot2docker/server.pem

export "NO_PROXY=192.168.99.100"
```

#### 重启Docker服务 

$ sudo /etc/init.d/docker restart

有时候会提示以下错误，可以忽略。如果在连接终端执行 docker info 提示服务未启动的话，可以尝试将虚拟机手动重启一下基本可以解决，我这里用的是virtualbox

> Stopping dockerd (3590)
>
> warning: ‘aufs’ is not a supported storage driver for this boot2docker install — ignoring request!
>
>
> Stopping dockerd (3590)
>
>  warning: ‘aufs’ is not a supported storage driver for this boot2docker install — ignoring request!
>
>  see https://github.com/boot2docker/boot2docker/issues/1326 for more details
>
>  Starting dockerd

另外对于windows上使用Docker Toolbox (VirtualBox)的用户需要注意，当创建一个容器的时候，并指定了端口映射的时候，在宿主本机没有办法通过端口访问容器实例的。因为参数 -p 33065:3306 中的33065指的是VirtualBox中的端口，而非当前宿主机器的端口，此时如果用netstat查看的话，是看不到33065端口的。想访问容器，需要在宿主机器与VirtualBox之间再映射一个端口 **33065:33065** 。

至于原因吗，很好理解的，docker运行需要使用Linux内核，于是windows使用VirtualBox 搞了一个linux 虚拟机器，所以对于docer run 命令指定端口的时候，其实指定的是 linux服务器与容器的端口映射关系。