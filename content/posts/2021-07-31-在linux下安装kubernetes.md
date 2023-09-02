---
title: 在linux下安装Kubernetes
author: admin
type: post
date: 2021-07-31T01:51:16+00:00
url: /archives/30924
categories:
 - 系统架构
tags:
 - k8s

---
环境 ubuntu18.04 64位

Kubernetes v1.21.3

这里需要注意，本教程安装的k8s版本号 `<- v1.24.0`，主要是因为从v1.24.0以后移除了 `Dockershim`，无法继续使用 `Docker Engine`，后续将默认采用 [containerd][1] ，它是一个从 CNCF 毕业的项目。如果仍想使用原来 `Docker Engine` 的方式可以安装  [cri-dockerd][2] ，它是 Dockershim 的替代品。

如果你想将现在 Docker Engine 的容器更换为 `containerd`，可以参考官方迁移教程 [将节点上的容器运行时从 Docker Engine 改为 containerd](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/migrating-from-dockershim/change-runtime-containerd/)

为了解决国内访问一些国外网站慢的问题，本文使用了国内阿里云的镜像。

# 更换apt包源 {#%E6%9B%B4%E6%8D%A2apt%E5%8C%85%E6%BA%90.wp-block-heading}

这里使用aliyun镜像 , 为了安全起见，建议备份原来系统默认的 /etc/apt/sources.list 文件

编辑文件 /etc/apt/sources.list，将默认网址  或  替换为

更新缓存

```
$ sudo apt-get clean all
$ sudo apt-get update

```

# 安装Docker {#%E5%AE%89%E8%A3%85docker.wp-block-heading}

参考官方文档  或 aliyun 文档

### 安装 docker {#%E8%AE%BE%E7%BD%AEdocker%E4%BB%93%E5%BA%93.wp-block-heading}

 1. 安装基础工具

```
$ sudo apt update
$ sudo apt-get install
    apt-transport-https
    ca-certificates
    curl
    gnupg
    lsb-release

```

2. 安装GPG证书

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

```

3: 写入软件源信息

```
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```

4. 安装 DOCKER ENGINE

```
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io

```

验证docker是否安装成功

```
$ sudo docker info

```

查看 docker 版本信息

```
$ sudo docker version
Client: Docker Engine - Community
 Version:           20.10.7
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        f0df350
 Built:             Wed Jun  2 11:56:40 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.7
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       b0f5bc3
  Built:            Wed Jun  2 11:54:48 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.8
  GitCommit:        7eba5930496d9bbe375fdf71603e610ad737d2b2
 runc:
  Version:          1.0.0
  GitCommit:        v1.0.0-0-g84113ee
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

```

如果看到以上信息则表示安装成功

### 设置 docker 源镜像 {#%E6%B7%BB%E5%8A%A0docker%E6%BA%90%E9%95%9C%E5%83%8F.wp-block-heading}

由于默认的docker源镜像是国外，国内用户访问太慢，所以我们这里使用aliyun提供的docker镜像源.

由于官方推荐使用 docker cgroup drive 为 **systemd**，同时对其进行修改。

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://gfxrbz51.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

```

查看配置是否生效

```
$ sudo docker info

```

# 安装 k8s {#%E5%AE%89%E8%A3%85-k8s.wp-block-heading}

### 关闭交换分区 {#2-%E5%85%B3%E9%97%AD%E4%BA%A4%E6%8D%A2%E5%88%86%E5%8C%BA.wp-block-heading}

```
$ sudo swapoff -a

```

建议永久关闭, 在 /etc/fstab 中注释掉 swapfile 那一行

```
#/swapfile                                 none            swap    sw              0       0

```

### 安装工具 kubectl、 kubelet 和 kubeadm {#1-%E5%AE%89%E8%A3%85%E5%B7%A5%E5%85%B7-kubectl-kubelet-%E5%92%8C-kubeadm.wp-block-heading}

这里使用aliyun 镜像，参考

要查看最新 Kubernetes 版本号可通过网址 [https://storage.googleapis.com/kubernetes-release/release/stable.txt](https://storage.googleapis.com/kubernetes-release/release/stable.txt) 查看, 截止目前最新版本为 v1.21.3。

我们这里安装的是最新版本 v1.21.3 版本，相应的这三个软件 kubelet、kubectl 和 kubeadmin 版本号要保持统一，否则容易出问题且不好排查，这里我们全部都使用这个版本号包括 kubernetes 在内。

验证安装结果

```
$ sudo kubectl version
Client Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.3", GitCommit:"ca643a4d1f7bfe34773c74f79527be4afd95bf39", GitTreeState:"clean", BuildDate:"2021-07-15T21:04:39Z", GoVersion:"go1.16.6", Compiler:"gc", Platform:"linux/amd64"}

$ sudo kubelet --version
Kubernetes v1.21.3

$ sudo kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.3", GitCommit:"ca643a4d1f7bfe34773c74f79527be4afd95bf39", GitTreeState:"clean", BuildDate:"2021-07-15T21:03:28Z", GoVersion:"go1.16.6", Compiler:"gc", Platform:"linux/amd64"}

```

也可以参考官方教程

如果想安装指定版本，可以使用

```
$ apt install kubeadm=1.21.3-00
```

版本号后面的 `-00` 不能省略。

注意情况下 kubeadm 会依赖于 kubelet和 kubectl 这两个命令，因此如果没有指定安装它们两个的话，则默认会自动安装 ”**最新**“ 版本号的 kubelet 和 kubectl 。因此如果安装了指定版本号的话，则需要将三者都指定一样的版本号，否则会存在兼容问题。

也可以查看当前软件所有的版本列表

```
$ apt-cache madison kubelet
```

### 使用 kubeadm 创建k8s集群 {#3-%E4%BD%BF%E7%94%A8-kubeadm-%E5%88%9B%E5%BB%BAk8s%E9%9B%86%E7%BE%A4.wp-block-heading}

参考官方教程

master 主节点初始化

```
kubeadm init
    --apiserver-advertise-address=192.168.0.52
    --image-repository registry.aliyuncs.com/google_containers
    --kubernetes-version v1.21.3
    --pod-network-cidr=10.244.0.0/16

```

初始化命令说明：

```
--apiserver-advertise-address

```

指明用 Master 的哪个 interface 与 Cluster 的其他节点通信。如果 Master 有多个 interface，建议明确指定，如果不指定，kubeadm 会自动选择有默认网关的 interface。

```
--pod-network-cidr

```

指定 Pod 网络的范围。Kubernetes 支持多种网络方案，而且不同网络方案对 –pod-network-cidr 有自己的要求，这里设置为 10.244.0.0/16 是因为我们将使用 flannel 网络方案，必须设置成这个 CIDR。

```
--image-repository

```

Kubenetes默认repository地址是 [k8s.gcr.io][3]，在国内并不能访问 [gcr.io][4]，在1.13版本中我们可以增加–image-repository参数，默认值是 [k8s.gcr.io][3]，将其指定为阿里云镜像地址：[registry.aliyuncs.com/google_containers][5] 。

```
--kubernetes-version=v1.21.3

```

我们这里指定了要安装的版本号 v1.21.3，如果不指定版本号的话，则默认会安装与当前 kubectladm 相同的版本号。

```
sxf@sxf-virtual-machine:~$ sudo kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.21.3  --pod-network-cidr=10.244.0.0/16

[init] Using Kubernetes version: v1.21.3
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local sxf-virtual-machine] and IPs [10.96.0.1 192.168.3.52]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost sxf-virtual-machine] and IPs [192.168.3.52 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost sxf-virtual-machine] and IPs [192.168.3.52 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 27.506486 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.21" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node sxf-virtual-machine as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node sxf-virtual-machine as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: lw58fm.04ywrp7f8m4q39j6
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.3.52:6443 --token lw58fm.04ywrp7f8m4q39j6
        --discovery-token-ca-cert-hash sha256:4dd451d1e3e7e0743bfaaf3c01d9ab0d524e1d71521428bccfcd9406ae9860da

```

> 如果遇到 “ [registry.aliyuncs.com/google_containers/coredns:v1.8.0](http://registry.aliyuncs.com/google_containers/coredns:v1.8.0%E2%80%9D)” 无法下载错误，请参考下方的“常见问题”

初始化过程说明：

[preflight] kubeadm 执行初始化前的检查。
[certs] 生成相关的各种token和证书
[kubeconfig] 生成 KubeConfig 文件，kubelet 需要这个文件与 Master 通信
[kubelet-start] 生成kubelet的配置文件 /var/lib/kubelet/config.yaml
[control-plane] 安装 Master 控制面组件(apiserver、controller-manager和 scheduler)，此时会从指定的 Registry 下载组件的 Docker 镜像。每个组件占用一个静态pod，注意区别静态pod和动态pod
[mark-control-plane] 通过添加labels标签，设置当前node节点为控制面，并设置当前节点为污节点，不允许部署pod调度
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory “/etc/kubernetes/manifests”. This can take up to 4m0s
[apiclient] 检查控制面组件是否健康
[bootstrap-token] 生成token记录下来，后边使用kubeadm join往集群中添加节点时会用到
[kubelet-finalize] 指定kubelet客户端证书和密钥
[addons] 安装附加组件 kube-proxy 和 kube-dns。

了解初始化步骤，对理解k8s非常的重要，因此推荐每个开发者都看一下每行输出信息。

为当前系统用户创建管理k8s的所需的一些配置文件

```
mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

现在我们只是创建了 master 节点，默认当前master节点是不可以部署pod的，只有node节点才可以调度pod。

K8s在进行POD调度的时候，是不允许调度到有 污点 的节点的，默认情况下 master 节点是存在污点的，即不允许在 master 节点进行调度POD

```
$ kubectl describe node master

...
Taints:             node-role.kubernetes.io/master:NoSchedule
                    node.kubernetes.io/not-ready:NoSchedule
...

```

对于我们个人学习来说，就一台电脑，只需要一个单机k8s集群即可。所以这里我们需要将master节点的污点删除，方便我们学习使用。

```
$ sudo kubectl taint nodes --all node-role.kubernetes.io/master-

```

此时将变为

```
Taints:             <none>

```

可以看到污点值变为了 `` 即表示删除了污点。

如果需要将其它 worker node加入集群,则需要使用 root 用户执行以下命令加入集群。

```
$ su root
# kubeadm join 192.168.3.52:6443 --token lw58fm.04ywrp7f8m4q39j6
        --discovery-token-ca-cert-hash sha256:4dd451d1e3e7e0743bfaaf3c01d9ab0d524e1d71521428bccfcd9406ae9860da

```

如果忘记token的话，可以查看

```
$ sudo kubeadm token list

```

token 默认都有一个有效期，如果过期的话，可以通过命令

```
$ sudo kubeadm token create --print-join-command

```

创建一个新的token，在新的node执行即可加入集群。参考 [https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-token/](https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-token/)

查看集群节点状态

```
$ sudo kubectl get node -o wide

NAME                  STATUS     ROLES                  AGE   VERSION
sxf-virtual-machine   NotReady   control-plane,master   67m   v1.21.3

```

发现 STATUS 状态为 NotReady ，则表示集群有问题，通过通过查看节点详情了解详细原因

```
$ sudo kubectl describe node
...
  RenewTime:       Thu, 29 Jul 2021 15:39:21 +0800
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Thu, 29 Jul 2021 15:37:33 +0800   Thu, 29 Jul 2021 14:31:09 +0800   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Thu, 29 Jul 2021 15:37:33 +0800   Thu, 29 Jul 2021 14:31:09 +0800   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Thu, 29 Jul 2021 15:37:33 +0800   Thu, 29 Jul 2021 14:31:09 +0800   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            False   Thu, 29 Jul 2021 15:37:33 +0800   Thu, 29 Jul 2021 14:31:09 +0800   KubeletNotReady              container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized
  ...

```

主要错误是由于没有安装网络插件引起的，安装一个就可以了。如果你查看pod状态的话，会发现 coredns 的状态一直处于 Penging 状态，也是同样的原因的

```
$ kubectl get pods -A
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-59d64cd4d4-5wp7k         0/1     Pending   0          86s
kube-system   coredns-59d64cd4d4-c49p6         0/1     Pending   0          86s
kube-system   etcd-ubuntu                      1/1     Running   0          93s
kube-system   kube-apiserver-ubuntu            1/1     Running   0          93s
kube-system   kube-controller-manager-ubuntu   1/1     Running   0          93s
kube-system   kube-proxy-8mpxg                 1/1     Running   0          86s
kube-system   kube-scheduler-ubuntu            1/1     Running   0          93s

```

同时这里有两个污点，一个是 `node-role.kubernetes.io/master`, 另一个是 \`node.kubernetes.io/not-ready\` 。

### 安装 pod 网络插件 {#%E5%AE%89%E8%A3%85-pod-%E7%BD%91%E7%BB%9C%E6%8F%92%E4%BB%B6.wp-block-heading}

你必须部署一个基于 Pod 网络插件的 容器网络接口 (CNI)，以便你的 Pod 可以相互通信。 在安装网络之前，集群 DNS (CoreDNS) 将不会启动。

常见的网络插件有 [Flannel][6]、[Calico][7]、[Weave][8]， 它们之间的区别请参考相关文章。
我们这里选择 Flannel 插件, 它也是最简单。

此插件需要在 `kubeadm init` 时设置 `--pod-network-cidr=10.244.0.0/16`

```
$ sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

```

这时会自动下载镜像文件 `quay.io/coreos/flannel:v0.14.0` 到本机（版本号可能会发生变化），如果超时的话，可能需要手动下载到本机，再执行当前命令。待安装成功后，会看到有两个 `flannel` 容器，且刚才处于 `Pending` 状态的 `coredns` 变为 `Running`，表示网络通讯正常。

同时将在本地创建一个 cni0 的网桥，其分配的ip 地址为 `10.244.0.1/24`。同时创建一个 `/run/flannel/subnet.env` 文件，内容为

```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.0.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
```

它是根据上面 `kubeadm init --pod-network-cidr=10.244.0.0/16` 指定参数配置而生成的。

对于 `cni0` 网桥如果删除的话将自动读取,将自动重建，所有通过k8s创建的pod都会创建一对 veth pair，其中一端将自动连接到cni0网桥，可通过命令查看。

```
# brctl show
bridge name	bridge id		STP enabled	interfaces
cni0		8000.0e6a4cbd3992	no		veth6f1dceae
							veth83ba60db

docker0		8000.02423ddafda3	no
```

这里的 docker0 网桥是docker自己的，只有手动创建容器时才会容器网络才会接入到这个网桥。

如果上面的文件无法下载，则需要手动在浏览器里打开复制内容，并将其内容复制到新创建的文件里，再执行 `kubectl apply -f filename` 命令，主要是 github.com 目前对此种方法进行了限制。

这时会发现多了一个 `kube-flannel-ds-xxx` 的pod。如果你执行 `ip link` 命令的话，还会看到创建出一个 `flannel.1` 的网卡。

```
# kubectl get pod -A
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-59d64cd4d4-5wp7k         1/1     Running   0          10m
kube-system   coredns-59d64cd4d4-c49p6         1/1     Running   0          10m
kube-system   etcd-ubuntu                      1/1     Running   0          10m
kube-system   kube-apiserver-ubuntu            1/1     Running   0          10m
kube-system   kube-controller-manager-ubuntu   1/1     Running   0          10m
kube-system   kube-flannel-ds-b4gzk            1/1     Running   0          5m53s
kube-system   kube-proxy-8mpxg                 1/1     Running   0          10m
kube-system   kube-scheduler-ubuntu            1/1     Running   0          10m

```

# 常见问题 {#%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98.wp-block-heading}

 1. 安装过程中会提示 “[registry.aliyuncs.com/google_containers/coredns:v1.8.0”][9] 这个镜像无法下载，解决办法：

```
$ sudo docker pull coredns/coredns:1.8.0
$ sudo docker tag coredns/coredns:1.8.0 registry.aliyuncs.com/google_containers/coredns:v1.8.0

```

再次执行初始化命令即可

1. 安装集群失败，请检查是否工具版本不匹配原因，见 [https://kubernetes.io/zh/docs/setup/release/version-skew-policy/#supported-versions](https://kubernetes.io/zh/docs/setup/release/version-skew-policy/#supported-versions)
2. 查看 kubectl 日志


```
$ journalctl -f -u kubelet

```

1. 查看 POD 调度日志


```
$ sudo kubectl logs kube-flannel-ds-gfhf2 -n kube-system

```

1. 集群初始化如果遇到问题，可以使用 `kubeadm reset` 命令进行清理然后重新执行初始化。

2. flannel pod 一直处于 `CrashLoopBackOff` 状态，可通过修改文件


```
$ vim /etc/kubernetes/manifests/kube-controller-manager.yaml

```

在 `spec.containers.command` 命令后面添加以两个参数

```
--allocate-node-cidrs=true
--cluster-cidr=10.244.0.0/16

```

1. 如果在安装网络插件后，显示 coredns 的POD 处于 STATUS:RUNNING READY:0/1, 可参考方法 [https://www.programmersought.com/article/14857690636/](https://www.programmersought.com/article/14857690636/) 或 [https://stackoverflow.com/questions/60782064/coredns-has-problems-getting-endpoints-services-namespaces](https://stackoverflow.com/questions/60782064/coredns-has-problems-getting-endpoints-services-namespaces)

```
$ iptables -P INPUT ACCEPT
$ kubectl rollout restart deployment coredns --namespace kube-system

```

重启 kubectl

```
$ sudo systemctl restart kubelet

```

主要原因是我们在 `kubeadm init` 的时候没有指定 `CIDR` 网络。

8. 在安装 Flannel时遇到 Error Registering network: operation not supported 解决办法

由于 `ubuntu 21.10` 以后操作系统移除了对 `vxlan` 的支持，因此可能会遇到这个错误，只时需要手动安装一下即可。这个错误我在 `Raspberry pi 4`  中遇到过，见 [issue1747](https://github.com/flannel-io/flannel/issues/1747)

```
sudo apt install -y linux-modules-extra-raspi
```

9. 从 k8s v24.0 版本以后，默认删除了 Dockershim（ [shim 介绍](https://kubernetes.io/zh-cn/blog/2022/05/03/dockershim-historical-context/)） 移除了对 docker 运行时的支持，一般采用  [containerd][1]  运行时，这已是一个从 CNCF 毕业的项目。如果仍想使用 docker 运行时，则需要安装一个 [cri-dockerd](https://github.com/Mirantis/cri-dockerd)， 它是 Dockershim 的代替品，官方对此支持变更介绍 [https://kubernetes.io/zh-cn/blog/2022/03/31/ready-for-dockershim-removal/](https://kubernetes.io/zh-cn/blog/2022/03/31/ready-for-dockershim-removal/)。幸运的是，Kubernetes 项目已经以 containerd 为例， 提供了[更改节点容器运行时][10]的过程文档。仍有疑问，请先查看[弃用 Dockershim 的常见问题][11]。

# 参考资料 {#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99.wp-block-heading}

 *
 *
 *
 *
 *
 *

 [1]: https://containerd.io/
 [2]: https://github.com/Mirantis/cri-dockerd
 [3]: http://k8s.gcr.io
 [4]: http://gcr.io
 [5]: http://registry.aliyuncs.com/google_containers
 [6]: https://github.com/coreos/flannel
 [7]: https://github.com/projectcalico/cni-plugin
 [8]: https://www.weave.works/oss/net/
 [9]: http://registry.aliyuncs.com/google_containers/coredns:v1.8.0%E2%80%9D
 [10]: https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/migrating-from-dockershim/change-runtime-containerd/
 [11]: https://kubernetes.io/zh-cn/blog/2022/02/17/dockershim-faq/