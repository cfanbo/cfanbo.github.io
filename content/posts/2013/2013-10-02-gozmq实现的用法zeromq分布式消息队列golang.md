---
title: gozmq的安装与使用教程(zeromq分布式消息队列+golang)
author: admin
type: post
date: 2013-10-02T07:28:35+00:00
url: /archives/14496
categories:
 - 程序开发
tags:
 - golang
 - zeromq

---
实现功能：用go实现消息队列的写入与读取(打算用在发送邮件服务)

环境工具：
Centos 64X 6.4
zeromq 3.2.4： [zeromq.org](http://www.zeromq.org)
golang： [http://golang.org/](http://golang.org/)

**一．安装golang( [http://golang.org/doc/install](http://golang.org/doc/install))**
这一步很简单，只需要从 [http://code.google.com/p/go/downloads](http://code.google.com/p/go/downloads) 下载到服务器，解压到/usr/local/go目录，再设置一下系统变量就可以了．

```
wget https://go.googlecode.com/files/go1.1.2.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.1.2.linux-amd64.tar.gz
```

**设置系统变量GOROOT**

Add `/usr/local/go/bin` to the `PATH` environment variable. You can do this by adding this line to your `/etc/profile` (for a system-wide installation) or `$HOME/.profile`:

```
export PATH=$PATH:/usr/local/go/bin
```

执行命令 #source /etc/profile 使环境变量生效．

**设置项目环境变量GOPATH**

下面我们需要设置开发项目使用的环境变量GOPATH的路径．可直接在命令行下执行下面的命令，也可以将下面的命令写入/etc/profile(或者 $HOME/.profile)中，这样下次使用的时候变量不会丢失．参考：

$ mkdir $HOME/go
For convenience, add the workspace’s bin subdirectory to your PATH:

```
$ export GOPATH=$HOME/go
$ export PATH=$PATH:$GOPATH/bin
```

也可以将上面两条命令写到/etc/profile里，然后再执行 `#source /etc/profile` 命令，使环境变量生效．

到此为止，go的配置基本上已经完成．可以用 go env 命令查看相关的配置信息.

[![linux_go_env](https://blogstatic.haohtml.com//uploads/2023/09/linux_go_env.png)][1]

**二．安装zeromq**

参考： [http://blog.haohtml.com/archives/13798](http://blog.haohtml.com/archives/13798),这里使用的是 [zeromq 2.3.4](http://zeromq.org/intro:get-the-software) 版本

**三．安装gozmq**

如果没有安装git的话，先安装一下.参考： [http://blog.haohtml.com/archives/10093](http://blog.haohtml.com/archives/10093)

这里用的是提供的库.

```
cd ~/go
go get -tags zmq_3_x github.com/alecthomas/gozmq
```

如果出现

> “# pkg-config –cflags libzmq libzmq libzmq libzmqPackage libzmq was not found in the pkg-config search path.Perhaps you should add the directory containing \`libzmq.pc’to the PKG\_CONFIG\_PATH environment variableNo package ‘libzmq’ foundPackage libzmq was not found in the pkg-config search path.Perhaps you should add the directory containing \`libzmq.pc’to the PKG\_CONFIG\_PATH environment variableNo package ‘libzmq’ foundPackage libzmq was not found in the pkg-config search path.Perhaps you should add the directory containing \`libzmq.pc’to the PKG\_CONFIG\_PATH environment variableNo package ‘libzmq’ foundPackage libzmq was not found in the pkg-config search path.Perhaps you should add the directory containing \`libzmq.pc’to the PKG\_CONFIG\_PATH environment variableNo package ‘libzmq’ foundexit status 1”

错误,提示没有在pkgconfig路径找到libzmq.pc文件，主要是pkgconfig路径的问题，只要配置一下pkgconfig目录给用户环境变量PKG\_CONFIG\_PATH即可．

```
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
```

**四．程序开发**

直接将提供的两段代码写到zmqServer.go 和 zmqClient.go两个文件即可．(也可以直接下载提供的实例 [https://github.com/alecthomas/gozmq/tree/master/examples](https://github.com/alecthomas/gozmq/tree/master/examples))

这个时候开两个终端．一个执行服务端，一个执行客户端．

server端：

```
go run zmqServer.go
```

注意这个时候，不会有什么东西输出的，因为客户商没有任何写入操作．下面执行客户端．注意看一下server端的变化

客户端:

```
go run zmqClient.go
```

这个时候会发现客户端有10个写入操作(生产者)，而服务端有10个输出操作(消费者).说明一切正常，否则请检查上面的操作是否有误．

[![zmq_client](https://blogstatic.haohtml.com//uploads/2023/09/zmq_client.png)][2]

[![zmq_server](https://blogstatic.haohtml.com//uploads/2023/09/zmq_server.png)][3]
至于下面的操作，看开发者怎么使用了．

[1]: http://blog.haohtml.com/wp-content/uploads/2013/10/linux_go_env.png
[2]: http://blog.haohtml.com/wp-content/uploads/2013/10/zmq_client.png
[3]: http://blog.haohtml.com/wp-content/uploads/2013/10/zmq_server.png