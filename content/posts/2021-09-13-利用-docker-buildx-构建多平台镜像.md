---
title: 利用 docker buildx 构建多平台镜像
author: admin
type: post
date: 2021-09-13T02:01:39+00:00
url: /archives/31052
categories:
 - 系统架构
tags:
 - docker

---
# 什么是 docker buildx 

Docker Buildx是一个CLI插件，它扩展了Docker命令，完全支持Moby BuildKit builder toolkit提供的功能。它提供了与docker build相同的用户体验，并提供了许多新功能，如创建作用域生成器实例和针对多个节点并发构建。

Docker Buildx包含在Docker 19.03中，并与以下Docker Desktop版本捆绑在一起。请注意，必须启用“实验特性”选项才能使用Docker Buildx。

Docker Desktop Enterprise version 2.1.0
Docker Desktop Edge version 2.0.4.0 or higher

# 用法 

```
Usage:  docker buildx [OPTIONS] COMMAND

Extended build capabilities with BuildKit

Options:
      --builder string   Override the configured builder instance

Management Commands:
  imagetools  Commands to work on images in registry

Commands:
  bake        Build from a file
  build       Start a build
  create      Create a new builder instance
  du          Disk usage
  inspect     Inspect current builder instance
  ls          List builder instances
  prune       Remove build cache
  rm          Remove a builder instance
  stop        Stop builder instance
  use         Set the current builder instance
  version     Show buildx version information

Run 'docker buildx COMMAND --help' for more information on a command
```

一般用到 create、ls 和 rm 就可以了

# 创建 builder 实例

由于 Docker 默认的 builder 实例不支持同时指定多个 –platform，所有我们必须先创建一个 builder 实例

```
$ docker buildx create --use --name=mybuilder-cn --driver docker-container

```

`--use` 表示使用当前创建的 builder 实例
`--name` 实例名称
`--driver` 实例驱动(docker、 docker-container 和 kubernetes)

更多用法通过命令 `docker buildx create -h` 查看

查看新实例

```
NAME/NODE       DRIVER/ENDPOINT             STATUS   PLATFORMS
mybuilder-cn *  docker-container
  mybuilder-cn0 unix:///var/run/docker.sock inactive
default         docker
  default       default                     running  linux/amd64, linux/386

```

其中 * 表示当前正在使用的builder实例。

我们查看一下实例详细信息

```
$ docker buildx inspect
Name:   mybuilder-cn
Driver: docker-container

Nodes:
Name:      mybuilder-cn0
Endpoint:  unix:///var/run/docker.sock
Status:    running
Platforms: linux/amd64, linux/386

```

# 构建镜像

创建Dockerfile 文件

```
FROM --platform=$TARGETPLATFORM alpine
RUN uname -a > /os.txt
CMD cat /os.txt

```

这里使用到了变量 `TARGETPLATFORM`, 内置的一些变量在本文下方了解

## 构建指定平台

使用 docker buildx build 命令构建镜像

```
$ docker buildx build .
WARN[0000] No output specified for docker-container driver. Build result will only remain in the build cache. To push result image into registry use --push or to load image into docker use --load
[+] Building 32.9s (6/6) FINISHED
 => [internal] booting buildkit                                                                                                                                                                      25.4s
 => => pulling image moby/buildkit:buildx-stable-1                                                                                                                                                   23.9s
 => => creating container buildx_buildkit_mybuilder-cn0                                                                                                                                               1.5s
 => [internal] load build definition from Dockerfile                                                                                                                                                  0.0s
 => => transferring dockerfile: 115B                                                                                                                                                                  0.0s
 => [internal] load .dockerignore                                                                                                                                                                     0.0s
 => => transferring context: 2B                                                                                                                                                                       0.0s
 => [internal] load metadata for docker.io/library/alpine:latest                                                                                                                                      5.1s
 => [1/2] FROM docker.io/library/alpine@sha256:e1c082e3d3c45cccac829840a25941e679c25d438cc8412c2fa221cf1a824e6a                                                                                       1.9s
 => => resolve docker.io/library/alpine@sha256:e1c082e3d3c45cccac829840a25941e679c25d438cc8412c2fa221cf1a824e6a                                                                                       0.0s
 => => sha256:a0d0a0d46f8b52473982a3c466318f479767577551a53ffc9074c9fa7035982e 2.81MB / 2.81MB                                                                                                        1.8s
 => => extracting sha256:a0d0a0d46f8b52473982a3c466318f479767577551a53ffc9074c9fa7035982e                                                                                                             0.1s
 => [2/2] RUN uname -a > /os.txt

```

从上面的输出可以看出，构建步骤大概如下：

 1. 加载引导buildkit
 2. 下载镜像文件 image moby/buildkit:buildx-stable-1
 3. 使用镜像创建容器 buildx\_buildkit\_mybuilder-cn0
 4. 将当前 Dockerfile 文件传输到容器内
 5. 在容器内下载镜像 [docker.io/library/alpine:latest][1]
 6. 运行Dockerfile中的 RUN 指令

如果你用 docker ps 命令查看容器的话，会看到创建了一个 buildkitd 容器

## 构建多平台镜像

也可以指定一些设置，如构建的目标平台等

```
$ docker buildx build --platform linux/arm,linux/arm64,linux/amd64 -t cfanbo/hello . --push
[+] Building 18.4s (18/18) FINISHED
 => [internal] load build definition from Dockerfile                                                                                                                                                  0.0s
 => => transferring dockerfile: 115B                                                                                                                                                                  0.0s
 => [internal] load .dockerignore                                                                                                                                                                     0.0s
 => => transferring context: 2B                                                                                                                                                                       0.0s
 => [linux/amd64 internal] load metadata for docker.io/library/alpine:latest                                                                                                                          2.3s
 => [linux/arm64 internal] load metadata for docker.io/library/alpine:latest                                                                                                                          2.4s
 => [linux/arm/v7 internal] load metadata for docker.io/library/alpine:latest                                                                                                                         2.7s
 => [auth] library/alpine:pull token for registry-1.docker.io                                                                                                                                         0.0s
 => [auth] library/alpine:pull token for registry-1.docker.io                                                                                                                                         0.0s
 => [auth] library/alpine:pull token for registry-1.docker.io                                                                                                                                         0.0s
 => [linux/arm/v7 1/2] FROM docker.io/library/alpine@sha256:e1c082e3d3c45cccac829840a25941e679c25d438cc8412c2fa221cf1a824e6a                                                                          0.0s
 => => resolve docker.io/library/alpine@sha256:e1c082e3d3c45cccac829840a25941e679c25d438cc8412c2fa221cf1a824e6a                                                                                       0.0s
 => [linux/arm64 1/2] FROM docker.io/library/alpine@sha256:e1c082e3d3c45cccac829840a25941e679c25d438cc8412c2fa221cf1a824e6a                                                                           0.0s
 => => resolve docker.io/library/alpine@sha256:e1c082e3d3c45cccac829840a25941e679c25d438cc8412c2fa221cf1a824e6a                                                                                       0.0s
 => [linux/amd64 1/2] FROM docker.io/library/alpine@sha256:e1c082e3d3c45cccac829840a25941e679c25d438cc8412c2fa221cf1a824e6a                                                                           0.0s
 => => resolve docker.io/library/alpine@sha256:e1c082e3d3c45cccac829840a25941e679c25d438cc8412c2fa221cf1a824e6a                                                                                       0.0s
 => CACHED [linux/arm/v7 2/2] RUN uname -a > /os.txt                                                                                                                                                  0.0s
 => CACHED [linux/arm64 2/2] RUN uname -a > /os.txt                                                                                                                                                   0.0s
 => CACHED [linux/amd64 2/2] RUN uname -a > /os.txt                                                                                                                                                   0.0s
 => exporting to image                                                                                                                                                                               15.5s
 => => exporting layers                                                                                                                                                                               0.0s
 => => exporting manifest sha256:fc3842a7caa48e4124d6a76dcb1cff3944e55eeefd5daeb21dfe8201de8d856c                                                                                                     0.0s
 => => exporting config sha256:4db341c87f5e347b12f2f8955da60ac4db9bcf2bc17aed18079d2ac55b759cca                                                                                                       0.0s
 => => exporting manifest sha256:91bcf398194707f1b9ce9fe6b7dcc29b809dc3278cddc1a13cb5dc76d60bb281                                                                                                     0.0s
 => => exporting config sha256:e0f919da8544b23e919c1257c6b40c54a4c9114447e1d637f4d965204757dbab                                                                                                       0.0s
 => => exporting manifest sha256:4a5479758b57441933984c2209bccff4f0c47c7dd74be95b4b22162d105e639a                                                                                                     0.0s
 => => exporting config sha256:6af666ad21adae02a9147dfcec4c9397e2be9e12d5861614e5fc20a287fe4086                                                                                                       0.0s
 => => exporting manifest list sha256:e8758d75c4d6cf2a633c645616c1d4f76006ee0503d7b16532a744346bca0cf7                                                                                                0.0s
 => => pushing layers                                                                                                                                                                                12.4s
 => => pushing manifest for docker.io/cfanbo/hello:latest@sha256:e8758d75c4d6cf2a633c645616c1d4f76006ee0503d7b16532a744346bca0cf7                                                                     3.1s
 => [auth] cfanbo/hello:pull,push token for registry-1.docker.io                                                                                                                                      0.0s
 => [auth] cfanbo/hello:pull,push token for registry-1.docker.io                                                                                                                                      0.0s
 => [auth] cfanbo/hello:pull,push token for registry-1.docker.io

```

这里我们指定了三个平台，它们将同时分别进行构建。由于我这里多次执行了构建命令，所以三个平台构建过程中都显示 CACHED，表示用到了缓存。

我这里  平台的用户名为 cfanbo, 替换为自己的 Docker Hub 用户名。

`--push` 参数表示将构建好的镜像推送到 Docker 仓库,在构建前记得先执行 docker login 命令登录。这里通过  可以看到分别为三个平台构建了不同的镜像。

1. 查看镜像信息


```
$ docker buildx imagetools inspect cfanbo/hello
Name:      docker.io/cfanbo/hello:latest
MediaType: application/vnd.docker.distribution.manifest.list.v2+json
Digest:    sha256:e8758d75c4d6cf2a633c645616c1d4f76006ee0503d7b16532a744346bca0cf7

Manifests:
  Name:      docker.io/cfanbo/hello:latest@sha256:fc3842a7caa48e4124d6a76dcb1cff3944e55eeefd5daeb21dfe8201de8d856c
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/arm/v7

  Name:      docker.io/cfanbo/hello:latest@sha256:91bcf398194707f1b9ce9fe6b7dcc29b809dc3278cddc1a13cb5dc76d60bb281
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/arm64

  Name:      docker.io/cfanbo/hello:latest@sha256:4a5479758b57441933984c2209bccff4f0c47c7dd74be95b4b22162d105e639a
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/amd64

```

1. 验证镜像


```
### Linux
$ docker run --rm cfanbo/hello:latest
Linux buildkitsandbox 5.11.0-31-generic #33-Ubuntu SMP Wed Aug 11 13:19:04 UTC 2021 x86_64 Linux

### aarch64
$ sudo docker run --rm cfanbo/hello:latest
Linux buildkitsandbox 5.11.0-31-generic #33-Ubuntu SMP Wed Aug 11 13:19:04 UTC 2021 aarch64 Linux

```

5. 优化

首先我们了解一些 Dockerfile内置的变量

Dockerfile 支持如下架构相关的变量

`TARGETPLATFORM` 构建镜像的目标平台，例如 linux/amd64, linux/arm/v7, windows/amd64。

`TARGETOS` 构建镜像目标平台的 OS 类型，例如 linux, windows

`TARGETARCH` 构建镜像目标平台的架构类型，例如 amd64, arm

`TARGETVARIANT` 构建镜像目标平台的变种，该变量可能为空，例如 v7

`BUILDPLATFORM` 构建镜像所使用的主机平台，例如 linux/amd64

`BUILDOS` 构建镜像所使用主机平台 的 OS 类型，例如 linux

`BUILDARCH` 构建镜像所使用主机平台 的架构类型，例如 amd64

`BUILDVARIANT`构建镜像所使用主机平台 的变种，该变量可能为空，例如 v7

**使用举例**
例如我们要构建支持 linux/arm/v7 和 linux/amd64 两种架构的镜像。假设已经生成了两个平台对应的二进制文件：

```
bin/dist-linux-arm
bin/dist-linux-amd64

```

那么 Dockerfile 可以这样书写：

```
FROM scratch

# 使用变量必须申明
ARG TARGETOS
ARG TARGETARCH

COPY bin/dist-${TARGETOS}-${TARGETARCH} /dist

ENTRYPOINT ["dist"]

```

上面的方法虽然可以实现，但还有一个比较麻烦的问题。我们要构建的不同平台的二进制程序，每次都需要手动指定目标一个平台再构建，法子太笨了，有没有更简单高效的的办法呢？

这里我们可以利用另一个变量 `BUILDPLATFORM`，再利用两阶段构建来解决这个问题，此时的 `Dockerfile` 内容为

```
FROM --platform=$BUILDPLATFORM golang:1.20-alpine AS builder
ARG TARGETOS
ARG TARGETARCH
WORKDIR /app
ADD . .
RUN GOOS=$TARGETOS GOARCH=$TARGETARCH go build -o hello .

FROM --platform=$TARGETPLATFORM alpine:latest
WORKDIR /app
COPY --from=builder /app/hello .
CMD ["./hello"]
```

我们写一个简单的golang程序

```
package main

import (
    "fmt"
    "runtime"
)

func main() {
    fmt.Printf("Hello, %s/%s!n", runtime.GOOS, runtime.GOARCH)
}
```

此时重新构建镜像

```
$ docker buildx build --platform linux/arm,linux/arm64,linux/amd64 -t cfanbo/hello:v2 . --push
[+] Building 833.3s (28/29)                                                                                                                            docker-container:mybuilder-cn
 => [internal] booting buildkit                                                                                                                                               223.0s
 => => pulling image moby/buildkit:buildx-stable-1                                                                                                                            166.2s
 => => creating container buildx_buildkit_mybuilder-cn0                                                                                                                        42.9s
 => [internal] load build definition from Dockerfile                                                                                                                            3.0s
 => => transferring dockerfile: 316B                                                                                                                                            0.2s
 => [internal] load .dockerignore                                                                                                                                               2.0s
 => => transferring context: 2B                                                                                                                                                 0.2s
 => [linux/amd64 internal] load metadata for docker.io/library/alpine:latest                                                                                                   15.9s
 => [linux/amd64 internal] load metadata for docker.io/library/golang:1.20-alpine                                                                                             600.3s
 => [auth] library/alpine:pull token for registry-1.docker.io                                                                                                                   2.0s
 => [linux/arm64 internal] load metadata for docker.io/library/alpine:latest                                                                                                   14.9s
 => [linux/arm/v7 internal] load metadata for docker.io/library/alpine:latest                                                                                                  14.5s
 => [auth] library/golang:pull token for registry-1.docker.io                                                                                                                   0.0s
 => [auth] library/alpine:pull token for registry-1.docker.io                                                                                                                   0.6s
 => [linux/amd64 builder 1/4] FROM docker.io/library/golang:1.20-alpine@sha256:7efb78dac256c450d194e556e96f80936528335033a26d703ec8146cec8c2090                               380.6s
 => => resolve docker.io/library/golang:1.20-alpine@sha256:7efb78dac256c450d194e556e96f80936528335033a26d703ec8146cec8c2090                                                     0.2s
 => => sha256:8fa91ef8ef8a80329ba55630fd46f90a465c3f58322192320a32044c49f7daad 156B / 156B                                                                                      1.1s
 => => sha256:f8f5e977ad9826fc24b1db3f0dbb1ea9a261cd25d4657a9d50e7bf435638cd11 101.36MB / 101.36MB                                                                            341.7s
 => => sha256:7f9bcf943fa5571df036dca6da19434d38edf546ef8bb04ddbc803634cc9a3b8 284.71kB / 284.71kB                                                                              5.2s
 => => extracting sha256:7f9bcf943fa5571df036dca6da19434d38edf546ef8bb04ddbc803634cc9a3b8                                                                                       2.0s
 => => extracting sha256:f8f5e977ad9826fc24b1db3f0dbb1ea9a261cd25d4657a9d50e7bf435638cd11                                                                                      36.8s
 => => extracting sha256:8fa91ef8ef8a80329ba55630fd46f90a465c3f58322192320a32044c49f7daad                                                                                       0.1s
 => [linux/amd64 stage-1 1/3] FROM docker.io/library/alpine:latest@sha256:82d1e9d7ed48a7523bdebc18cf6290bdb97b82302a8a9c27d4fe885949ea94d1                                     38.2s
 => => resolve docker.io/library/alpine:latest@sha256:82d1e9d7ed48a7523bdebc18cf6290bdb97b82302a8a9c27d4fe885949ea94d1                                                          0.2s
 => => sha256:31e352740f534f9ad170f75378a84fe453d6156e40700b882d737a8f4a6988a3 3.40MB / 3.40MB                                                                                 34.8s
 => => extracting sha256:31e352740f534f9ad170f75378a84fe453d6156e40700b882d737a8f4a6988a3                                                                                       3.0s
 => [linux/arm/v7 stage-1 1/3] FROM docker.io/library/alpine:latest@sha256:82d1e9d7ed48a7523bdebc18cf6290bdb97b82302a8a9c27d4fe885949ea94d1                                    32.2s
 => => resolve docker.io/library/alpine:latest@sha256:82d1e9d7ed48a7523bdebc18cf6290bdb97b82302a8a9c27d4fe885949ea94d1                                                          0.2s
 => => sha256:633ba29fd335042456b6e2c073636f6fa30de56f1331c442914739b92a479974 2.90MB / 2.90MB                                                                                 29.7s
 => => extracting sha256:633ba29fd335042456b6e2c073636f6fa30de56f1331c442914739b92a479974                                                                                       1.8s
 => [linux/arm64 stage-1 1/3] FROM docker.io/library/alpine:latest@sha256:82d1e9d7ed48a7523bdebc18cf6290bdb97b82302a8a9c27d4fe885949ea94d1                                     38.5s
 => => resolve docker.io/library/alpine:latest@sha256:82d1e9d7ed48a7523bdebc18cf6290bdb97b82302a8a9c27d4fe885949ea94d1                                                          0.2s
 => => sha256:8c6d1654570f041603f4cef49c320c8f6f3e401324913009d92a19132cbf1ac0 3.33MB / 3.33MB                                                                                 34.5s
 => => extracting sha256:8c6d1654570f041603f4cef49c320c8f6f3e401324913009d92a19132cbf1ac0                                                                                       3.2s
 => [internal] load build context                                                                                                                                               0.6s
 => => transferring context: 544B                                                                                                                                               0.0s
 => [linux/arm/v7 stage-1 2/3] WORKDIR /app                                                                                                                                     0.9s
 => [linux/amd64 stage-1 2/3] WORKDIR /app                                                                                                                                      1.4s
 => [linux/arm64 stage-1 2/3] WORKDIR /app                                                                                                                                      1.5s
 => [linux/amd64 builder 2/4] WORKDIR /app                                                                                                                                      2.9s
 => [linux/amd64 builder 3/4] ADD . .                                                                                                                                           0.3s
 => [linux/amd64 builder 4/4] RUN GOOS=linux GOARCH=arm64 go build -o hello .                                                                                                 157.8s
 => [linux/amd64 builder 4/4] RUN GOOS=linux GOARCH=amd64 go build -o hello .                                                                                                 158.4s
 => [linux/amd64 builder 4/4] RUN GOOS=linux GOARCH=arm go build -o hello .                                                                                                   156.1s
 => [linux/arm/v7 stage-1 3/3] COPY --from=builder /app/hello .                                                                                                                 0.3s
 => [linux/arm64 stage-1 3/3] COPY --from=builder /app/hello .                                                                                                                  0.2s
 => [linux/amd64 stage-1 3/3] COPY --from=builder /app/hello .                                                                                                                  0.3s
 => exporting to image                                                                                                                                                         37.3s
 => => exporting layers                                                                                                                                                         1.4s
 => => exporting manifest sha256:148ce9bd7ff585e4fd51208cc609012b010a6688ceaf432ea1c9931ed25966b8                                                                               0.0s
 => => exporting config sha256:9212722eba519486331427497448bfa025bcf8430f4fc63e59eae029e6e9572c                                                                                 0.0s
 => => exporting manifest sha256:fe88bf0f3ea814d797d1981c1d9807674dab23a767130da5100ac12374581a14                                                                               0.0s
 => => exporting config sha256:634f645cb55a93ddf999ce79b04878c9859e238f01f556876278f8abf1b76395                                                                                 0.0s
 => => exporting manifest sha256:d4f1a318546840a94de6e1a3e4f609097eb63ba7338ebdf1813bf364a8b4d4ad                                                                               0.0s
 => => exporting config sha256:6053d6935151693c906d4adfb2ab0545b28f88a35e18940520bd20a6c7dabf52                                                                                 0.0s
 => => exporting manifest list sha256:c7281230c39c54ae2556fc8ec711bccd95cee6d89ce218c65fb36346c5022a53                                                                          0.0s
 => => pushing layers                                                                                                                                                          31.2s
 => => pushing manifest for docker.io/cfanbo/hello:v2@sha256:c7281230c39c54ae2556fc8ec711bccd95cee6d89ce218c65fb36346c5022a53                                                   4.5s
 => [auth] cfanbo/hello:pull,push token for registry-1.docker.io                                                                                                                0.0s
 => [auth] cfanbo/hello:pull,push token for registry-1.docker.io                                                                                                                0.0s
```

从日志中可以看到，首先在基础镜像 `golang:1.20-alpine` 里通过设置不同的环境变量实现了 golang 交叉编译，然后在对每个不同的架构平台进行镜像构建，本次生成的镜像为 `hello:v2`。![](https://blogstatic.haohtml.com/uploads/2023/08/d2b5ca33bd970f64a6301fa75ae2eb22.png)

我们用 manifest 命令验证一下

```
$ docker manifest inspect cfanbo/hello:v2
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
   "manifests": [
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 945,
         "digest": "sha256:148ce9bd7ff585e4fd51208cc609012b010a6688ceaf432ea1c9931ed25966b8",
         "platform": {
            "architecture": "arm",
            "os": "linux",
            "variant": "v7"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 945,
         "digest": "sha256:fe88bf0f3ea814d797d1981c1d9807674dab23a767130da5100ac12374581a14",
         "platform": {
            "architecture": "arm64",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 945,
         "digest": "sha256:d4f1a318546840a94de6e1a3e4f609097eb63ba7338ebdf1813bf364a8b4d4ad",
         "platform": {
            "architecture": "amd64",
            "os": "linux"
         }
      }
   ]
}
```

可以看到镜像包含了不同构架平台的层信息。

测试一下(amd64)

```
$ docker run --rm cfanbo/hello:v2
Hello, linux/amd64!
```

# 清理工作 

删除cache 和 builder instance

```
$ docker buildx prune
$ docker buildx rm mybuilder-cn
```

# 参考资料 
* https://docs.docker.com/buildx/working-with-buildx/
* https://docs.docker.com/engine/reference/commandline/buildx/

 [1]: http://docker.io/library/alpine:latest