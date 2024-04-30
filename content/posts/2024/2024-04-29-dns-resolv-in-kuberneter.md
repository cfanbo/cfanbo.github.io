---
title: kubernetes 网络中DNS解析原理
date: 2024-04-29T14:28:21+08:00
type: post
toc: true
url: /posts/dns-resolv-in-kubernetes-networking
categories:
  - 程序开发
tags:
  - k8s
  - dns
---

当我们通过域名（例如 www.example.com）访问一个网站时，第一步就是通过DNS服务器找到目的服务器IP地址（例如 93.184.215.14），接着再将请求数据包发送到这个 IP 服务器。

![img](https://blogstatic.haohtml.com/uploads/2024/04/watermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMwNzcyMQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70.jpeg)

而要想通过 DNS 服务器进行域名，必须得先知道 DNS 服务器地址才行，而这一般是通过读取配置文件实现，在 `*nux` 操作系统中，DNS 服务器一般配置在 `/etc/resolv.conf`文件，如

```ini
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

用户可以通过 nameserver 指定多个 DNS 服务器，依次对域名进行解析，如果解析成功，则解析操作立即中止，如果解析不到的话，则将回退到公网 DNS 服务器进行解析，这个公网 DNS 服务器一般是由网络运营商来定的，用户不需要关心。如果公网 DNS 仍解析失败的话，则直接响应域名无法解析，此时用户将无法正常访问域名。

# 什么是 FQDN

在介绍域名解析前，我们先了解一下什么是 FQDN。它是 `Fully Qualified Domain Name` 的缩写,中文称为 "完全限定域名"。

一个 FQDN 由以下部分组成:

1. **主机名(Hostname)**: 如 www、mail、app 等,用于标识特定的主机或服务
2. **域名(Domain Name)**: 如 example.com、company.org 等,用于标识特定的组织或网络域
3. **根域(Root Domain)**: 表示为"."的根域,代表 DNS 域名层次结构的根

一个完整的 FQDN 看起来类似这样:

```ini
www.example.com.
```

注意最后的"."表示根域的结尾。

FQDN 的作用是唯一地标识互联网上的主机名和域名。它能够避免由于域名冲突导致的解析歧义。例如,如果有 `www.example.com` 和 `www.example.net` 两个不同的域名,仅使用 `www` 是不够的,必须使用 FQDN 来精确指定。

在域名解析过程中,DNS 服务器如果收到一个 FQDN,它会直接对这个完整的域名进行查询和解析。如果没有根域,则 DNS 会根据配置附加上不同的搜索域（search 字段）进行查询尝试。因此 FQDN 可以避免搜索域造成的错误解析。

总之,FQDN 提供了一种唯一标识互联网上主机和域名的方式,确保域名解析的准确性,是实现可靠域名解析的关键机制。

# Kubernetes 域名解析

kubernetes 网络中，在 Pod 中访问一个域名（一般是指 service ）时，解析原理也是一样的。只不过相比公网 DNS 来讲，这里使用了私有 DNS 服务器(coredns)。

## DNS 解析原理

在安装 k8s 集群的时，将自动在 `kube-system`命名空间创建两个 `coredns` Pods。

```shell
$ kubectl get pods --selector=k8s-app=kube-dns -n kube-system
NAME                       READY   STATUS    RESTARTS      AGE
coredns-76f75df574-7pbp8   1/1     Running   1 (31m ago)   3d18h
coredns-76f75df574-cp4zg   1/1     Running   1 (31m ago)   3d18h

$ kubectl get svc  -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   3d20h
```

同时集群里通过一个名为 `kube-dns`的 `service` 为集群内域名解析提供 DNS 解析服务。

这里我们创建一个 nginx pod，来看一下对它的 DNS Server 的配置。

```shell
$ kubectl create deployment nginx --image=nginx:1.23-alpine --replicas=2
```

查看 pod

```shell
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS       AGE
nginx-69b68b44b9-rjpcd   1/1     Running   0              120m
nginx-69b68b44b9-xc9rk   1/1     Running   0              120m
```

查看 `/etc/resolv.conf` 配置

```shell
$ kubectl exec pod/nginx-69b68b44b9-xc9rk -- cat /etc/resolv.conf
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

这里一共三类配置，分别为 `search`、 `name server` 和 `options`。

1. search 配置

```
search default.svc.cluster.local svc.cluster.local cluster.local
```

这一行设置了 DNS 搜索域的顺序,当执行单标签名称查询时,DNS 解析器会按照列出的顺序依次附加这些域,尝试进行 FQDN(完全限定域名)解析。

- `default.svc.cluster.local` 是 Kubernetes 集群中的 DNS 域,用于解析同一个命名空间中的服务。
- `svc.cluster.local` 用于解析集群中所有命名空间中的服务。
- `cluster.local` 是整个集群的域。

2. `nameserver 10.96.0.10`:

- 这一行指定了集群 DNS 服务的 IP 地址,一般是 Kubernetes DNS 插件(如 CoreDNS)在集群中的 ClusterIP 服务地址。
- 所有节点上的 DNS 查询都会发送到这个地址。

3. `options ndots:5`

- 这个选项设置了执行 DNS 查询前,判断主机名是否为 FQDN 所需的点号数。
- `ndots:5` 表示如果主机名中包含至少 5 个点号,则会被视为 FQDN,直接查询而不会追加搜索域。
- 如果主机名中点号少于 5 个,则会依次附加上面设置的搜索域进行查询。

这些配置确保了 Kubernetes 集群内的应用可以正确解析集群内部的 DNS 记录,包括服务、Pod 等,同时也支持外部 DNS 解析。由于应用通常使用短名称相互访问,因此设置合理的 DNS 解析行为对集群内的服务发现非常重要。

假如在 `default` 命名空间的 pod 里要访问域名 `mysvc`，DNS 解析器会按 search 配置的多个域的先后顺序依次尝试以下完全限定域名(FQDN)进行查询:

1. `mysvc.default.svc.cluster.local.`
2. `mysvc.svc.cluster.local.`
3. `mysvc.cluster.local.`

解析过程如下:

1. 首先尝试将 `mysvc` 解析为 `mysvc.default.svc.cluster.local.`

   - 因为 `default.svc.cluster.local.` 是 search 列表中的第一个域。这里的 `default` 对应的是当前 pod 所属的命名空间（上面创建 pod 时未指定 namespace）
   - 会在 `mysvc` 后面添加 `.default.svc.cluster.local.` 并查询这个 FQDN

2. 如果第一次查询失败,则尝试 `mysvc.svc.cluster.local.`

   - 在 `mysvc` 后面添加 `.svc.cluster.local.` 这个 search 域重新解析。这里表示不限定 namespace

3. 如果还失败,则再尝试 `mysvc.cluster.local.`

   - 在 `mysvc` 后面添加最后一个 `.cluster.local.` 域

4. 如果所有这些 FQDN 都查询失败,则最后会将 `mysvc` 作为非限定域名发送到上游递归 DNS 服务器进行查询

> DNS 查询可能因为执行查询的 Pod 所在的名字空间而返回不同的结果。 不指定名字空间的 DNS 查询会被限制在 Pod 所在的名字空间内。 要访问其他名字空间中的 Service，需要在 DNS 查询中指定名字空间。参考： https://kubernetes.io/zh-cn/docs/concepts/services-networking/dns-pod-service/#namespaces-of-services

可以看到解析器会依次使用 `mysvc` 加上 `search` 列表中的域构造 FQDN 尝试查询,直到找到结果或遍历完整个 `search` 列表。

这个 `search` 域列表的设计主要目的是:

- 先查询集群内部服务,因为 `default.svc.cluster.local` 和 `svc.cluster.local` 对应的就是集群服务 service
- 最后才查询外部 `cluster.local` 域
- 如果都失败再考虑其他外部域名解析选项

这样可以优先解析集群内部服务,同时也支持外部域名查询,充分满足了 Kubernetes 集群内应用的需求。

我们再来看一下 `nameserver 10.96.0.10`, 这里配置的正是 `svc/kube-dns`的 IP，也就是 DNS 服务器地址，对一个域名进行解析时，需要到这个地址进行查询。

这里我们再创建一个 http pod,作为服务端。

```shell
$ kubectl create ns lab
namespace/lab created
$ kubectl create deploy httpd -n lab --image=httpd:alpine --replicas=1
deployment.apps/httpd created
$ kubectl expose deployment httpd -n lab --port=80
deployment.apps/httpd created


$ kubectl get pod,svc -n lab
NAME                        READY   STATUS    RESTARTS   AGE
pod/httpd-764dbd796-n4bmb   1/1     Running   0          24s

NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/httpd   ClusterIP   10.96.131.72   <none>        80/TCP    15s
```

> Kubernetes 将为 Service 和 Pod 自动创建 DNS 记录。 你可以使用一致的 DNS 名称而非 IP 地址访问 Service。这里我们只讲针对 Service 的情况。

我们在 `default/nginx` Pod 里访问 `lab/httpd`服务。

```shell
$ kubectl exec pod/nginx-69b68b44b9-xc9rk -- curl httpd
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0curl: (6) Could not resolve host: httpd
command terminated with exit code 6
```

由于 nginx 和 httpd 属于不同的命名空间，因此这里需要解析的 FQDN 有三个，分别为 `httpd.default.svc.cluster.local.`、`httpd.svc.cluster.local.` 和 `httpd.cluster.local.`很明显是无法解析的。而它又不是一个完整的域名，因此更不可能在公网 DNS 进行解析。

如果我们指定了命名空间 `lab`，则三个 FQDN 为 `httpd.lab.default.svc.cluster.local.`、`httpd.lab.svc.cluster.local.` 和 `httpd.lab.cluster.local.`。当解析第一个 FQDN 时，返回解析失败；接着再使用第二个 FQDN 解析，此时 DNS 服务器将解析成功结果立即返回给客户端（并中止后续 FQDN 解析）并进行后续请求。

```shell
$ kubectl exec pod/nginx-69b68b44b9-xc9rk -- curl httpd.lab
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<html><body><h1>It works!</h1></body></html>
100    45  100    45    0     0   7137      0 --:--:-- --:--:-- --:--:--  7500
```

可以看到，如果解析的服务与发起请求的客户端同属一个命名空间的话，直接使用第一个 FQDN 即可解析成功，避免后续 FQDN 解析，因此解析效率将是最高的。

## 公网解析流程

如果在 Pod 里访问的域名是一个公网正式域名，如 `www.example.com` , 在集群内部无法解析后，最后只能通过公网 DNS 进行解析，并最终返回解析结果。

> 在公网 DNS 解析时，也是通过 Kubernetes DNS 插件(如 CoreDNS)将其**代理**到配置的上游公网 DNS 递归解析服务器。

整个过程是这样的:

1. 集群内的 Pod 发起一个域名解析请求
2. 该请求首先被发送到 Kubernetes DNS 服务的 ClusterIP (通常是 `10.96.0.10`)
3. Kubernetes DNS 插件(如 CoreDNS)尝试解析该请求
   - 如果是集群内部的服务名或 Pod 的 hostname,它可以直接解析
   - 如果是外部的公网域名,则无法直接解析
4. 对于无法直接解析的域名,Kubernetes DNS 插件会启用**代理模式**
5. 它会将该域名解析请求代理(forward)到预先配置的上游公网 DNS 服务器
6. 上游公网 DNS 服务器(如 8.8.8.8)会递归解析该域名
7. 解析结果会返回给 Kubernetes DNS 插件
8. Kubernetes DNS 插件再将结果返回给最初的请求端

所以 Kubernetes DNS 插件扮演了一个**代理角色**,对于集群内部域名自行解析,对于无法解析的域名则代理到公网 DNS 上进行递归解析。这样就实现了对集群内外服务的统一域名解析支持。

值得注意的是,这个上游公网 DNS 的地址通常是在安装 Kubernetes 集群时预先配置好的,并写入了每个节点的 `/etc/resolv.conf` 文件中,从而为整个集群提供了统一的外部 DNS 解析支持。

## Pod 的 DNS 策略

对于集群 Pod 可以在 `dnsPolicy` 字段中设置不同的 DNS 策略：

- "`Default`": Pod 从运行所在的节点继承名称解析配置。 参考[相关讨论](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/dns-custom-nameservers)获取更多信息。

- "`ClusterFirst`": **默认的 DNS 策略**。与配置的集群域后缀不匹配的任何 DNS 查询（例如 "www.kubernetes.io"） 都会由 DNS 服务器转发到上游名称服务器。集群管理员可能配置了额外的存根域和上游 DNS 服务器。 参阅[相关讨论](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/dns-custom-nameservers) 了解在这些场景中如何处理 DNS 查询的信息。

- "`ClusterFirstWithHostNet`": 对于以 `hostNetwork` 方式运行的 Pod，应将其 DNS 策略显式设置为 "ClusterFirstWithHostNet"。否则，以 hostNetwork 方式和"ClusterFirst"策略运行的 Pod 将会做出回退至

  "Default"策略的行为。

  - 注意：这在 Windows 上不支持。 有关详细信息，请参见[下文](https://kubernetes.io/zh-cn/docs/concepts/services-networking/dns-pod-service/#dns-windows)。

- "`None`": 此设置允许 Pod 忽略 Kubernetes 环境中的 DNS 设置。Pod 会使用其 `dnsConfig` 字段所提供的 DNS 设置。 参见 [Pod 的 DNS 配置](https://kubernetes.io/zh-cn/docs/concepts/services-networking/dns-pod-service/#pod-dns-config)节。

配置示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
    - image: busybox:1.28
      command:
        - sleep
        - "3600"
      imagePullPolicy: IfNotPresent
      name: busybox
  restartPolicy: Always
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
```

对于 Pod 的 DNS 策略，参考 https://kubernetes.io/zh-cn/docs/concepts/services-networking/dns-pod-service/#pod-s-dns-policy

也可以在创建 Pod 时指定 DNS 配置项，此时需要使用`None` 这种 DNS 策略。如 https://raw.githubusercontent.com/kubernetes/website/main/content/zh-cn/examples/service/networking/custom-dns.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: dns-example
spec:
  containers:
    - name: test
      image: nginx
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 192.0.2.1 # 这是一个示例
    searches:
      - ns1.svc.cluster-domain.example
      - my.dns.search.suffix
    options:
      - name: ndots
        value: "2"
      - name: edns0
```

这里手动指定了 dns 配置信息，此时 Pod `dns-example` 里的 `/etc/resolv.conf` 内容将为

```
nameserver 192.0.2.1
search ns1.svc.cluster-domain.example my.dns.search.suffix
options ndots:2 edns0
```

# 参考资料

- https://kubernetes.io/zh-cn/docs/concepts/services-networking/dns-pod-service/
- https://github.com/kubernetes/dns/blob/master/docs/specification.md
