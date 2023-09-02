---
title: 初识kubernetes 组件
author: admin
type: post
date: 2021-04-29T06:05:52+00:00
url: /archives/30007
categories:
 - 系统架构
tags:
 - k8s

---
对于一个刚刚接触kubernetes(k8s)的新手来说，想好更好的学习它，首先就要对它有一个大概的认知，所以本文我们先以全局观来介绍一个kubernetes。

## kubernetes架构 ![8ee9f2fa987eccb490cfaa91c6484f67](https://blogstatic.haohtml.com/uploads/2021/04/56aec9997240192091adad3e14358736-52.png)kubernetes 架构图

kubernets整体可以分为两大部分，分别为 `Master` 和 `Node` ，我们一般称其为节点，这两种角色分别对应着控制节点和计算节点，根据我们的经验可以清楚的知道 Master 是控制节点。

## Master 节点 

控制节点 `Master` 节点由三部分组成，分别为 `Controller Manager` 、 `API Server` 和 `Scheduler` ，它们相互紧密协作，每个部分负责不同的工作职责。

 * `controller-manager` 全称为 kube-controler-manager 组件，主要用来负责容器编排。如一个容器(实际上是pod，pod是最基本的调度单元。一般一个pod里会部署一个容器服务)服务可以指定副本数量，如果实际运行的副本数据与期望的不一致，则会自动再启动几个容器副本，最终实现期望的数量。这个组件，就是一系列控制器的集合。我们可以查看一下 Kubernetes 项目的 [pkg/controller](https://github.com/kubernetes/kubernetes/tree/master/pkg/controller) 目录, 伪代码如下：

```
for {
  实际状态 := 获取集群中对象X的实际状态（Actual State）
  期望状态 := 获取集群中对象X的期望状态（Desired State）
  if 实际状态 == 期望状态{
    什么都不做
  } else {
    执行编排动作，将实际状态调整为期望状态
  }
}
```

 * `api server` 对外提供api服务，用来接收命令进行集群管理。对内负责与etcd注册中心进行通讯，进行一些配置信息的存储与读取
 * `scheduler` 负责调度。如一个容器存放到k8s集群中的哪个node节点最为合适

实际上这三个组件的功能远远多于我们这里描述的。

## Node 节点 

对于节点node，一般我们可以理解为一台物理机或者vm服务器。同样 node 也是由多个组件组成，其中最为重要的是一个 `kubelet` 组件。它负责与容器运行时打交道，一般每个node节点都会安装这个组件。

这里有几个关键的概念

 * `CNI` Container Networking Interface 负责 kubelet 与 网络(Network) 通讯。
 * `CRI` Container Runtime Interface 负责 kubelet 与 运行时(Container Runtime) 通讯。定义了容器运行时的一些关键操作，如启动容器时需要的所有参数。
 * `CSI` Container Storage Interface 负责 kubelet 与 存储(Volume Plugin) 通讯，
 * `OCI` Open Container Initiative 可以看作是一个容器运行时标准。负责 运行时 Runtime 与 操作系统OS 通讯。即把CRI 请求转换成对Linux 操作系统的调用（如 namespace 和 cgroups)

对于CRI 和 OCI 接口的关系变成为下图所示![](https://blogstatic.haohtml.com/uploads/2021/04/3b8a77d4d4dce733afc487ac6997fa00.png)CRI / OCI

对于 CNI 接口来讲，不管你使用的什么容器技术，只要你可以通过 CNI 接口接入到k8s中就没有问题。实现此接口的容器运行时有 `runC` 和 ` [gVisor](https://github.com/google/gvisor) `。而对于 `docker` 项目来讲则需要借助 `docker-shim` 组成来实现，这也就导致了 《 [Kubernetes 将弃用 Docker](https://mp.weixin.qq.com/s/GHjvvTJ8ZerIyCqXB1BSUQ) 》。![](https://blogstatic.haohtml.com/uploads/2021/04/8a7137e8918a8eb5d67f1dc7ddad07ca.png)docker

上图可以看出 `docker` 并没有实现 `CRI` 接口，而 k8s 项目中的 kubelet 又要使用这个接口通讯，所以只能使用 `docker-shim` 组件作为桥接服务，将 `CRI` 接口指令转换成 `docker api`，然后与 `docker` 进行通讯。官方不打算继续维护 docker-shim 组件，这也正是导致 docker 被 k8s 弃用的主要原因。

对于 CRI 接口的定义![](https://blogstatic.haohtml.com/uploads/2023/07/d2b5ca33bd970f64a6301fa75ae2eb22-4.png)

 **CRI 分为两组：**
第一组，是 RuntimeService。它提供的接口，主要是跟容器相关的操作。比如，创建和启动容器、删除容器、执行 exec 命令等等。
第二组，则是 ImageService。它提供的接口，主要是容器镜像相关的操作，比如拉取镜像、删除镜像等等。

`kubelet` 组件还通过 gRPC 协议与 `Device Plugin` 插件进行通讯, 实现k8s项目管理GPU等物理设备核心硬件。

而 kubelet 的另一个重要功能，则是调用 `网络插件` 和 `存储插件` 为容器配置网络和持久化存储。

由此可以看出 kubelet 组件在 Node 节点的重要性，每个 Node 节点也只有一个 `kubelet` 组件。

总体来讲，对于 k8s里 containerd 和 runC 的关系，Containerd是一个用于管理容器生命周期的守护进程，它作为一个容器运行时，负责管理容器的创建、运行和销毁等操作。而runC是由Open Container Initiative（OCI）开发的一个用于容器运行时的工具，它负责解析并执行OCI容器规范定义的容器配置和运行时环境。runC实际上是containerd的默认容器运行时，用于实际运行和管理容器。 从整体上看，Containerd和runC是Kubernetes中容器运行时的两个组件，Containerd实际上使用runC来执行容器。这种分层架构使得Kubernetes的容器运行时非常灵活和可扩展，因为容器运行时和容器管理器可以独立地进行升级和替换。

本文主要从全局观来讲了一个 kubernetes 的主要架构，相信以后随着更深入的学习对它们每个组件的理解会更加深刻。

## 参考资料 

 * [https://github.com/kubernetes/kubernetes/blob/242a97307b34076d5d8f5bbeb154fa4d97c9ef1d/docs/design/architecture.md](https://github.com/kubernetes/kubernetes/blob/242a97307b34076d5d8f5bbeb154fa4d97c9ef1d/docs/design/architecture.md)
 * [https://kubernetes.io/zh-cn/blog/2022/02/17/dockershim-faq/](https://kubernetes.io/zh-cn/blog/2022/02/17/dockershim-faq/)
 * [K8s、Docker、CRI、OCI 之间的爱恨情仇](https://blog.csdn.net/yangchao1125/article/details/111209995)
 * [为什么 Kubernetes 要替换 Docker](https://draveness.me/whys-the-design-kubernetes-deprecate-docker/)
 * [OCI,CRI到kubernetes runtime](https://www.jianshu.com/p/c7748893ab00)
 * [K8s 网络之深入理解 CNI](https://zhuanlan.zhihu.com/p/450140876)
 * [Kubernetes网络模型与CNI网络插件](https://time.geekbang.org/column/article/67351)
 * [解读 CRI 与 容器运行时](https://time.geekbang.org/column/article/71499)