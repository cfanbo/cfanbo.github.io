---
title: 使用Dockerfile 多阶段构建Golang 应用
author: admin
type: post
date: 2019-12-31T08:53:29+00:00
url: /archives/19334
categories:
 - 程序开发
tags:
 - docker
 - dockerfile

---
docker在开发和运维中使用的场景越来越多，作为开发人员非常有必要了解一些docker的基本知识，而离我们工作中最近的也就是对应用的docker部署编排了，小到一个dockerfile, docker-compse文件的编写，大到k8s的管理。这里我们以 golang应用为例讲解一些Dockerfile的基本用法，在ci/cd中经常用到这些知识。

# 前提 

**项目清单：**

```
drwxr-xr-x   9 sxf  staff   288 12 31 16:13 .
drwx------@ 17 sxf  staff   544 12 31 14:59 ..
-rw-r--r--   1 sxf  staff    14 12 31 16:09 .dockerignore
drwxr-xr-x  14 sxf  staff   448 12 31 16:21 .git
-rw-r--r--   1 sxf  staff   467 12 31 16:08 Dockerfile
-rw-r--r--   1 sxf  staff    11 12 31 15:01 README.md
-rw-r--r--   1 sxf  staff    84 12 31 15:51 go.mod
-rw-r--r--   1 sxf  staff  3433 12 31 15:51 go.sum
-rw-r--r--   1 sxf  staff   191 12 31 16:02 main.go
```

```
文件说明：
.dockerignore 看名字就知道他的作用是用为忽略一些文件的，它的使用主要是在Dockerfile中使用COPY/ADD 指令时发挥作用。以行为单位，这里共两行，行内容分别是.git 和 README.md
.git 这个是项目Git仓库
Dockerfile 我们文章的重点
go.mod Golang启用了模块管理功能
go.sum 启用模块管理时，会在此文件中记录依赖的三方库
main.go 我们的主要go程序文件，一个简单的webserver应用
```

项目仓库地址：[github.com/cfanbo/democice][1]

**Dockerfile文件内容：**

```
# 以下内容生成docker镜像
# 基础镜像
FROM golang:latest
# 容器环境变量设置，会覆盖默认的变量值
ENV GOPROXY=https://goproxy.cn,direct
ENV GO111MODULE="on"
# 作者
MAINTAINER cfanbo haohtml@gmail.com
# 工作区
WORKDIR /go/src/app
# 复制仓库源文件到容器里
COPY . .
# 编译可执行二进制文件
RUN go build -o webserver
# 容器向外提供服务的暴露端口
EXPOSE 8080
# 启动服务
ENTRYPOINT ["./webserver"]
```

上面用到了COPY指令，另外还有一个ADD的指令，大部分场景下两者可以互用的，但仍存在有一定的区别的，具体请参考网上相关文章，另外还有一个CMD和ENTRYPOINT的区别也最好了解下。

# 创建镜像文件 

文件名为webserver

**\# docker build -t webserver .**

```
Sending build context to Docker daemon  7.351MB
 Step 1/9 : FROM golang:latest
  ---> ed081345a3da
 Step 2/9 : ENV GOPROXY=https://goproxy.cn,direct
  ---> Running in de85e43c2236
 Removing intermediate container de85e43c2236
  ---> 62e570a7a1ef
 Step 3/9 : ENV GO111MODULE="on"
  ---> Running in b1df8e38b573
 Removing intermediate container b1df8e38b573
  ---> c49293adea60
 Step 4/9 : MAINTAINER cfanbo haohtml@gmail.com
  ---> Running in 5a88eb2d35d1
 Removing intermediate container 5a88eb2d35d1
  ---> fa24c592ac35
 Step 5/9 : WORKDIR /go/src/app
  ---> Running in feabcabfb1d9
 Removing intermediate container feabcabfb1d9
  ---> da35eae9decb
 Step 6/9 : COPY . .
  ---> c6ffa2d1af58
 Step 7/9 : RUN go build -o webserver
  ---> Running in 258fa7eaf50f
 go: downloading github.com/gin-gonic/gin v1.5.0
 go: extracting github.com/gin-gonic/gin v1.5.0
 go: downloading github.com/ugorji/go v1.1.7
 go: downloading github.com/golang/protobuf v1.3.2
 go: downloading gopkg.in/yaml.v2 v2.2.2
 go: downloading github.com/gin-contrib/sse v0.1.0
 go: downloading gopkg.in/go-playground/validator.v9 v9.29.1
 go: downloading github.com/mattn/go-isatty v0.0.9
 go: extracting github.com/ugorji/go v1.1.7
 go: downloading github.com/ugorji/go/codec v1.1.7
 go: extracting gopkg.in/yaml.v2 v2.2.2
 go: extracting github.com/mattn/go-isatty v0.0.9
 go: extracting github.com/gin-contrib/sse v0.1.0
 go: downloading golang.org/x/sys v0.0.0-20190813064441-fde4db37ae7a
 go: extracting gopkg.in/go-playground/validator.v9 v9.29.1
 go: downloading github.com/leodido/go-urn v1.1.0
 go: downloading github.com/go-playground/universal-translator v0.16.0
 go: extracting github.com/ugorji/go/codec v1.1.7
 go: extracting github.com/golang/protobuf v1.3.2
 go: extracting github.com/leodido/go-urn v1.1.0
 go: extracting github.com/go-playground/universal-translator v0.16.0
 go: downloading github.com/go-playground/locales v0.12.1
 go: extracting github.com/go-playground/locales v0.12.1
 go: extracting golang.org/x/sys v0.0.0-20190813064441-fde4db37ae7a
 go: finding github.com/gin-gonic/gin v1.5.0
 go: finding github.com/gin-contrib/sse v0.1.0
 go: finding github.com/golang/protobuf v1.3.2
 go: finding github.com/ugorji/go/codec v1.1.7
 go: finding gopkg.in/go-playground/validator.v9 v9.29.1
 go: finding github.com/go-playground/universal-translator v0.16.0
 go: finding github.com/go-playground/locales v0.12.1
 go: finding github.com/leodido/go-urn v1.1.0
 go: finding gopkg.in/yaml.v2 v2.2.2
 go: finding github.com/mattn/go-isatty v0.0.9
 go: finding golang.org/x/sys v0.0.0-20190813064441-fde4db37ae7a
 Removing intermediate container 258fa7eaf50f
  ---> 0eca9a67e6d2
 Step 8/9 : EXPOSE 8080
  ---> Running in 8c74ecfcaaf3
 Removing intermediate container 8c74ecfcaaf3
  ---> 6e96325fe3f0
 Step 9/9 : ENTRYPOINT ["./webserver"]
  ---> Running in f0f6c49b5786
 Removing intermediate container f0f6c49b5786
  ---> c6de2a9a834a
 Successfully built c6de2a9a834a
 Successfully tagged webserver:latest
```

可以看到整个过程共9步：
第一步：从远程仓库下载基础镜像，如果本地有些镜像，则优先使用本地的
第二/三步：设置容器环境变量。后面我们会确认这一点
第四步：设置镜像作者
第五步：设置容器里的应用工作区目录为 /go/src/app
第六步：复制Dockerfile所在目录的文件到容器的工作区目录WORKDIR
第七步：编译程序生成二进制文件，由于示例中用到了Gin框架，所以会下载依赖的三方包。输出内容的最后两行为生成的镜像ID为c6de2a9a834a, 标签为 webserver:latest
第八步：容器向外提供服务的端口。使用时需要指定其对应的宿主机器端口才可以访问
第九步：容器启动后的程序执行命令

使用以下命令验证镜像

## \# docker images 

```
REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
webserver                                       latest              c6de2a9a834a        33 seconds ago      890MB
```

# 部署镜像 docker 容器 

```
# docker run -d --name webServer -p 8080:8080 webserver
d488ca7c8d5c7b1fd5c1094b36f87c4268a5bb588f093916a4aa56ce49b9e02b
```

```
参数：
-d  容器以守护进程的方式运行
--name 容器的名字,如果不指定的话，则取上面输出字符串的前12位字符
-p 指定宿主机器与容器的端口对应关系, 格式为“宿主端口:容器端口”，如果不指定此项，将无法访问容器里的服务
最后面的webserver 就是指我们要基于哪个镜像来创建容器了
```

部署成功，我们确认一下

## \# docker ps -a 

```
CONTAINER ID        IMAGE           COMMAND                  CREATED             STATUS                        PORTS                            NAMES
d488ca7c8d5c        webserver       "./webserver"            37 seconds ago      Up 36 seconds                 0.0.0.0:8080->8080/tcp           webServer
```

# 访问web站点 

```
# curl http://localhost:8080/ping
{"message":"pong"}
```

可以看到应用已经部署成功!

## 进入容器确认以上操作 

```
$ docker exec -it webServer /bin/bash
```

```
root@d488ca7c8d5c:/go/src/app# go env
 GO111MODULE="on"
 GOARCH="amd64"
 GOBIN=""
 GOCACHE="/root/.cache/go-build"
 GOENV="/root/.config/go/env"
 GOEXE=""
 GOFLAGS=""
 GOHOSTARCH="amd64"
 GOHOSTOS="linux"
 GONOPROXY=""
 GONOSUMDB=""
 GOOS="linux"
 GOPATH="/go"
 GOPRIVATE=""
 GOPROXY="https://goproxy.cn,direct"
 GOROOT="/usr/local/go"
 GOSUMDB="sum.golang.org"
 GOTMPDIR=""
 GOTOOLDIR="/usr/local/go/pkg/tool/linux_amd64"
 GCCGO="gccgo"
 AR="ar"
 CC="gcc"
 CXX="g++"
 CGO_ENABLED="1"
 GOMOD="/go/src/app/go.mod"
 CGO_CFLAGS="-g -O2"
 CGO_CPPFLAGS=""
 CGO_CXXFLAGS="-g -O2"
 CGO_FFLAGS="-g -O2"
 CGO_LDFLAGS="-g -O2"
 PKG_CONFIG="pkg-config"
 GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build036971232=/tmp/go-build -gno-record-gcc-switches"
```

注意观察系统变量已经变成了我们设置的值，GOPROXY设置值直接影响了我们 build 的时间，因为我们国内糟糕的网络环境！

root@d488ca7c8d5c:/go/src/app# ls -al

```
total 14920
 drwxr-xr-x 1 root root     4096 Dec 31 08:12 .
 drwxrwxrwx 1 root root     4096 Dec 31 08:12 ..
 -rw-r--r-- 1 root root       14 Dec 31 08:09 .dockerignore
 -rw-r--r-- 1 root root      467 Dec 31 08:08 Dockerfile
 -rw-r--r-- 1 root root       84 Dec 31 07:51 go.mod
 -rw-r--r-- 1 root root     3533 Dec 31 08:12 go.sum
 -rw-r--r-- 1 root root      191 Dec 31 08:02 main.go
 -rwxr-xr-x 1 root root 15247007 Dec 31 08:12 webserver
```

可以看到我们在.dockerignore文件中忽略掉的.git目录和README.md文件都不在容器内。

至此，我们基本已经完成了使用Docker构建的过程。

**但这里存在两个比较明显的问题：**
1. 镜像里的存在源码文件，实际上我们只需要编译的二进制文件就可以了，如果源文件特别大的话，镜像会非常大
2. 生成的镜像文件太大了，主要原因是国为我们使用的镜像由于需要编译需要，里面里包含了太多工具，这里大小为890M。这里如果后期我们直接使用此镜像部署容器的话，会发现下载镜像时会非常的慢。所以我们需要对此镜像进行精简，这里就需要用到多阶段构建来实现了。

# 多阶段构建 

首先我们修改Dockerfile文件内容

```
# 以下内容生成docker镜像
# 构建阶段 可指定阶段别名 FROM amd64/golang:latest as build_stage
# 基础镜像
FROM golang:latest
# 容器环境变量添加，会覆盖默认的变量值
ENV GOPROXY=https://goproxy.cn,direct
ENV GO111MODULE="on"
# 作者
LABEL author="cfanbo"
LABEL email="haohtml@gmail.com"
# 工作区
WORKDIR /go/src/app
# 复制仓库源文件到容器里
COPY . .
# 编译可执行二进制文件(一定要写这些编译参数，指定了可执行程序的运行平台,参考：https://www.jianshu.com/p/4b345a9e768e)
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o webserver
# 构建生产镜像，使用最小的linux镜像，只有5M
# 同一个文件里允许多个FROM出现的，每个FROM被称为一个阶段，多个FROM就是多个阶段，最终以最后一个FROM有效，以前的FROM被抛弃
# 多个阶段的使用场景就是将编译环境和生产环境分开
# 参考：https://docs.docker.com/engine/reference/builder/#from
FROM alpine:latest
WORKDIR /root/
# 从编译阶段复制文件
# 这里使用了阶段索引值，第一个阶段从0开始，如果使用阶段别名则需要写成 COPY --from=build_stage /go/src/app/webserver /
COPY --from=0 /go/src/app/webserver .
# 容器向外提供服务的暴露端口
EXPOSE 8080
# 启动服务
ENTRYPOINT ["./webserver"]
```

上面文件主要分为两个构建阶段，第一阶段是编译可执行程序，第二阶段是部署可执行程序到另一个镜像中，这个镜像大小只有5M，这样以后直接通过镜像部署的话，速度要快的多。

```
注意点：
1. 编译参数 指定了一些编译参数，指定了程序运行的平台、操作系统和是否关闭cgo(交叉编译不支持cgo)
2. COPY 指令的用法和上面的有些不一样，这里指定了--from参数，用来指定了第一个构建阶段索引0， 后面的 /go/src/app/webserver 文件来源，参数： https://docs.docker.com/engine/reference/builder/#from
3. 弃用“MAINTAINER” 关键字，使用 LABEL 代替
```

## 重新构建镜像 

```
# docker build -t webserver .
```

```
Sending build context to Docker daemon  10.24kB
Step 1/13 : FROM golang:latest
 ---> ed081345a3da
Step 2/13 : ENV GOPROXY=https://goproxy.cn,direct
 ---> Running in 5ae74ce28fea
Removing intermediate container 5ae74ce28fea
 ---> bb443a8c71d6
Step 3/13 : ENV GO111MODULE="on"
 ---> Running in 409f63a23060
Removing intermediate container 409f63a23060
 ---> 6f0af4ec9003
Step 4/13 : LABEL author="cfanbo"
 ---> Running in 07172a18839d
Removing intermediate container 07172a18839d
 ---> d795099894b3
Step 5/13 : LABEL email="haohtml@gmail.com"
 ---> Running in 4d00232ed84e
Removing intermediate container 4d00232ed84e
 ---> caf30443fb08
Step 6/13 : WORKDIR /go/src/app
 ---> Running in 86ab6103b181
Removing intermediate container 86ab6103b181
 ---> d3432afa35d1
Step 7/13 : COPY . .
 ---> b8aab51c07b6
Step 8/13 : RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o webserver
 ---> Running in 23f7df074228
go: downloading github.com/gin-gonic/gin v1.5.0
go: extracting github.com/gin-gonic/gin v1.5.0
go: downloading github.com/mattn/go-isatty v0.0.9
go: downloading github.com/golang/protobuf v1.3.2
go: downloading github.com/gin-contrib/sse v0.1.0
go: downloading gopkg.in/yaml.v2 v2.2.2
go: downloading gopkg.in/go-playground/validator.v9 v9.29.1
go: extracting github.com/mattn/go-isatty v0.0.9
go: downloading github.com/ugorji/go v1.1.7
go: downloading golang.org/x/sys v0.0.0-20190813064441-fde4db37ae7a
go: extracting gopkg.in/go-playground/validator.v9 v9.29.1
go: extracting github.com/gin-contrib/sse v0.1.0
go: extracting github.com/ugorji/go v1.1.7
go: downloading github.com/ugorji/go/codec v1.1.7
go: extracting gopkg.in/yaml.v2 v2.2.2
go: downloading github.com/leodido/go-urn v1.1.0
go: downloading github.com/go-playground/universal-translator v0.16.0
go: extracting github.com/golang/protobuf v1.3.2
go: extracting github.com/leodido/go-urn v1.1.0
go: extracting github.com/go-playground/universal-translator v0.16.0
go: extracting github.com/ugorji/go/codec v1.1.7
go: downloading github.com/go-playground/locales v0.12.1
go: extracting golang.org/x/sys v0.0.0-20190813064441-fde4db37ae7a
go: extracting github.com/go-playground/locales v0.12.1
go: finding github.com/gin-gonic/gin v1.5.0
go: finding github.com/gin-contrib/sse v0.1.0
go: finding github.com/golang/protobuf v1.3.2
go: finding github.com/ugorji/go/codec v1.1.7
go: finding gopkg.in/go-playground/validator.v9 v9.29.1
go: finding github.com/go-playground/universal-translator v0.16.0
go: finding github.com/go-playground/locales v0.12.1
go: finding github.com/leodido/go-urn v1.1.0
go: finding gopkg.in/yaml.v2 v2.2.2
go: finding github.com/mattn/go-isatty v0.0.9
go: finding golang.org/x/sys v0.0.0-20190813064441-fde4db37ae7a
Removing intermediate container 23f7df074228
 ---> df21e2afb3b1
Step 9/13 : FROM alpine:latest
 ---> 961769676411
Step 10/13 : WORKDIR /root/
 ---> Running in 1c1e2532114c
Removing intermediate container 1c1e2532114c
 ---> bac929e6e841
Step 11/13 : COPY --from=0 /go/src/app/webserver .
 ---> aa0a1ad63443
Step 12/13 : EXPOSE 8080
 ---> Running in cd3d1de8f526
Removing intermediate container cd3d1de8f526
 ---> 00ee9d353265
Step 13/13 : ENTRYPOINT ["./webserver"]
 ---> Running in 4d12bc7cece1
Removing intermediate container 4d12bc7cece1
 ---> 56279170f1b9
Successfully built 56279170f1b9
Successfully tagged webserver:latest
```

这里多了一些步骤，此时我们会发现一次会生成两个镜像文件 df21e2afb3b1 和 56279170f1b9 （webserver:latest）

```
# docker images
```

```
REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
webserver                                       latest              56279170f1b9        3 minutes ago       20.8MB
<none>                                          <none>              df21e2afb3b1        3 minutes ago       891MB
```

其中 df21e2afb3b1 镜像是我们用来编译可执行文件的，名字为，另一个webserver:latest 就是我们想要的镜像了，可以看到只有20.8M。我们可以看一下这个镜像的构建历史

```
docker history --no-trunc webserver
```

```
IMAGE                                                                     CREATED             CREATED BY                                                                                           SIZE                COMMENT
sha256:56279170f1b96a25f93fe8ffc125350d5da1dd4d690f70a36433c6b88e403d8e   8 minutes ago       /bin/sh -c #(nop)  ENTRYPOINT ["./webserver"]                                                        0B
sha256:00ee9d3532653dfbbddbe061407a9bb85e02d930c05466964407d3ace85a5e59   8 minutes ago       /bin/sh -c #(nop)  EXPOSE 8080                                                                       0B
sha256:aa0a1ad634433908e4e5b45e59a3e1fc41bda9d6a8c06b04b2180cc0c5a7bcb5   8 minutes ago       /bin/sh -c #(nop) COPY file:d6dc61b2e76bdb1ac02581e93ff49be50b27accc112311e2b5498043e4d88f4f in .    15.2MB
sha256:bac929e6e84140086399163fe5b1a785474aeecc424b72f50d76ec9b5812101f   8 minutes ago       /bin/sh -c #(nop) WORKDIR /root/                                                                     0B
sha256:961769676411f082461f9ef46626dd7a2d1e2b2a38e6a44364bcbecf51e66dd4   4 months ago        /bin/sh -c #(nop)  CMD ["/bin/sh"]                                                                   0B
<missing>                                                                 4 months ago        /bin/sh -c #(nop) ADD file:fe64057fbb83dccb960efabbf1cd8777920ef279a7fa8dbca0a8801c651bdf7c in /     5.58MB
```

从镜像层创建时间可以看到第一层执行的命令，最上层为最新的，同时也可以看到可执行文件为15.2MB大小，加上基础镜像，整好是20M多。

# 部署容器 

```
# docker run -d -–name webServer -p 8080:8080 webserver
```

# 测试容器服务 

```
# curl http://localhost:8080/ping
```

可以看到服务启动成功。

这里我们不再演示将webserver:latest镜像放在远程仓库中，然后在服务器拉取并部署了。

**总结：**
1. 对于不需要部署在docker容器里的内容可以使用 .dockerignore文件来实现
2. 可以在 Dockerfile 中来操作容器里的环境变量
3. 可以使用多阶段构建来实现我们的应用，一个Dockerfile中允许出现多个 FROM关键字
4. 注意本例中编译相关参数的作用
5. “MAINTAINER” 关键字被弃用了，使用LABEL代替（）
6. 对于镜像文件，一定要精简精简再精简，参考小米技术的文章 [https://mp.weixin.qq.com/s/S1Ib08SpQbf1SCbCutUoqQ](https://mp.weixin.qq.com/s/S1Ib08SpQbf1SCbCutUoqQ)
7. 编译阶段我们直接使用了golang的镜像，如果想方便的话，也可以使用 stretch 空镜像，手动下载golang安装包并设置相应的环境变量来实现

如果你对微服务有所了解的话，会发现这种方法和微服务中的 sidecar 基本一样的。

# **参考** 

* [https://blog.csdn.net/weixin_42852772/article/details/82013418](https://blog.csdn.net/weixin_42852772/article/details/82013418)
* [https://mp.weixin.qq.com/s/S1Ib08SpQbf1SCbCutUoqQ](https://mp.weixin.qq.com/s/S1Ib08SpQbf1SCbCutUoqQ)
* [Golang 交叉编译](https://book.eddycjy.com/golang/gin/cgo.html)
* [go如何进行交叉编译](https://www.jianshu.com/p/4b345a9e768e)
* [多阶段用法Dockerfile示例](https://github.com/golang/playground/blob/master/Dockerfile)
* [Dockerfile文件用法（官方）](https://docs.docker.com/engine/reference/builder/)

 [1]: http://github.com/cfanbo/democice