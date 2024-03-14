---
title: 使用kubectl create service 命令无法为pod创建service问题
date: 2024-01-12T11:22:20+08:00
type: post
toc: true
url: "/posts/k8s-Unable-to-create-service-for-deployment"
categories:
- 程序开发
tags:
- k8s
---



在做一个试验时，无意中发现使用通过 `kubectl create service` 命令无法为一个通过 `deployment` 创建出来的pod创建对应的 `service`， 顿时有点奇怪，经过分析才明白怎么回事，这里将过程记录一下。

这里需要说明一下，本文操作全部是通过 `kubectl create` 命令来完成的，并没有使用 `kubectl apply -f pod.yaml` 这种方式。



这里先创建一个实现命名空间 `lab`

```shell
$ kubectl create ns lab
```



首先创建一个`deployment` 对象

````shell
$ kubectl create deployment test --image=nginx:1.23-alpine --replicas=2 --port=80 -n lab
````

确认创建成功

```shell
$ kubectl get deploy,pod -n lab
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/test   2/2     2            2           16s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/test-8544f5598b   2         2         2       16s

NAME                        READY   STATUS    RESTARTS   AGE
pod/test-8544f5598b-7tfq4   1/1     Running   0          16s
pod/test-8544f5598b-d48js   1/1     Running   0          16s
```

这里我们成功创建了一个 `test`的 `deployment`  对象，而由此创建的 `Pod.Lables` 也都有 `app=test` 标签对。

注意这里的名称是 `test`  。

现在我们再用 `kubectl create service` 命令为这些 pod 创建对应的 `service` (这里并没有指定任何Lables)

```shell
$ kubectl create service clusterip mysvc --tcp=80:80 -n lab
```

在同一个`namespace` 里创建一个 `mysvc `的 `service`, 并指定 `serviceType=clusterip`， 对应的端口号与上面保持一致。此命令表示当访问 `服务:80` 时，实现访问的是`容器:80`。

确认一下服务创建成功

```shell
$ kubectl get svc -n lab
NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
mysvc   ClusterIP   10.96.147.26   <none>        80/TCP    100s
```

看似是没什么问题的。

我们在上面的容器里访问一下这个服务，验证一下。

```shell
$ kubectl exec pod/test-8544f5598b-7tfq4 -- curl http://mysvc:80
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
curl: (7) Failed to connect to mysvc port 80 after 4 ms: Couldn't connect to server
command terminated with exit code 7
```

可以看到这个服务无法访问，哪里出问题了呢？

我们知道 `service` 与 `pod` 的关系是通过 `EndPoints` 进行关系绑定的，我们先看一下这个服务绑定的 `EndPoints` 有哪些?

```shell
$ kubectl get ep mysvc
NAME    ENDPOINTS   AGE
mysvc   <none>      7m26s
```

发现 `ENDPOINTS` 值竟然是空的，说明 `service`  并未成功与 `Pod` 建立映射关系。

我们看一下这个 `service` 的定义

```shell
$ kubectl describe svc mysvc
kubectl describe svc mysvc
Name:              mysvc
Namespace:         lab
Labels:            app=mysvc
Annotations:       <none>
Selector:          app=mysvc
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.147.26
IPs:               10.96.147.26
Port:              80-80  80/TCP
TargetPort:        80/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>
```

发现 `Labels` 标签竟然是 `app=mysvc` ，并不是我们想要的 `app=test`, 由此说明我们上面创建 `service` 的命令是错误的。

原因找到了，我们指定一下 `selector` 标签试一下

```shell
$ kubectl delete svc mysvc -n lab // 删除已存在的 service

$ kubectl create service clusterip mysvc --tcp=80:80 --selector=app=test -n lab
error: unknown flag: --selector
See 'kubectl create service clusterip --help' for usage.
```

发现不支持 `--selector`。没办法只有将 `service` 名称与 `deployment` 名称保持一致了，我们试一下

```shell
kubectl create service clusterip test --tcp=80:80 -n lab
```

将命令中的 `mysvc` 修改为 `test`，执行并确认

```shell
$ kubectl get svc -n lab
NAME   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
test   ClusterIP   10.96.55.44   <none>        80/TCP    80s

$ kubectl get ep test -n lab
NAME             ENDPOINTS           AGE
test   10.244.1.6:80,10.244.1.7:80   17s
```

可以看到 test 服务以及对应的 `EndPoints`，它们正是上面创建的Pod的IP地址，说明service 与 pod 的关系建立成功。

我们在容器里访问一下这个服务测试一下，注意现在服务名称已由 `mysvc` 改为 `test` 

```shell
$ kubectl exec pod/test-8544f5598b-7tfq4 -- curl http://test:80
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   615  100   615    0     0  85750      0 --:--:-- --:--:-- --:--:-- 87857
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

一切正常。



**总结**

如果要使用 `kubectl create service` 命令为 `deployment` 创建服务的话，只能将 `service` 的

名称与其保持一致，使用方法有些不太灵活。

有一种快速为 `deployment` 创建 `service` 的命令

```shell
$ kubectl expose deployment test --name=mysvc --port=80 --target-port=80 -n lab
```

确认是否成功

```shell
$ kubectl get ep -n lab
NAME    ENDPOINTS                     AGE
mysvc   10.244.1.6:80,10.244.1.7:80   4s
test    10.244.1.6:80,10.244.1.7:80   9m49s
```

可以看到两个service 的 `EndPoints` 都是对应Pod地址，这里使用 `kubectl expose` 要灵活一些，另外它还支持对多种资源对象的暴露，而 `deployment` 只是其中一种。

```
  # Create a service for a replicated nginx, which serves on port 80 and connects to the containers on port 8000
  kubectl expose rc nginx --port=80 --target-port=8000

  # Create a service for a replication controller identified by type and name specified in "nginx-controller.yaml",
which serves on port 80 and connects to the containers on port 8000
  kubectl expose -f nginx-controller.yaml --port=80 --target-port=8000

  # Create a service for a pod valid-pod, which serves on port 444 with the name "frontend"
  kubectl expose pod valid-pod --port=444 --name=frontend

  # Create a second service based on the above service, exposing the container port 8443 as port 443 with the name
"nginx-https"
  kubectl expose service nginx --port=443 --target-port=8443 --name=nginx-https

  # Create a service for a replicated streaming application on port 4100 balancing UDP traffic and named 'video-stream'.
  kubectl expose rc streamer --port=4100 --protocol=UDP --name=video-stream

  # Create a service for a replicated nginx using replica set, which serves on port 80 and connects to the containers on
port 8000
  kubectl expose rs nginx --port=80 --target-port=8000

  # Create a service for an nginx deployment, which serves on port 80 and connects to the containers on port 8000
  kubectl expose deployment nginx --port=80 --target-port=8000
```



