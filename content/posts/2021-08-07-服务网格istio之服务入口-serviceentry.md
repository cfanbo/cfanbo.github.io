---
title: 服务网格Istio之服务入口 ServiceEntry
author: admin
type: post
date: 2021-08-07T07:36:11+00:00
url: /archives/30931
categories:
 - 系统架构
tags:
 - istio

---
使用服务入口（Service Entry） 来添加一个服务入口到 Istio 内部维护的服务注册中心。添加了服务入口后，Envoy 代理可以向服务发送流量，就好像它是网格内部的服务一样，可参考 [https://istio.io/latest/zh/docs/concepts/traffic-management/#service-entries](https://istio.io/latest/zh/docs/concepts/traffic-management/#service-entries)。

简单的理解就是允许内网向外网服务发送流量请求，但你可能会说正常情况下在pod里也是可以访问外网的，这两者有什么区别呢?

确实默认情况下，Istio 配置 Envoy 代理可以将请求传递给外部服务。但是无法使用 Istio 的特性来控制没有在网格中注册的目标流量。这也正是 ServiceEntry 真正发挥的作用，通过配置服务入口允许您管理运行在网格外的服务的流量。

此外，可以配置虚拟服务和目标规则，以更精细的方式控制到服务条目的流量，就像为网格中的其他任何服务配置流量一样。

为了更好的理解这一块的内容，我们先看一下普通POD发送请求的流程图![](https://blogstatic.haohtml.com/uploads/2021/08/d2b5ca33bd970f64a6301fa75ae2eb22.png)普通 Pod 请求

# 创建 ServiceEntry 资源 {#%E5%88%9B%E5%BB%BA-serviceentry-%E8%B5%84%E6%BA%90.wp-block-heading}

举例来说：

svc-entry.yaml

```
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: svc-entry
spec:
  hosts:
  - "www.baidu.com"
  ports:
  - number: 80
    name: http
    protocol: HTTP
  - number: 443
    name: https
    protocol: HTTPS
  location: MESH_EXTERNAL
  resolution: DNS

```

该 ServiceEntry 资源定义了一个外部网站 [www.baidu.com][1] 服务，并将它纳入到 Istio 内部维护的服务注册表（服务网格）中。

创建资源

```
$ kubectl apply -f svc-entry.yaml
serviceentry.networking.istio.io/svc-entry created

```

现在我们已经创建了一个 ServiceEntry, 默认在 default 命名空间。

```
root@vm:/home/sxf/service-entry# kubectl get se
NAMESPACE   NAME        HOSTS               LOCATION        RESOLUTION   AGE
default     svc-entry   ["www.baidu.com"]   MESH_EXTERNAL   DNS          13s

```

为了测试这个效果，我们再创建一个pod，然后在pod里访问上面创建的 ServiceEntry。

# 创建请求客户端资源 {#%E5%88%9B%E5%BB%BA%E8%AF%B7%E6%B1%82%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%B5%84%E6%BA%90.wp-block-heading}

为了测试这里创建一个 deployment，并指定了pod数量1个,将其添加到服务网格中

svc-entry-client.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: svc-entry-client
spec:
  replicas: 1
  selector:
    matchLabels:
      app: svc-entry-client-app
  template:
    metadata:
      labels:
        app: svc-entry-client-app
    spec:
      containers:
      - name: busybox
        image: busybox
        imagePullPolicy: IfNotPresent
        command: ["/bin/sh", "-c", "sleep 3600"]

```

创建资源：

```
root@vm:/home/sxf/service-entry# kubectl apply -f svc-entry-client.yaml
deployment.apps/svc-entry-client created

```

默认使用的 default 命名空间，且已默认启用了自动注入功能，否则需要手动注入，执行命令为

```
$ istioctl kube-inject -f svc-entry-client.yaml | kubectl apply -f -
```

查看注入结果

```
root@vm:/home/sxf/service-entry# kubectl get pods -n default
NAME                                READY   STATUS    RESTARTS   AGE
svc-entry-client-67dd4c7794-qkcxg   2/2     Running   0          4m49s

```

READY 字段的值为2/2，表示当前pod中有两个容器, 说明代理容器已经通过 sidecar 模式注入成功。

我们这里确认一下，可以从下面的 Containers 字段中看到有一个busybox容器，还有一个刚刚注入的 istio-proxy 代理容器。

```
root@vm:/home/sxf/service-entry# kubectl describe pod/svc-entry-client-67dd4c7794-qkcxg
...
Init Containers:
  istio-init:
    Container ID:  docker://9c9025d0461461c4cebf27ae8aa81570979682eac5f38899016b383fb88d259d
...
Containers:
  busybox:
    Container ID:  docker://7bd46baf68bb00b1c43e128adc4a051f46d5fa9b7aab0207885195e8c596727b
  istio-proxy:
    Container ID:  docker://6d21efecc66bad9a7ea0a76b83d893db6bd60adb663f7fc812964cbf0aa6ce3b
...

```

此时的 client 已经处于服务网络之中。对于上面的 istio-init 这个容器是什么什么的，有兴趣的话，可以参考相关文档。

# 测试效果 {#%E6%B5%8B%E8%AF%95%E6%95%88%E6%9E%9C.wp-block-heading}

我们在pod 里测试一下效果。

```
root@vm:/home/sxf/service-entry# kubectl exec -it svc-entry-client-67dd4c7794-qkcxg -- wget -q -O - http://www.baidu.com
<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Com...</div> </body> </html

```

访问成功。

为了通过对比了解它的作用，我们对 svc-entry.yaml 文件做一些修改。将 Service discovery mode 字段resolution 值由 DNS 修改为 STATIC, 同时再添加一个 endpoint 字段。
svc-entry.yaml

```
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: svc-entry
spec:
  hosts:
  - "www.baidu.com"
  ports:
  - number: 80
    name: http
    protocol: HTTP
  - number: 443
    name: https
    protocol: HTTPS
  location: MESH_EXTERNAL
  resolution: STATIC
  endpoints:
  - address: 192.168.0.100

```

重新应用svc-enry.yaml

```
root@vm:/home/sxf/service-entry# kubectl apply -f svc-entry.yaml
serviceentry.networking.istio.io/svc-entry configured

root@vm:/home/sxf/service-entry# kubectl exec -it svc-entry-client-67dd4c7794-qkcxg -- wget -q -O - http://www.baidu.com
wget: server returned error: HTTP/1.1 503 Service Unavailable
command terminated with exit code 1

```

再次测试，发现已经无法访问。这里503错误是由代理返回的。

出现此现象的主要原因就是 ServiceEntry 在发挥作用。当域名通过istio-proxy（envoy）的时候，会将流量自动定位到endpoints的IP列表上，所以就出现了无法访问的现象。

可以看到我们已经实现到对服务的流量管理。

将外部服务通过 ServiceEntry 加入服务网格后大概是这个样子![](https://blogstatic.haohtml.com/uploads/2021/08/d2b5ca33bd970f64a6301fa75ae2eb22-3.png)Service Entry Pod 请求

> 此实验主要是面向网格外部的服务，即 `location: MESH_EXTERNAL` 的情况。如果是网格内部的服务的话，则为 `MESH_INTERNAL`

我们先看下关键字段resolution的定义, 它用来定义服务发现的模式，它有三种值，分别为 DNS、STATIC 和 NONE。

> 对于 DNS 模式来说，当 pod 里的应用发送 [www.baidu.com](http://www.baidu.com/) 请求的时候，会使用dns来查找域名指定的服务器，在这种情况下，就和我们平时打开一个网站逻辑完全一样，所以肯定是可以正常访问的。
>
>
> 而对于 STATIC 模式来说，当 pod 里的应用发送 [www.baidu.com](http://www.baidu.com/) 请求的时候，会向请求转到 endpoints 字段指定的IP 服务器，我们这里指向的是一个内网IP地址，所以无法正常响应。对于endpoints字段可以多个，并允许对其根据 LB 权重设置, 有兴趣的可以了解下 [https://istio.io/latest/zh/docs/reference/config/networking/service-entry/#ServiceEntry-Endpoint](https://istio.io/latest/zh/docs/reference/config/networking/service-entry/#ServiceEntry-Endpoint)
>
> 对于 NONE 模式来说，要小心使用，在这种模式下，如果未指定任何IP地址的话，则默认将允许发送到任意IP上。

# 清理 {#%E6%B8%85%E7%90%86.wp-block-heading}

```
$ kubectl delete -f svc-entry-client.yaml
$ kubectl delete -f svc-entry.yaml

```

# 总结 {#%E6%80%BB%E7%BB%93.wp-block-heading}

ServiceEntry 主要用来将一些从内部流向外部的流量进行拦截，并根据配置进行流量转发。如果想要以更细粒度的方式控制到服务入口的流量，可能通过配置虚拟服务和目标规则来实现。

推荐阅读： [Envoy中查看ServiceEntry注入信息](https://mp.weixin.qq.com/s/qrjWHks9Zc2WjAl9WWPEDA)

# 参考 {#%E5%8F%82%E8%80%83.wp-block-heading}

 * [https://istio.io/latest/zh/docs/concepts/traffic-management/#service-entry-example](https://istio.io/latest/zh/docs/concepts/traffic-management/#service-entry-example)
 * [https://istio.io/latest/zh/docs/reference/config/networking/service-entry/#ServiceEntry-Endpoint](https://istio.io/latest/zh/docs/reference/config/networking/service-entry/#ServiceEntry-Endpoint)
 * [https://www.cnblogs.com/haoyunlaile/p/12937978.html](https://www.c)

 [1]: http://www.baidu.com/