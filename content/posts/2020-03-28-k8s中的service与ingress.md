---
title: k8s中的Service与Ingress
author: admin
type: post
date: 2020-03-28T09:40:26+00:00
url: /archives/19945
categories:
 - 系统架构
tags:
 - k8s

---
集群中的服务要想向外提供服务，就不得不提到Service和Ingress。 下面我们就介绍一下两者的区别和关系。

# Service 

必须了解的一点是 Service 的访问信息在 Kubernetes 集群内是有效的，集群之外是无效的。

Service可以看作是一组提供相同服务的Pod对外的访问接口。借助Service，应用可以方便地实现服务发现和负载均衡。对于Service 的工作原理请参考

当需要从集群外部访问k8s里的服务的时候，方式有四种：`ClusterIP`（默认）、`NodePort`、`LoadBalancer`、`ExternalName` 。

**下面我们介绍一下这几种方式的区别**

## **一、ClusterIP** 

该方式是指通过集群的内部 IP 暴露服务，但此服务只能够在集群内部可以访问，这种方式也是默认的 ServiceType。

我们先看一下最简单的Service定义

```
apiVersion: v1
kind: Service
metadata:
  name: hostnames
spec:
  selector:
    app: hostnames
  ports:
  - name: default
    protocol: TCP
    port: 80
    targetPort: 9376
```

这里我使用了 `selector` 字段来声明这个 Service 只携带了 `app=hostnames` 标签的 Pod。并且这个 Service 的 `80` 端口，代理的是 Pod 的 `9376` 端口。

我们定义一个 Deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hostnames
spec:
  selector:
    matchLabels:
      app: hostnames
  replicas: 3
  template:
    metadata:
      labels:
        app: hostnames
    spec:
      containers:
      - name: hostnames
        image: k8s.gcr.io/serve_hostname
        ports:
        - containerPort: 9376
          protocol: TCP
```

这里我们使用的 webservice 镜像为 `k8s.gcr.io/serve_hostname`，其主要提供输出当前服务器的 `hostname` 的功能，这里声明的`Pod`份数是 `3` 份，此时如果我们依次访问 `curl 10.0.1.175:80` 的话，会发现每次响应内容不一样，说明后端请求了不同的 pod 。这是因为 Service 提供的负载均衡方式是 `Round Robin`。

这里的 `10.0.1.175` 是当前集群的IP，俗称为 `VIP`，是 Kubernetes 自动为 Service 分配的。对于这种方式称为 `ClusterIP 模式的 Service`。

## **二、NodePort** 

通过每个 Node 上的 IP 和静态端口（NodePort）暴露服务。NodePort 服务会路由到 ClusterIP 服务，这个 ClusterIP 服务会自动创建。通过请求 `NodeIP:Port`，可以从集群的外部访问一个 NodePort 服务。

```
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  type: NodePort
  ports:
  - nodePort: 8080
    targetPort: 80
    protocol: TCP
    name: http
  - nodePort: 443
    protocol: TCP
    name: https
  selector:
    run: my-nginx
```

**Service描述文件**
`spec.type` 声明Service的type
`spec.selector` 这个Service将要使用哪些Label，本例中指所有具有 `run: my-nginx` 标签的Pod。
`spec.ports.nodePort` 表示此Service将会监听8080端口，并将所有监听到的请求转发给其管理的Pod。
`spec.ports.targetPort` 表示此Service监听到的8080端口的请求都会被转发给其管理的Pod的80端口。此字段可以省略，省略后其值会被设置为`spec.ports.port`的值。

如果你未指定 `spec.ports.nodePort` 的话，则系统会随机选择一个范围为 `30000-32767` 的端口号使用。

这时要访问这个Service的话，只需要通过访问

```
<任何一台宿主机器的IP>:8080
```

这样可以访问到某一个被代理Pod的80端口。后端的Pod并不是一定在当前Node上，有可能你访问的Node1:8080，而后端对应的Pod是在Node2上。

可以看到这种方式很好理解，类似于平时我们使用docker部署容器应用后，将容器服务端口暴露出来宿主端口就可以了。

## **三、LoadBalancer** 

使用云提供商的负载均衡器向外部暴露服务。 外部负载均衡器可以将流量路由到自动创建的 `NodePort` 服务和 `ClusterIP` 服务上（ [官方解释](https://kubernetes.io/zh/docs/concepts/services-networking/service/#publishing-services-service-types)）。

该模式需要底层云平台（例如GCE、亚马孙AWS）支持。

```
---
kind: Service
apiVersion: v1
metadata:
  name: example-service
spec:
  ports:
  - port: 8765
    targetPort: 9376
  selector:
    app: example
  type: LoadBalancer
```

创建这个服务后，系统会自动创建一个外部负载均衡器，其端口为8765, 并且把被代理的地址添加到公有云的负载均衡当中。

## **四、ExternalName** 

创建一个dns别名指到service name上，主要是防止service name发生变化，要配合dns插件使用。

通过返回 CNAME 和它的值，可以将服务映射到 `externalName` 字段的内容（例如 `foo.bar.example.com`）。
没有任何类型代理被创建，这只有 Kubernetes 1.7 或更高版本的 kube-dns 才支持。

```
kind: Service
 apiVersion: v1
 metadata:
   name: my-service
 spec:
   type: ExternalName
   externalName: my.database.example.com
```

指定了一个 `externalName=my.database.example.com` 的字段。而且你应该会注意到，这个 YAML 文件里不需要指定 selector。
这时候，当你通过 Service 的 DNS 名字访问它的时候，比如访问：`my-service.default.svc.cluster.local`。
那么，Kubernetes 为你返回的就是 `my.database.example.com`。所以说，`ExternalName` 类型的 Service，其实是在 `kube-dns` 里为你添加了一条 CNAME 记录。
这时访问 `my-service.default.svc.cluster.local` 就和访问 `my.database.example.com` 这个域名是一个效果了。

# Ingress 

上面我们提到有一个叫作 `LoadBalancer` 类型的 `Service`，它会为你在 Cloud Provider（比如：Google Cloud 或者 OpenStack）里创建一个与该 Service 对应的负载均衡服务。但是，相信你也应该能感受到，由于每个 Service 都要有一个负载均衡服务，所以这个做法实际上既浪费成本又高。作为用户，我其实更希望看到 Kubernetes 为我内置一个全局的负载均衡器。然后，通过我访问的 URL，把请求转发给不同的后端 Service。这种全局的、为了代理不同后端 Service 而设置的负载均衡服务，就是 Kubernetes 里的 `Ingress` 服务。

Ingress 的功能其实很容易理解：所谓 Ingress 就是 Service 的“Service”，这就是它们两者的关系。

```
    internet
        |
   [ Ingress ]
   --|-----|--
   [ Services ]
```

通过使用 Kubernetes 的 Ingress 来创建一个统一的负载均衡器，从而实现当用户访问不同的域名时，访问后端不同的服务。![](https://blogstatic.haohtml.com/uploads/2021/04/d2b5ca33bd970f64a6301fa75ae2eb22-8.png)Ingress 负载均衡

可以将 `Ingress` 配置为服务提供外部可访问的 URL、负载均衡流量、终止 SSL/TLS，以及提供基于名称的虚拟主机等能力。 [Ingress 控制器][1] 通常负责通过`负载均衡器`来实现 Ingress，尽管它也可以配置边缘路由器或其他前端来帮助处理流量。

你必须具有 [`Ingress 控制器`][1] 才能满足 `Ingress` 的要求。 仅创建 Ingress 资源本身没有任何效果。你可能需要部署 Ingress 控制器，例如 [`ingress-nginx`][2]。 你可以从许多 [Ingress 控制器][1] 中进行选择。

假如我现在有这样一个站点：`https://cafe.example.com`。其中 `https://cafe.example.com/coffee`，对应的是“咖啡点餐系统”。而 `https://cafe.example.com/tea`，对应的则是“茶水点餐系统”。这两个系统，分别由名叫 `coffee` 和 `tea` 这样两个 Deployment 来提供服务，可以看到这是一种经典的扇出（fanout）行为。

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: cafe-ingress
spec:
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea
        backend:
          serviceName: tea-svc
          servicePort: 80
      - path: /coffee
        backend:
          serviceName: coffee-svc
          servicePort: 80
```

最值得我们关注的，是 `rules` 字段。在 Kubernetes 里，这个字段叫作：`IngressRule`。
IngressRule 的 Key，就叫做：`host`。它必须是一个标准的域名格式（Fully Qualified Domain Name）的字符串，而不能是 IP 地址。

**Ingress 规则**

每个 HTTP 规则都包含以下信息：

 * `host`。可选项。如果未指定 `host`，则该规则适用于通过指定 IP 地址的所有入站 HTTP 通信。 如果提供了 `host`，则 `rules` 适用于该 `host`。
 * `paths` 路径列表 paths（例如，`/testpath`）,每个路径都有一个由 `serviceName` 和 `servicePort` 定义的关联后端。 在负载均衡器将流量定向到引用的服务之前，主机和路径都必须匹配传入请求的内容。
 * `backend`（后端）是 [Service 文档][3]中所述的服务和端口名称的组合。 与规则的 `host` 和 `path` 匹配的对 Ingress 的 HTTP（和 HTTPS ）请求将发送到指定对应的 `backend`。

通常在 Ingress 控制器中会配置 `defaultBackend`（默认后端），以服务于任何不符合规约中 `path` 的请求。

所以在我们的例子里，我定义了两个 path，它们分别对应 `coffee` 和 `tea` 这两个 Deployment 的 Service（即 `coffee-svc` 和 `tea-svc`）。

通过上面的介绍，不难看到所谓 Ingress 对象，其实就是 Kubernetes 项目对“`反向代理`”的一种抽象。

一个 `Ingress` 对象的主要内容，实际上就是一个“反向代理”服务（比如：Nginx）的配置文件的描述。而这个代理服务对应的转发规则，就是 `IngressRule`。

这就是为什么在每条 IngressRule 里，需要有一个 host 字段来作为这条 IngressRule 的入口，然后还需要有一系列 path 字段来声明具体的转发策略。这其实跟 Nginx、HAproxy 等项目的配置文件的写法是一致的。

在实际使用中，我们一般选择一种 `Ingress Controller`, 将其部署在k8s集群中，这样它就会根据我们定义的 Ingress 对象来提供对应的代理功能。

业界常用的各种反向代理项目，比如 Nginx、HAProxy、Envoy、Traefik 等，都已经为 Kubernetes 专门维护了对应的 `Ingress Controller`。

Nginx Ingress Controller 的示例请参考 [https://time.geekbang.org/column/article/69214](https://time.geekbang.org/column/article/69214)

推荐参考官方推荐脚本：

由于k8s的api 一直在不停的变化，如果您按上方配置运行出错的话，建议参考官方文档进行调整

# 参考资料 

 * [https://kubernetes.io/zh/docs/concepts/services-networking/service/](https://kubernetes.io/zh/docs/concepts/services-networking/service/)
 * [https://kubernetes.io/zh/docs/concepts/services-networking/ingress/](https://kubernetes.io/zh/docs/concepts/services-networking/ingress/)
 * [https://kubernetes.io/docs/reference/kubernetes-api/service-resources/](https://kubernetes.io/docs/reference/kubernetes-api/service-resources/)

 [1]: https://kubernetes.io/zh/docs/concepts/services-networking/ingress-controllers
 [2]: https://kubernetes.github.io/ingress-nginx/deploy/
 [3]: https://kubernetes.io/zh/docs/concepts/services-networking/service/