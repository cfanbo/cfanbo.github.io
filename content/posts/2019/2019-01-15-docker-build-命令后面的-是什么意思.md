---
title: docker build . 命令后面的.是什么意思
author: admin
type: post
date: 2019-01-15T01:01:18+00:00
url: /archives/18711
categories:
 - 系统架构
tags:
 - docker
 - dockerfile

---
今天来公司自己构建了一个Dockerfile,放在一个经常用到的项目目录里，内容如下：

```
# This is a comment
FROM ubuntu:14.04
MAINTAINER Docker Newbee <newbee@docker.com>
RUN apt-get -qq update
RUN apt-get -qqy install ruby ruby-dev
RUN gem install sinatra
```

然后执行

```
sudo docker build -t "cfanbo/test:v2" .
```

发现在构建的时候发送给 docker daemon 竟然有4G多，超极大。首先的第一反映出问题了。一个ubuntu镜像也没有这么大呀，况且现在还没有开始从远程pull 镜像呢。

那到底什么情况了呢？经过一翻搜索，发现在docker build . 的时候，会将当前目录里的内容发送给 docker daemon。只需要加一个 .dockerignore 文件，将其它内容排除掉就可以了，类似于git中的.gitignore文件的作用。

后面就想通过docker build -h 命令查看一下相关的参数，发现对于Dockerfile 文件位置需要 -f 参数指定，并非上面命令后面的.符号。那就有些疑惑了，原来学习的时候一直把.当作当前目录指定的。难道.有其它的作用，后面查看了一些文档，发现一直对 docker build 命令的过程不太了解。通过查找了一些资料才发现它的真正作用。

Docker 在运行时分为 Docker引擎（服务端守护进程） 以及 客户端工具，我们日常使用各种 docker 命令，其实就是在使用客户端工具与 Docker 引擎 进行交互。

那么当我们使用 docker build 命令来构建镜像时，这个构建过程其实是在 Docker引擎 中完成的，而不是在本机环境。

那么如果在 Dockerfile 中使用了一些 COPY 等指令来操作文件，如何让 Docker引擎 获取到这些文件呢？

这里就有了一个镜像构建上下文的概念，当构建的时候，由用户指定构建镜像的上下文路径，而 docker build 会将这个路径下所有的文件都打包上传给 Docker 引擎，引擎内将这些内容展开后，就能获取到所有指定上下文中的文件了(参考下方docker架构图)。

比如说 dockerfile 中的 COPY ./package.json /project，其实拷贝的并不是本机目录下的 package.json 文件，而是 docker引擎中展开的构建上下文中的文件，所以如果拷贝的文件超出了构建上下文的范围，Docker引擎是找不到那些文件的。![](https://blogstatic.haohtml.com//uploads/2023/09/docker-architecture.jpg)
Docker构架

**所以 docker build . 最后的 . 号，其实是在指定镜像构建过程中的上下文环境的目录。**

理解了上面的这些概念，就更方便的去理解 .dockerignore 文件的作用了。

为验证.的作用，我们可以创建一个空的目录，在目录里执行

```
docker build -f /home/tom/Dockerfile .
```

会发现命令完全正常执行。