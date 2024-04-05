---
title: Raspberry Pi 安装Kubernetes
date: 2024-02-02T21:53:47+08:00
type: post
toc: true
url: /posts/install-kubernetes-in-raspberry-pi
categories:
  - 程序开发
tags:
  - k8s
  - metallb
  - ingress-nginx
  - ingress
  - containerd
---

这里是 arm64 架构，**树莓派 4B**, 四核八 G 内存 配置，系统为 Ubuntu 22.04.1 LTS

```shell
$ uname -a
Linux ubuntu 5.15.0-1049-raspi #52-Ubuntu SMP PREEMPT Thu Mar 14 08:39:42 UTC 2024 aarch64 aarch64 aarch64 GNU/Linux

$ cat /etc/issue
Ubuntu 22.04.1 LTS \n \l
```

## 环境检查

由于 k8s 会使用 8080 和 6443 这两个端口，因此要保证端口可用，然后禁用 swap。

```
sudo swapoff -a
```

最后对安装环境初始化，参考 https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/

## 安装 Docker

参考 https://docs.docker.com/engine/install/ubuntu/

安装成功后，修改 `cgroupdriver` 为 `systemd`，同时为了国内访问 docker 镜像加速，需要设置一下 aliyun 的 docker 镜像地址

```shell
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://gfxrbz51.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

用 `docker info` 可以验证镜像是否设置成功。

> 由于依赖关系，将自动安装 CNI containerd

## 配置 containerd

由于我们在安装 Docker 时，由于依赖关系已自动安装 `containerd.io`，这里我们只需要重新生成一份默认配置即可。

1. 生成默认配置

   ```shell
   containerd config default > /etc/containerd/config.toml
   ```

2. 配置 containerd 的 systemd cgroup 驱动

   ```toml
   [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
     ...
     [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
       SystemdCgroup = true
   ```

3. 配置 sandbox_image 沙箱 pause 镜像

   ```toml
   [plugins."io.containerd.grpc.v1.cri"]
     sandbox_image = "registry.k8s.io/pause:3.2"
   ```

   改为

   ```toml
   [plugins."io.containerd.grpc.v1.cri"]
     sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"
   ```

   这个镜像会在安装 k8s 时被自动下载到本机。

最后重启服务

```shell
systemctl restart containerd
```

## 安装 kubectl/kubelet/kubeadm

参考： https://developer.aliyun.com/mirror/kubernetes

## 初始化集群

```shell
kubeadm init --image-repository registry.aliyuncs.com/google_containers --pod-network-cidr=10.244.0.0/16 --v=5
```

这里指定 `--image-repository`参数表示从阿里云下载镜像，同时指定 Pod 分配的 IP 范围，`--v`表示打印安装日志信息，方便出错排查。

在安装过程中将自动下载所需要的镜像，另外也可以先手动下载集群需要的镜像，再执行 `kubeadm init` 命令

```shell
kubeadm config images pull  --image-repository registry.aliyuncs.com/google_containers
```

查看集群 pod 状态

```shell
$ kubectl get pod -A
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-66f779496c-kvrlt         0/1     Pending   0          18s
kube-system   coredns-66f779496c-pqvx2         0/1     Pending   0          18s
kube-system   etcd-ubuntu                      1/1     Running   0          35s
kube-system   kube-apiserver-ubuntu            1/1     Running   0          32s
kube-system   kube-controller-manager-ubuntu   1/1     Running   0          32s
kube-system   kube-proxy-9l7dt                 1/1     Running   0          18s
kube-system   kube-scheduler-ubuntu            1/1     Running   0          30s
```

由于还没有安装 CNI 插件，因此这里 `coredns` 为 `Pending` 状态。同样节点也是 `NotReady` 状态

```shell
$ kubectl get nodes
NAME     STATUS     ROLES           AGE   VERSION
ubuntu   NotReady   control-plane   60s   v1.28.8
```

默认情况下，对于 master 节点是不允许 Pod 调度到这个节点的，如果你想取消这个限制，可以设置将 master 节点移除污点。

```shell
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

对于生产环境一般都多由个节点组成，因此不推荐将 Pod 调度到 master 节点。

## 安装网络插件

常用的网络插件有 [Flannel](https://github.com/flannel-io/flannel) 、[Calico](https://www.tigera.io/project-calico/) 和 [Cilium](https://cilium.io/)，功能从左到右依次强大，不过对于小集群，我们使用 `Flannel` 足够了，如果对性能要求较高的话，则推荐 `Cilium`。

```shell
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

如果上面指定的 podCIDR 不是 `10.244.0.0/16`, 则需要先把文件下载到本地进行修改才可以。

如果安装失败，可能需要手动创建一个 `/run/flannel/subnet.env` 文件

```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.0.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=false
```

再次查看 pod 和 node ，会发现变为正常状态了，如果仍有问题可以试着重启一下 containerd 看看。

如果在安装 Flannel 时遇到 `Error Registering network: operation not supported` 错误，这是由于缺少 vxlan 模块引起的 (https://github.com/k3s-io/k3s/issues/4234#issuecomment-947954002)，解决办法

```shell
sudo apt install -y linux-modules-extra-raspi && reboot
```

重新执行上面的命令查看 pod 状态

## 查看本地镜像

上面下载的镜像，使用 `docker images` 是无法看到的，需要使用 crictl 命令代替，它几乎支持 docker 所有命令。

```
$ crictl images
WARN[0000] image connect using default endpoints: [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead.
ERRO[0000] validate service connection: validate CRI v1 image API for endpoint "unix:///var/run/dockershim.sock": rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing: dial unix /var/run/dockershim.sock: connect: no such file or directory"
IMAGE                                                             TAG                 IMAGE ID            SIZE
docker.io/flannel/flannel-cni-plugin                              v1.4.0-flannel1     421077d438927       4.34MB
docker.io/flannel/flannel                                         v0.24.4             3f482a2dfa7ce       31.6MB
docker.io/library/nginx                                           1.23-alpine         510900496a6c3       16.2MB
registry.aliyuncs.com/google_containers/coredns                   v1.10.1             97e04611ad434       14.6MB
registry.aliyuncs.com/google_containers/etcd                      3.5.12-0            014faa467e297       66.2MB
registry.aliyuncs.com/google_containers/kube-apiserver            v1.28.8             883d43b86efe0       31.8MB
registry.aliyuncs.com/google_containers/kube-controller-manager   v1.28.8             7beedd93d8e53       30.6MB
registry.aliyuncs.com/google_containers/kube-proxy                v1.28.8             837f825eec6c1       24.8MB
registry.aliyuncs.com/google_containers/kube-scheduler            v1.28.8             36dcd04414a4b       17MB
registry.aliyuncs.com/google_containers/pause                     3.9                 829e9de338bd5       268kB
```

提示错误，我们只需要创建一个 `/etc/crictl.yaml` 文件即可。

```yaml
runtime-endpoint: "unix:///run/containerd/containerd.sock"
image-endpoint: "unix:///run/containerd/containerd.sock"
timeout: 0
debug: false
pull-image-on-create: false
disable-pull-on-run: false
```

这里指定 endpoint 地址为 `unix:///run/containerd/containerd.sock`

```shell
$ crictl images
IMAGE                                                             TAG                 IMAGE ID            SIZE
docker.io/flannel/flannel-cni-plugin                              v1.4.0-flannel1     421077d438927       4.34MB
docker.io/flannel/flannel                                         v0.24.4             3f482a2dfa7ce       31.6MB
docker.io/library/nginx                                           1.23-alpine         510900496a6c3       16.2MB
registry.aliyuncs.com/google_containers/coredns                   v1.10.1             97e04611ad434       14.6MB
registry.aliyuncs.com/google_containers/etcd                      3.5.12-0            014faa467e297       66.2MB
registry.aliyuncs.com/google_containers/kube-apiserver            v1.28.8             883d43b86efe0       31.8MB
registry.aliyuncs.com/google_containers/kube-controller-manager   v1.28.8             7beedd93d8e53       30.6MB
registry.aliyuncs.com/google_containers/kube-proxy                v1.28.8             837f825eec6c1       24.8MB
registry.aliyuncs.com/google_containers/kube-scheduler            v1.28.8             36dcd04414a4b       17MB
registry.aliyuncs.com/google_containers/pause                     3.9                 829e9de338bd5       268kB
```



## 安装 Load Balancer 器

对于自建k8s集群的Load Balancer 这里选择 [Metallb](https://metallb.universe.tf/)。

MetalLB 主要有两个组件：

- **Controller**：实现地址分配，以 *Deployment* 方式运行，用于监听 Service 的变更，分配/回收 IP 地址。
- **Speaker**：实现地址对外广播，以 *DaemonSet* 方式运行，对外广播 Service 的 IP 地址。

配置模式分 [BGP](https://metallb.universe.tf/configuration/_advanced_bgp_configuration/) 和 [L2](https://metallb.universe.tf/configuration/_advanced_l2_configuration/) 两种，对于小集群选择 L2 模式即可, 安装文档 https://metallb.universe.tf/configuration/_advanced_l2_configuration/

### 安装 MetalLB

参考文档 https://metallb.universe.tf/installation/

```shell
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.4/config/manifests/metallb-native.yaml
```

### 启用ARP广播功能

由于从 `0.13.4`版本开始(见 [#2274](https://github.com/metallb/metallb/issues/2274#issuecomment-1928226470)、[#2285](https://github.com/metallb/metallb/issues/2285))，MetalLB不会从标记为 `exclude-from-external-load-balancers`的节点上进行公告，也就是说不会在存在此Label的节点上进行ARP广播，将导致其它机器无法找到 Load Balancer IP，要解决这个问题需要将 master 节点的这个 Label 删除。

```shell
kubectl label node ubuntu node.kubernetes.io/exclude-from-external-load-balancers-
```

> 这种情况一般是由于使用单节点的集群时，kubeadm 会给master节点添加一个 `exclude-from-external-load-balancers` 标签，导致无法

### 设置Load Balancer IP池

参考文档 https://metallb.universe.tf/configuration/_advanced_ipaddresspool_configuration/

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.100-192.168.1.250
```

创建一个名为 `first-pool` 的IP池，表示从指定IP段选择一个空闲的IP作为 `Load Balancer IP`。您也可以使用网络掩码格式写法，如 `192.168.1.0/24`。

 如果需要指定一个固定的IP地址话，可直接使用 `192.168.1.64/32`。

如果没有 IPAddressPool 选择器则被解释为该实例与所有可用的 IPAddressPools 相关联。

> 这里的IP段是我内网所属IP段，根据您的实际情况可能需要做调整。

### L2 模式配置

一旦一个IP被分配给一个service，就必须对其进行公告。上面说过有主要有两种方式，这里我们选择 L2模式，只选择一个节点来通告来自的IP。通常，Speaker 运行的所有节点都有资格获得给定的 IP。

```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
spec:
  ipAddressPools:
  - first-pool
```

这里选择了上面声明的IPPool。

如果想将 Speaker 安装在指定节点，也可以使用 `nodeSelector` 字段，官方文档有对此用法的介绍。

### 测试Load Balancer

至此，我们已经安装 Load Balancer 器完成，现在我们创建一个 服务试一下

```shell
kubectl create deployment nginx --image=nginx:1.23-alpine --port=80
kubectl expose deployment nginx --port=80 --target-port=80  --type=LoadBalancer
```

查看 service  的 EXTERNAL-IP

```shell
$ kubectl get svc nginx
NAME    TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
nginx   LoadBalancer   10.108.116.171   192.168.1.101   80:32165/TCP   25m
```

外网访问服务

```shell
$ curl 192.168.1.101
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

## 安装 Ingress 控制器

上面我们对一个service分配了一个LB IP，如果有多个服务的话，每个 service 都分配一个IP的话，实在太浪费IP了。

对于常用的 HTTP  服务，一般是多个域名对应同一个IP地址，这里就需要使用到  [Ingress](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/) 。但只有一个Ingress还不行，还得需要有一个想对应的 Ingress-controller 才可以。常见的控制器见 https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress-controllers/

这里我们以最流行的 [nginx Ingress controller](https://github.com/kubernetes/ingress-nginx/blob/main/README.md#readme) 为例。

### 安装 ingress-nginx

参考文档 https://kubernetes.github.io/ingress-nginx/deploy/

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml
```

安装完成后，会创建一个 Load Balancer 类型的 ingress-nginx-controller 服务

```shell
$ kubectl ge svc -n ingress-nginx
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.103.50.74     192.168.1.100   80:32628/TCP,443:31948/TCP   4m15s
ingress-nginx-controller-admission   ClusterIP      10.100.134.238   <none>          443/TCP                      4m15s
```

同时还将创建一个 IngressClass，后面创建 Ingress 资源是会用到这个 `NAME`

```shell
$ kubectl get ingressClass
NAME    CONTROLLER             PARAMETERS   AGE
nginx   k8s.io/ingress-nginx   <none>       13m
```

一旦 MetalLB 设置了 ingress-nginx LoadBalancer 服务的外部 IP 地址，就会在 iptables NAT 表中创建相应的条目，并且具有所选 IP 地址的节点开始响应 LoadBalancer 服务中配置的端口上的 HTTP 请求。

不过目前我们还无法访问任何服务，到目前我们只是创建了一个流量入口而已，还需要创建一个 Ingress 来配置多个域名与它的关系。

> 安装 ingress-nginx时，有些镜像需要从 registry.k8s.io 下载，请保证网络正常。
>
> 对于 ingress-nginx-controller 是一个以pod形式运行的 nginx 服务，扮演着webserver 的角色，用法并未有什么改变。主要是通过修改 nginx.conf 文件来实现流量入口的，感兴趣的话，可以到这个容器里观察一下 nginx.conf  的配置内容。

### 创建pod和service

首先创建一些pod和service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.23-alpine
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: myservicea
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apache-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: apache
  template:
    metadata:
      labels:
        app: apache
    spec:
      containers:
      - name: apache
        image: httpd:alpine3.19
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: myserviceb
spec:
  selector:
    app: apache
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

```

这里创建了两个服务，myservicea 对应的是nginx服务，myserviceb 对应的是apache服务。

查看服务

```
$ kubectl get svc myservicea myserviceb
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
myservicea   ClusterIP   10.99.79.129    <none>        80/TCP    4m40s
myserviceb   ClusterIP   10.109.134.92   <none>        80/TCP    93s

$ kubectl get ep myservicea myserviceb
NAME         ENDPOINTS                                      AGE
myservicea   10.244.0.19:80,10.244.0.24:80,10.244.0.25:80   5m5s
myserviceb   10.244.0.26:80,10.244.0.27:80                  118s
```

> 这里 myservicea 有3个 endpoint, 主要是我们当前集群中已有一个nginx pod。

### 配置 Ingress 

通过 ingress 来指定 域名和后端 service  关系，参考文档 https://kubernetes.github.io/ingress-nginx/user-guide/basic-usage/

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-myservicea
spec:
  rules:
  - host: myservicea.foo.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myservicea
            port:
              number: 80
  ingressClassName: nginx
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-myserviceb
spec:
  rules:
  - host: myserviceb.foo.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myserviceb
            port:
              number: 80
  ingressClassName: nginx
```

这里通过在ingress里定义 `ingressClassName` 的字段指定当前ingress使用的 ingress controller。如果你不为 Ingress 指定 IngressClass，并且你的集群中只有一个 IngressClass 被标记为默认，那么 Kubernetes 会将此集群的默认 IngressClass [应用](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/#default-ingress-class)到 Ingress 上。 你可以通过将 [`ingressclass.kubernetes.io/is-default-class` 注解](https://kubernetes.io/zh-cn/docs/reference/labels-annotations-taints/#ingressclass-kubernetes-io-is-default-class) 的值设置为 `"true"` 来将一个 IngressClass 标记为集群默认。



 确认结果

```shell
$ kubectl get ingress
NAME                 CLASS   HOSTS                ADDRESS   PORTS   AGE
ingress-myservicea   nginx   myservicea.foo.org   192.168.1.100   80      115s
ingress-myserviceb   nginx   myserviceb.foo.org   192.168.1.100   80      115s

$ kubectl get svc -n ingress-nginx
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.103.50.74     192.168.1.100   80:32628/TCP,443:31948/TCP   40m
ingress-nginx-controller-admission   ClusterIP      10.100.134.238   <none>          443/TCP                      40m
```

可以看到两个域名绑定的IP是 ingress-nginx 服务的IP地址。

### 访问域名

用curl访问域名 `myservicea.foo.org`，对应的是 myservicea service 是 `nginx` 服务。

```shell
$ curl -D- http://192.168.1.100 -H 'Host: myservicea.foo.org'
HTTP/1.1 200 OK
Date: Wed, 03 Apr 2024 03:36:49 GMT
Content-Type: text/html
Content-Length: 615
Connection: keep-alive
Last-Modified: Tue, 28 Mar 2023 17:09:24 GMT
ETag: "64231f44-267"
Accept-Ranges: bytes

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

服务正常。

再访问另一个myserviceb service 对应的是apache服务，域名 `myserviceb.foo.org`

```shell
$ curl -D- http://192.168.1.100 -H 'Host: myserviceb.foo.org'
HTTP/1.1 200 OK
Date: Wed, 03 Apr 2024 03:35:58 GMT
Content-Type: text/html
Content-Length: 45
Connection: keep-alive
Last-Modified: Mon, 11 Jun 2007 18:53:14 GMT
ETag: "2d-432a5e4a73a80"
Accept-Ranges: bytes

<html><body><h1>It works!</h1></body></html>
```

> 命令中的"-D-"参数会让curl把服务器的响应头信息打印到标准输出。

对于 ingress-nginx 更多用法，请参考 https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/

如果在其它机器无法访问域名，请参考常见问题。

## 常见问题

1. 某些设备（如Raspberry Pi）在使用WiFi时不会响应ARP请求（Metallb 的 Speaker组件不进行广播）。这可能会导致服务最初是可访问的，但不久之后就会中断。在这个阶段，尝试arping将导致超时，并且无法访问服务。

   参考文档 https://metallb.universe.tf/troubleshooting/

   一种解决方法是在接口上启用混杂模式：`sudo ifconfig＜device＞promisc`。例如：

   ```shell
   sudo ifconfig wlan0 promisc
   ```

   确认

   ```shell
   $ ip link show wlan0
   3: wlan0: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DORMANT group default qlen 1000
       link/ether e4:5f:01:56:9a:cb brd ff:ff:ff:ff:ff:ff
   ```

   存在 `PROMISC` 表示启用此网卡启用了混杂格式。

   另外也可以手动进行广播

   ```shell
   arping -I wlan0 <Load Balancer IP>
   ```

​	遗憾的是，在我的RaspPI环境里，上面给出的方法仍没有进行arp广播的，目前不确定是什么原因。暂时用下面的命令解决掉了

```
$ arping -c 3 -S 192.168.1.200 192.168.1.108
ARPING 192.168.1.108
42 bytes from a4:5e:60:bb:0c:19 (192.168.1.108): index=0 time=103.134 msec
42 bytes from a4:5e:60:bb:0c:19 (192.168.1.108): index=1 time=22.920 msec
42 bytes from a4:5e:60:bb:0c:19 (192.168.1.108): index=2 time=47.132 msec

--- 192.168.1.108 statistics ---
3 packets transmitted, 3 packets received,   0% unanswered (0 extra)
rtt min/avg/max/std-dev = 22.920/57.728/103.134/33.593 ms
```

表示将 LB IP 广播到指定机器 192.168.1.108。

此时在 192.168.1.108 这台机器查看 arp 记录

```shell
$ arp -a | grep 192.168.1.200
? (192.168.1.200) at e4:5f:1:56:9a:cb on en0 ifscope [ethernet]
```

此时再访问域名即可。

>  对于arp广播记录都会存在一个有效期，到期后arp记录将自动删除。如果手动进行广播的话，过一段时间服务可能会出现无法访问的情况。

## 总结

本文主要介绍内容

1. 安装 kubernetes集群，并使用 containerd 运行时
2. 为集群安装 Load Balancer 
3. 安装 Ingress 控制器，通过 Ingress 来通过域名访问集群内的 service

## 参考文档

- https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/
- https://docs.docker.com/engine/install/ubuntu/
- https://github.com/containerd/containerd
- https://developer.aliyun.com/mirror/kubernetes
- https://metallb.universe.tf/
- https://kubernetes.github.io/ingress-nginx/
- https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/
- https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress-controllers/
