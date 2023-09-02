---
title: windows平台在dos下执行docker pull 命令出错
author: admin
type: post
date: 2018-02-08T04:15:48+00:00
url: /archives/17573
categories:
 - 系统架构
tags:
 - docker

---
**这里docker Machine 是安装和管理 Docker 的工具(用来代替Boot2Docker，对于个人玩的话，不建议使用docker Machine，毕竟多了一个虚拟层，不如直接使用当前物理机器作为容器的宿主机）**

```
$ docker pull gitlab/gitlab-ce:latest
```

> Warning: failed to get default registry endpoint from daemon (error during connect: Get http://%2F%2F.%2Fpipe%2Fdocker\_engine/v1.33/info: open //./pipe/docker\_engine: The system cannot find the file specified. In the default daemon configuration on Windows, the docker client must be run elevated to connect. This error may also indicate that the docker daemon is not running.). Using system default: https://index.docker.io/v1/
> error during connect: Post http://%2F%2F.%2Fpipe%2Fdocker\_engine/v1.33/images/create?fromImage=gitlab%2Fgitlab-ce&tag=latest: open //./pipe/docker\_engine: The system cannot find the file specified. In the default daemon configuration on Windows, the docker client must be run elevated to connect. This error may also indicate that the docker daemon is not running.

解决方法:打开新窗口后执行(

```
docker-machine env
```

根据提示运行命令

```
@FOR /f "tokens=*" %i IN ('docker-machine env default') DO @%i
```

default 是 [docker-machine](http://www.cnblogs.com/sparkdev/p/7044950.html) 的name，可以通过**docker-machine -ls** 查看。

如果还提示这个错误，说明没有启用 default ，执行下面的命令

```
D:\docker
$ docker-machine ls
NAME ACTIVE DRIVER STATE URL SWARM DOCKER ERRORS
default - virtualbox Stopped Unknown

```

如果为stopped状态，则需要启用一下

```
$ docker-machine start default
Starting "default"...
(default) Check network to re-create if needed...
(default) Windows might ask for the permission to configure a dhcp server. Sometimes, such confirmation window is minimized in the taskbar.
(default) Waiting for an IP...
Machine "default" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.

```

最后再执行

```
$ docker pull hub-mirror.c.163.com/gitlab/gitlab-ce:latest
latest: Pulling from gitlab/gitlab-ce
1be7f2b886e8: Pull complete
6fbc4a21b806: Pull complete
c71a6f8e1378: Pull complete
4be3072e5a37: Pull complete
06c6d2f59700: Pull complete
3e236075b07f: Pull complete
9d3aa9a75297: Pull complete
1fad4f75154b: Pull complete
5383bef86102: Pull complete
7343d4ab63b3: Pull complete
7f250f1b8a34: Downloading [==================> ] 157.4MB/415.4MB

```