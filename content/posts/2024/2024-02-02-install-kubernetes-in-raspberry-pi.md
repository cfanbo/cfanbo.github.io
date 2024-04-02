---
title: Raspberry Pi 安装Kubernetes
date: 2024-04-02T21:53:47+08:00
type: post
toc: true
url: /posts/install-kubernetes-in-raspberry-pi
categories:
  - 程序开发
tags:
  - k8s
---

这里是 arm64 架构，**树莓派 4B**, 四核八 G 内存 配置，系统为 Ubuntu 22.04.1 LTS

```shell
$ uname -a
Linux ubuntu 5.15.0-1049-raspi #52-Ubuntu SMP PREEMPT Thu Mar 14 08:39:42 UTC 2024 aarch64 aarch64 aarch64 GNU/Linux

$ cat /etc/issue
Ubuntu 22.04.1 LTS \n \l
```

## 环境检查

由于 k8s 会使用 8080 和 6443 这两个端口，因此要保证端口可用，其实禁用 swap。

```
sudo swapoff -a
```

对安装环境初始化，参考 https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/

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
