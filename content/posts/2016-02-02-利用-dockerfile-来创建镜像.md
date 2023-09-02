---
title: 利用 Dockerfile 来创建镜像
author: admin
type: post
date: 2016-02-02T09:56:55+00:00
url: /archives/16568
categories:
 - 服务器
tags:
 - docker
 - dockerfile

---

使用 `docker commit` 来扩展一个镜像比较简单，但是不方便在一个团队中分享。我们可以使用 `docker build` 来创建一个新的镜像。为此，首先需要创建一个 Dockerfile，包含一些如何创建镜像的指令。

新建一个目录和一个 Dockerfile

 $ mkdir sinatra
 $ cd sinatra
 $ touch Dockerfile


Dockerfile 中每一条指令都创建镜像的一层，例如：

 # This is a comment
 FROM ubuntu:14.04
 MAINTAINER Docker Newbee
 RUN apt-get -qq update
 RUN apt-get -qqy install ruby ruby-dev
 RUN gem install sinatra


Dockerfile 基本的语法是

 * 使用`#`来注释
 * `FROM` 指令告诉 Docker 使用哪个镜像作为基础
 * 接着是维护者的信息
 * `RUN`开头的指令会在创建中运行，比如安装一个软件包，在这里使用 apt-get 来安装了一些软件

编写完成 Dockerfile 后可以使用 `docker build` 来生成镜像。

 $ sudo docker build -t="ouruser/sinatra:v2" .
 Uploading context 2.56 kB
 Uploading context
 Step 0 : FROM ubuntu:14.04
 ---> 99ec81b80c55
 Step 1 : MAINTAINER Newbee
 ---> Running in 7c5664a8a0c1
 ---> 2fa8ca4e2a13
 Removing intermediate container 7c5664a8a0c1
 Step 2 : RUN apt-get -qq update
 ---> Running in b07cc3fb4256
 ---> 50d21070ec0c
 Removing intermediate container b07cc3fb4256
 Step 3 : RUN apt-get -qqy install ruby ruby-dev
 ---> Running in a5b038dd127e
 Selecting previously unselected package libasan0:amd64.
 (Reading database ... 11518 files and directories currently installed.)
 Preparing to unpack .../libasan0_4.8.2-19ubuntu1_amd64.deb ...
 Setting up ruby (1:1.9.3.4) ...
 Setting up ruby1.9.1 (1.9.3.484-2ubuntu1) ...
 Processing triggers for libc-bin (2.19-0ubuntu6) ...
 ---> 2acb20f17878
 Removing intermediate container a5b038dd127e
 Step 4 : RUN gem install sinatra
 ---> Running in 5e9d0065c1f7
 . . .
 Successfully installed rack-protection-1.5.3
 Successfully installed sinatra-1.4.5
 4 gems installed
 ---> 324104cde6ad
 Removing intermediate container 5e9d0065c1f7
 Successfully built 324104cde6ad


其中 `-t` 标记来添加 tag，指定新的镜像的用户信息。

“.” 是 Docker daemon构建上下文使用的环境目录（当前目录），而不是Dockerfile 所在的路径，对于Dockerfile文件路径是使用 -f 参数指定。参考： [https://blog.haohtml.com/archives/18711](https://blog.haohtml.com/archives/18711)

可以看到 build 进程在执行操作。它要做的第一件事情就是上传这个 Dockerfile 内容，上传到哪里呢，根据Docker 构架图所求，是上传到 Docker daemon了。因为所有的操作都要依据 Dockerfile 来进行。 然后，Dockfile 中的指令被一条一条的执行。每一步都创建了一个新的容器，在容器中执行指令并提交修改（就跟之前介绍过的 `docker commit` 一样）。当所有的指令都执行完毕之后，返回了最终的镜像 id。所有的中间步骤所产生的容器都被删除和清理了。

*注意一个镜像不能超过 127 层

此外，还可以利用 `ADD` 命令复制本地文件到镜像；用 `EXPOSE` 命令来向外部开放端口；用 `CMD` 命令来描述容器启动后运行的程序等。例如

 # put my local web site in myApp folder to /var/www
 ADD myApp /var/www
 # expose httpd port
 EXPOSE 80
 # the command to run
 CMD ["/usr/sbin/apachectl", "-D", "FOREGROUND"]


现在可以利用新创建的镜像来启动一个容器。

 $ sudo docker run -t -i ouruser/sinatra:v2 /bin/bash
 root@8196968dac35:/#


其中， `-t` 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， `-i` 则让容器的标准输入保持打开。

还可以用 `docker tag` 命令来修改镜像的标签。

 $ sudo docker tag 5db5f8471261 ouruser/sinatra:devel
 $ sudo docker images ouruser/sinatra
 REPOSITORY TAG IMAGE ID CREATED VIRTUAL SIZE
 ouruser/sinatra latest 5db5f8471261 11 hours ago 446.7 MB
 ouruser/sinatra devel 5db5f8471261 11 hours ago 446.7 MB
 ouruser/sinatra v2 5db5f8471261 11 hours ago 446.7 MB


*注：更多用法，请参考 [Dockerfile](https://yeasy.gitbooks.io/docker_practice/content/dockerfile/index.html) 章节。

如果需要修改镜像的话请参考： [https://blog.csdn.net/ling811/article/details/53817123](https://blog.csdn.net/ling811/article/details/53817123)![](https://blog.haohtml.com/wp-content/uploads/2019/01/docker-architecture.jpg)Docker构架