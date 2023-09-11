---
title: Golang开发中中使用GitHub私有仓库
author: admin
type: post
date: 2020-09-19T09:01:18+00:00
url: /archives/20179
categories:
 - 程序开发
tags:
 - golang

---
私有仓库地址为

```
github.com/cfanbo/websocket
```

## 一、设置私有环境变量 GOPRIVATE 

```
$ go env -w GOPRIVATE=github.com/cfanbo/websocket
```

对于为什么需要设置 GOPRIMARY 变量，可以参考 [这里](https://gocn.vip/topics/9904)

对于GOPRIVATE值级别分为仓库级别和账号级别。

如果只有一个仓库，直接设置为仓库地址即可。如果有多个私有仓库的话，使用”,”分开，都在这个账号下，也可以将值设置为账号级别，这样账号下的所有私有仓库都可以正常访问。如 http://github.com/cfanbo

如果不想每次都重新设置，我们也可以利用通配符，例如：

```
$ go env -w GOPRIVATE="*.example.com"
```

这样子设置的话，所有模块路径为 example.com 的子域名（例如：git.example.com）都将不经过 Go module proxy 和 Go checksum database，需要注意的是不包括 example.com 本身。

国内用户访问仓库建议设置 GORPOXY为 https://proxy.golang.org,direct

## 二、设置凭证 

使用私有仓库一定要绕不开权限设置这一步。访问仓库来常见的有两种方式，分别为SSH和 Https 。对于私有仓库来说，ssh可以设置rsa私钥来访问，https这种则可以使用用户名和密码，一般通过命令行访问的时候，会自动提示用户输入这些信息。

对于权限控制这一块可参[考官方文档][1] 。其实在官方文档里还提供了第三种访问仓库的方式，那就是 Personal access token，简称 PAT, 这种 Token 是专门为api调用提供的，常见于自动化工作流中，如 CICD场景。

这里我们就利用PAT 来实现

 1. 在Github.com 网站生成 [Personal access tokens][2]，新手可参考[官方教程文档][3]
 2. 本地配置token凭证


```
$ git config --global url."https://${username}:${access_token}@github.com".insteadOf / "https://github.com"
```

如果你在使用Github Actions部署时，遇到无法读取版本号问题，需要改写成 **git config –global url.”https://${username}:${access_token}@github.com”.insteadOf “https://github.com**“

命令验证

```
go get github.com/cfanbo/websocket
```

到这里基本配置基本完成了。

## 其它场景 

如果要用在docker环境中的话，也要记得设置上面的几个环境变量值。

以下为一个docker示例

```
# Start from the latest golang base image
FROM golang:alpine

RUN GOCACHE=OFF

# 设置环境变量
RUN go env -w GOPRIVATE=github.com/ereshzealous

# Set the Current Working Directory inside the container
WORKDIR /app

# Copy everything from the current directory to the Working Directory inside the container
COPY . .

RUN apk add git

# 设置访问仓库凭证
RUN git config --global url."https://user-name:<access-token>@github.com".insteadOf "https://github.com"

# Build the Go app
RUN go build -o main .

# Expose port 8080 to the outside world
EXPOSE 8080

#ENTRYPOINT ["/app"]

# Command to run the executable
CMD ["./main"]
```

 [1]: https://docs.github.com/en/github/authenticating-to-github/about-authentication-to-github#authenticating-with-the-command-line
 [2]: https://github.com/settings/tokens
 [3]: https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token