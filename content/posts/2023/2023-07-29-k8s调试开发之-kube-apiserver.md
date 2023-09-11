---
title: k8s调试之 kube-apiserver 组件
author: admin
type: post
date: 2023-07-28T17:53:19+00:00
url: /archives/34454
categories:
 - 程序开发
tags:
 - apiserver
 - goland
 - k8s

---
上一节[《GoLand+dlv进行远程调试》][1]我们介绍了如何使用 `GoLand` 进行远程调试，本节我们就以 `kube-apiserver` 为例演示一下调试方法。

# 服务器环境 

作为开发调试服务器，需要安装以下环境

 1. 安装 `Golang` 环境，国内最好设置 `GOPROXY`
 2. 安装 `dlv` 调试工具
 3. 安装 `Docker` 环境， 同时安装 `containerd` 服务（对应官方教程中的 `containerd.io` 安装包）并设置代理

# 同步代码(本地) 

以下为我们本机环境设置。

本机下载 [kubernetes](https://github.com/kubernetes/kubernetes) 仓库

```shell
git clone --filter=blob:none https://github.com/kubernetes/kubernetes.git
```

> 这里指定 –filter=bold:none 可以实现最小化下载

这里 k8s 项目目录为 `/Users/sxf/workspace/kubernetes`, 对应远程服务器目录为 `/home/sxf/workspace/kubernetes`，如图所示![](https://blogstatic.haohtml.com/uploads/2023/07/d2b5ca33bd970f64a6301fa75ae2eb22-6.png)

映射关系配置![](https://blogstatic.haohtml.com/uploads/2023/07/d2b5ca33bd970f64a6301fa75ae2eb22-7.png)

同时选择自动上传 `Automatic upload (Always)` 菜单，这样以后当本地文件有变更时将自动同步到远程服务器。

首次手动同步远程代码(右键`Upload here`菜单)![](https://blogstatic.haohtml.com/uploads/2023/07/d2b5ca33bd970f64a6301fa75ae2eb22-8.png)

> 如果文件特别多的话，首次同步将会比较慢，可以手动将本地项目打包上传到远程服务器再解压。

# 环境检测 

首先对当前调试环境进行一系列的检查，如果条件不满足将给出提示信息

```shell
$ make verify
```

如果本地 Git 仓库存在未提交的文件的话，则此时将提示先提交。这一块有点不好，我这里直接终止了这个检查继续下一步。

# 安装Etcd 

由于 `kube-apiserver` 依赖于 `ectd` ，所以必须先安装etcd。

```shell
$ cd workspace/kubernetes
$ ./hack/install-etcd.sh
Downloading https://github.com/coreos/etcd/releases/download/v3.5.7/etcd-v3.5.7-linux-amd64.tar.gz succeed
etcd v3.5.7 installed. To use:
export PATH="/home/sxf/workspace/kubernetes/third_party/etcd:${PATH}"
```

此时将自动从远程下载etcd二进制文件到目录 `/third_party`。

接着按照提示设置环境变量

```shell
$ export PATH="/home/sxf/workspace/kubernetes/third_party/etcd:${PATH}"
```

此时我们查看一下目录内容

```shell
$ ls -al third_party/
total 36
drwxr-xr-x  7 sxf sxf 4096 Jul 28 05:20 .
drwxr-xr-x 22 sxf sxf 4096 Jul 28 07:38 ..
-rw-r--r--  1 sxf sxf  259 Jun  3 07:59 OWNERS
lrwxrwxrwx  1 sxf sxf   23 Jul 28 05:20 etcd -> etcd-v3.5.7-linux-amd64
drwxr-xr-x  3 sxf sxf 4096 Jan 20  2023 etcd-v3.5.7-linux-amd64
-rw-r--r--  1 sxf sxf   24 Jun  3 07:59 etcd.BUILD
drwxr-xr-x  6 sxf sxf 4096 Jul 28 02:32 forked
drwxr-xr-x  2 sxf sxf 4096 Jul 28 02:32 gimme
drwxr-xr-x  3 sxf sxf 4096 Jul 28 02:32 multiarch
drwxr-xr-x  3 sxf sxf 4096 Apr 20 12:16 protobuf
```

可以看到一个 etcd 文件（是个软连接）

# 启动集群 

首先通过设置环境变量`DBG=1` 禁止编译二进制时做任何优化和内联（必须禁止）。

同时通过设置环境变量 `ENABLE_DAEMON=true` 表示集群启动成功后（执行二进制文件）以守护进程的方式运行，否则过一段时间将自动退出服务。

## 安装依赖 

安装三方依赖库

```shell
$ go get ./...
```

安装 `cfssl` 和 `cfssljson`

```shell
$ go install github.com/cloudflare/cfssl/cmd/...@latest
```

## 启动k8s集群 

```shell
$ ENABLE_DAEMON=true DBG=1 ./hack/local-up-cluster.sh
make: Entering directory '/home/sxf/workspace/kubernetes'
go version go1.20.5 linux/amd64
+++ [0728 16:53:35] Building go targets for linux/amd64
    k8s.io/kubernetes/cmd/kubectl (static)
    k8s.io/kubernetes/cmd/kube-apiserver (static)
    k8s.io/kubernetes/cmd/kube-controller-manager (static)
    k8s.io/kubernetes/cmd/cloud-controller-manager (non-static)
    k8s.io/kubernetes/cmd/kubelet (non-static)
    k8s.io/kubernetes/cmd/kube-proxy (static)
    k8s.io/kubernetes/cmd/kube-scheduler (static)
make: Leaving directory '/home/sxf/workspace/kubernetes'
API SERVER secure port is free, proceeding...
Detected host and ready to start services.  Doing some housekeeping first...
Using GO_OUT /home/sxf/workspace/kubernetes/_output/local/bin/linux/amd64
Starting services now!
Starting etcd
etcd --advertise-client-urls http://127.0.0.1:2379 --data-dir /tmp/tmp.df4hoqT1KJ --listen-client-urls http://127.0.0.1:2379 --log-level=warn 2> "/tmp/etcd.log" >/dev/null
Waiting for etcd to come up.
+++ [0728 16:55:23] On try 2, etcd: : {"health":"true","reason":""}

{"header":{"cluster_id":"14841639068965178418","member_id":"10276657743932975437","revision":"2","raft_term":"2"}}Generating a RSA private key
..........+++++
...............+++++
writing new private key to '/var/run/kubernetes/server-ca.key'
-----
Generating a RSA private key
...........+++++
..........................................................................+++++
writing new private key to '/var/run/kubernetes/client-ca.key'
-----
Generating a RSA private key
........................................+++++
.........................................................................+++++
writing new private key to '/var/run/kubernetes/request-header-ca.key'
-----
2023/07/28 16:55:25 [INFO] generate received request
2023/07/28 16:55:25 [INFO] received CSR
2023/07/28 16:55:25 [INFO] generating key: rsa-2048
2023/07/28 16:55:25 [INFO] encoded CSR
2023/07/28 16:55:25 [INFO] signed certificate with serial number 591476403309736501145132082157932131672501196215
2023/07/28 16:55:25 [INFO] generate received request
2023/07/28 16:55:25 [INFO] received CSR
2023/07/28 16:55:25 [INFO] generating key: rsa-2048
2023/07/28 16:55:25 [INFO] encoded CSR
2023/07/28 16:55:25 [INFO] signed certificate with serial number 312094899001296746655962556813904233515408582408
2023/07/28 16:55:25 [INFO] generate received request
2023/07/28 16:55:25 [INFO] received CSR
2023/07/28 16:55:25 [INFO] generating key: rsa-2048
2023/07/28 16:55:26 [INFO] encoded CSR
2023/07/28 16:55:26 [INFO] signed certificate with serial number 174773804806558810260456087825925370815840847427
2023/07/28 16:55:26 [INFO] generate received request
2023/07/28 16:55:26 [INFO] received CSR
2023/07/28 16:55:26 [INFO] generating key: rsa-2048
2023/07/28 16:55:26 [INFO] encoded CSR
2023/07/28 16:55:26 [INFO] signed certificate with serial number 406555199599251281487351419198465816155410838715
2023/07/28 16:55:26 [INFO] generate received request
2023/07/28 16:55:26 [INFO] received CSR
2023/07/28 16:55:26 [INFO] generating key: rsa-2048
2023/07/28 16:55:26 [INFO] encoded CSR
2023/07/28 16:55:26 [INFO] signed certificate with serial number 306422220153984589648426173951052538350603672102
2023/07/28 16:55:26 [INFO] generate received request
2023/07/28 16:55:26 [INFO] received CSR
2023/07/28 16:55:26 [INFO] generating key: rsa-2048
2023/07/28 16:55:27 [INFO] encoded CSR
2023/07/28 16:55:27 [INFO] signed certificate with serial number 261195493413132532478940896454614280039686591470
2023/07/28 16:55:27 [INFO] generate received request
2023/07/28 16:55:27 [INFO] received CSR
2023/07/28 16:55:27 [INFO] generating key: rsa-2048
2023/07/28 16:55:27 [INFO] encoded CSR
2023/07/28 16:55:27 [INFO] signed certificate with serial number 170518182962877110931343935635851469708422931922
2023/07/28 16:55:27 [INFO] generate received request
2023/07/28 16:55:27 [INFO] received CSR
2023/07/28 16:55:27 [INFO] generating key: rsa-2048
2023/07/28 16:55:27 [INFO] encoded CSR
2023/07/28 16:55:27 [INFO] signed certificate with serial number 631171054960913534211538909232599649610356823693
Waiting for apiserver to come up
+++ [0728 16:55:34] On try 6, apiserver: : ok

clusterrolebinding.rbac.authorization.k8s.io/kube-apiserver-kubelet-admin created
clusterrolebinding.rbac.authorization.k8s.io/kubelet-csr created
Cluster "local-up-cluster" set.
use 'kubectl --kubeconfig=/var/run/kubernetes/admin-kube-aggregator.kubeconfig' to use the aggregated API server
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
coredns addon successfully deployed.
Checking CNI Installation at /opt/cni/bin
CNI Installation not found at /opt/cni/bin
Installing CNI plugin binaries ...
5238fbb2767cbf6aae736ad97a7aa29167525dcd405196dfbc064672a730d3cf /tmp/local-up-cluster.sh.iU0gc3/cni.amd64.tgz
/tmp/local-up-cluster.sh.iU0gc3/cni.amd64.tgz: OK
./
./macvlan
./static
./vlan
./portmap
./host-local
./vrf
./bridge
./tuning
./firewall
./host-device
./sbr
./loopback
./dhcp
./ptp
./ipvlan
./bandwidth
Configuring cni
{
  "cniVersion": "0.4.0",
  "name": "containerd-net",
  "plugins": [
    {
      "type": "bridge",
      "bridge": "cni0",
      "isGateway": true,
      "ipMasq": true,
      "promiscMode": true,
      "ipam": {
        "type": "host-local",
        "ranges": [
          [{
            "subnet": "10.88.0.0/16"
          }],
          [{
            "subnet": "2001:4860:4860::/64"
          }]
        ],
        "routes": [
          { "dst": "0.0.0.0/0" },
          { "dst": "::/0" }
        ]
      }
    },
    {
      "type": "portmap",
      "capabilities": {"portMappings": true}
    }
  ]
}
WARNING : The kubelet is configured to not fail even if swap is enabled; production deployments should disable swap unless testing NodeSwap feature.
2023/07/28 16:58:46 [INFO] generate received request
2023/07/28 16:58:46 [INFO] received CSR
2023/07/28 16:58:46 [INFO] generating key: rsa-2048
2023/07/28 16:58:46 [INFO] encoded CSR
2023/07/28 16:58:46 [INFO] signed certificate with serial number 402094159703320247669839439951229863298823668651
kubelet ( 49854 ) is running.
wait kubelet ready
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
No resources found
time out on waiting 127.0.0.1 info
```

由于首次编译需要一些时间，当看到 `No resources found` 字样的时候即表示成功。

> 期间生成证书到 /var/run/kubernetes 目录
>
>
> 安装过程如果非root用户的话，需要输入用户密码，我这里在前面已经输入过密码，因此是日志中没有体现出来提醒输入密码的信息
>
>
> 这里的出错信息 “time out on waiting 127.0.0.1 info ” 不清楚什么原因引起的，不过这不影响我们下面的流程

安装成功后，会在 `_output/bin` 目录里生成一些 k8s组件二进制文件

```shell
$ ls -al _output
total 12
drwxrwxr-x  3 sxf sxf 4096 Jul 28 16:55 .
drwxr-xr-x 22 sxf sxf 4096 Jul 28 16:55 ..
lrwxrwxrwx  1 sxf sxf   60 Jul 28 16:55 bin -> /home/sxf/workspace/kubernetes/_output/local/bin/linux/amd64
drwxrwxr-x  5 sxf sxf 4096 Jul 28 16:51 local

$ ls ./_output/bin
cloud-controller-manager  kube-apiserver  kube-controller-manager  kube-proxy  kube-scheduler  kubectl  kubelet
```

可以看到它其实是一个链接。

> 目录 _output 占用空间比较多，大概会占用几个G。

此时再开启一个终端，会发现有以下三个个进程。

```shell
$ ps -a|grep kube
  49297 pts/1    00:01:10 kube-apiserver
  49600 pts/1    00:00:45 kube-controller
  49605 pts/1    00:00:08 kube-scheduler
  49600 pts/1    00:00:45 kubelet
  49605 pts/1    00:00:08 kube-proxy
```

默认 `kube-apiserver` 进程监听在 `6443` 端口，`kube-controller` 监听在`10257` 端口， `kube-scheduler` 则监听在 `10259` 端口，`kube-proxy` 监听在 `10256` 端口，还有 `kubelet` 监听在 `10250` 和 `10248(healthz endpoint)` 端口，可以通过这几个端口号确认服务是否正常运行。

如果以后重启集群的话，只需要执行一下命令即可

```shell
$ ENABLE_DAEMON=true DBG=1 ./hack/local-up-cluster.sh  -O
skipped the build.
API SERVER secure port is free, proceeding...
Detected host and ready to start services.  Doing some housekeeping first...
...
```

此命令将跳过编译步骤，直接运行上次生成的二进制文件。

对于 `local-up-cluster.sh` 的用法参考

```shell
$ ./hack/local-up-cluster.sh -h
This script starts a local kube cluster.
Example 0: hack/local-up-cluster.sh -h  (this 'help' usage description)
Example 1: hack/local-up-cluster.sh -o _output/dockerized/bin/linux/amd64/ (run from docker output)
Example 2: hack/local-up-cluster.sh -O (auto-guess the bin path for your platform)
Example 3: hack/local-up-cluster.sh (build a local copy of the source)
```

## 确认集群服务 

首先设置环境变量，指定连接集群配置文件

```shell
$ export KUBECONFIG=/var/run/kubernetes/admin.kubeconfig
```

这时使用 `./cluster/kubectl.sh` 脚本验证

```shell
$ ./cluster/kubectl.sh get nodes
NAME        STATUS   ROLES    AGE   VERSION
127.0.0.1   Ready    <none>   9ms   v1.27.3-dirty

$ ./cluster/kubectl.sh get ns
NAME              STATUS   AGE
default           Active   10m
kube-node-lease   Active   10m
kube-public       Active   10m
kube-system       Active   10m

$ ./cluster/kubectl.sh get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.0.0.1     <none>        443/TCP   10m33s
```

其它验证命令

```shell
$ ./cluster/kubectl.sh get pods
$ ./cluster/kubectl.sh get replicationcontrollers
```

在等待配置完成时，您可以使用这些命令在另一个终端中观察进度。

```shell
# containerd
# To list images
ctr --namespace k8s.io image ls
# To list containers
ctr --namespace k8s.io containers ls
```

会显示出来所有下载的镜像和创建的容器

创建一个pod

```shell
$ ./cluster/kubectl.sh create -f test/fixtures/doc-yaml/user-guide/pod.yaml
```

> 官方文档提示说其局限性在于仅支持给定 Pod 的单个副本。但经过测试是可以通过deployment创建多副本的pod的。

查看pod

```shell
$ ./cluster/kubectl.sh get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          12m
```

测试Pod启动成功。

# dlv 启动 apiserver 

先找出来 `kube-apiserver` 进程PID

```shell
$ ps aux | grep kube-apiserver
root       49292  0.0  0.1   9280  4512 pts/1    S    16:55   0:00 sudo -E /home/sxf/workspace/kubernetes/_output/local/bin/linux/amd64/kube-apiserver --authorization-mode=Node,RBAC  --cloud-provider= --cloud-config=   --v=3 --vmodule= --audit-policy-file=/tmp/local-up-cluster.sh.iU0gc3/kube-audit-policy-file --audit-log-path=/tmp/kube-apiserver-audit.log --authorization-webhook-config-file= --authentication-token-webhook-config-file= --cert-dir=/var/run/kubernetes --egress-selector-config-file=/tmp/local-up-cluster.sh.iU0gc3/kube_egress_selector_configuration.yaml --client-ca-file=/var/run/kubernetes/client-ca.crt --kubelet-client-certificate=/var/run/kubernetes/client-kube-apiserver.crt --kubelet-client-key=/var/run/kubernetes/client-kube-apiserver.key --service-account-key-file=/tmp/local-up-cluster.sh.iU0gc3/kube-serviceaccount.key --service-account-lookup=true --service-account-issuer=https://kubernetes.default.svc --service-account-jwks-uri=https://kubernetes.default.svc/openid/v1/jwks --service-account-signing-key-file=/tmp/local-up-cluster.sh.iU0gc3/kube-serviceaccount.key --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,Priority,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota,NodeRestriction --disable-admission-plugins= --admission-control-config-file= --bind-address=0.0.0.0 --secure-port=6443 --tls-cert-file=/var/run/kubernetes/serving-kube-apiserver.crt --tls-private-key-file=/var/run/kubernetes/serving-kube-apiserver.key --storage-backend=etcd3 --storage-media-type=application/vnd.kubernetes.protobuf --etcd-servers=http://127.0.0.1:2379 --service-cluster-ip-range=10.0.0.0/24 --feature-gates=AllAlpha=false --external-hostname=localhost --requestheader-username-headers=X-Remote-User --requestheader-group-headers=X-Remote-Group --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-client-ca-file=/var/run/kubernetes/request-header-ca.crt --requestheader-allowed-names=system:auth-proxy --proxy-client-cert-file=/var/run/kubernetes/client-auth-proxy.crt --proxy-client-key-file=/var/run/kubernetes/client-auth-proxy.key --cors-allowed-origins=/127.0.0.1(:[0-9]+)?$,/localhost(:[0-9]+)?$

root       49297  6.6  7.4 1120168 298608 pts/1  Sl   16:55   1:14 /home/sxf/workspace/kubernetes/_output/local/bin/linux/amd64/kube-apiserver --authorization-mode=Node,RBAC  --cloud-provider= --cloud-config=   --v=3 --vmodule= --audit-policy-file=/tmp/local-up-cluster.sh.iU0gc3/kube-audit-policy-file --audit-log-path=/tmp/kube-apiserver-audit.log --authorization-webhook-config-file= --authentication-token-webhook-config-file= --cert-dir=/var/run/kubernetes --egress-selector-config-file=/tmp/local-up-cluster.sh.iU0gc3/kube_egress_selector_configuration.yaml --client-ca-file=/var/run/kubernetes/client-ca.crt --kubelet-client-certificate=/var/run/kubernetes/client-kube-apiserver.crt --kubelet-client-key=/var/run/kubernetes/client-kube-apiserver.key --service-account-key-file=/tmp/local-up-cluster.sh.iU0gc3/kube-serviceaccount.key --service-account-lookup=true --service-account-issuer=https://kubernetes.default.svc --service-account-jwks-uri=https://kubernetes.default.svc/openid/v1/jwks --service-account-signing-key-file=/tmp/local-up-cluster.sh.iU0gc3/kube-serviceaccount.key --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,Priority,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota,NodeRestriction --disable-admission-plugins= --admission-control-config-file= --bind-address=0.0.0.0 --secure-port=6443 --tls-cert-file=/var/run/kubernetes/serving-kube-apiserver.crt --tls-private-key-file=/var/run/kubernetes/serving-kube-apiserver.key --storage-backend=etcd3 --storage-media-type=application/vnd.kubernetes.protobuf --etcd-servers=http://127.0.0.1:2379 --service-cluster-ip-range=10.0.0.0/24 --feature-gates=AllAlpha=false --external-hostname=localhost --requestheader-username-headers=X-Remote-User --requestheader-group-headers=X-Remote-Group --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-client-ca-file=/var/run/kubernetes/request-header-ca.crt --requestheader-allowed-names=system:auth-proxy --proxy-client-cert-file=/var/run/kubernetes/client-auth-proxy.crt --proxy-client-key-file=/var/run/kubernetes/client-auth-proxy.key --cors-allowed-origins=/127.0.0.1(:[0-9]+)?$,/localhost(:[0-9]+)?$
sxf        54148  0.0  0.0   6404  2456 pts/0    S+   17:14   0:00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn --exclude-dir=.idea --exclude-dir=.tox kube-apiserver
```

> 我这里使用的普通用户启动的服务，因此 kube-apiserver 的进程ID为 `49297`

## 侵入服务PID 

```shell
$ dlv --listen=:2345 --headless=true --api-version=2 attach <PID>
```

如果遇到以下错误

```shell
$ dlv --listen=:2345 --headless=true --api-version=2 attach 49297
API server listening at: [::]:2345
2023-07-28T09:20:59Z warning layer=rpc Listening for remote connections (connections are not authenticated nor encrypted)
2023-07-28T09:20:59Z info layer=debugger attaching to pid 49297
Could not attach to pid 49297: this could be caused by a kernel security setting, try writing "0" to /proc/sys/kernel/yama/ptrace_scope
```

需要修改内核配置

```shell
$ echo 0 > /proc/sys/kernel/yama/ptrace_scope
```

## 重新启动服务 

使用这种方法，命令格式将发生变化，假如原来的命令是

```shell
$ ./myApp --config=/path/config.json
```

对应的新命令为

```shell
$ dlv --listen=:2345 --headless=true --api-version=2 exec ./myApp -- --config=/path/config.conf
```

主要有两点变化：

 1. 首先在原来命令 `./myApp` 前添加 `dlv --listen=:2345 --headless=true --api-version=2 exec`
 2. 将原来的命令与参数 `--config=/path/config.json`之间使用 `--` 分隔开

下面我们按这种方法重启 `kube-apiserver` 服务。

先将原来的进程 `kill` 掉，然后用 dlv 命令启动这个服务，原来所有参数要保证不变。

```shell
$ kill -9 49292
```

使用 dlv 重新启动 `kube-apiserver` 服务

```shell
$ dlv --listen=:2345 --headless=true --api-version=2 exec /home/sxf/workspace/kubernetes/_output/local/bin/linux/amd64/kube-apiserver -- --authorization-mode=Node,RBAC  --cloud-provider= --cloud-config=   --v=3 --vmodule= --audit-policy-file=/tmp/local-up-cluster.sh.iU0gc3/kube-audit-policy-file --audit-log-path=/tmp/kube-apiserver-audit.log --authorization-webhook-config-file= --authentication-token-webhook-config-file= --cert-dir=/var/run/kubernetes --egress-selector-config-file=/tmp/local-up-cluster.sh.iU0gc3/kube_egress_selector_configuration.yaml --client-ca-file=/var/run/kubernetes/client-ca.crt --kubelet-client-certificate=/var/run/kubernetes/client-kube-apiserver.crt --kubelet-client-key=/var/run/kubernetes/client-kube-apiserver.key --service-account-key-file=/tmp/local-up-cluster.sh.iU0gc3/kube-serviceaccount.key --service-account-lookup=true --service-account-issuer=https://kubernetes.default.svc --service-account-jwks-uri=https://kubernetes.default.svc/openid/v1/jwks --service-account-signing-key-file=/tmp/local-up-cluster.sh.iU0gc3/kube-serviceaccount.key --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,Priority,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota,NodeRestriction --disable-admission-plugins= --admission-control-config-file= --bind-address=0.0.0.0 --secure-port=6443 --tls-cert-file=/var/run/kubernetes/serving-kube-apiserver.crt --tls-private-key-file=/var/run/kubernetes/serving-kube-apiserver.key --storage-backend=etcd3 --storage-media-type=application/vnd.kubernetes.protobuf --etcd-servers=http://127.0.0.1:2379 --service-cluster-ip-range=10.0.0.0/24 --feature-gates=AllAlpha=false --external-hostname=localhost --requestheader-username-headers=X-Remote-User --requestheader-group-headers=X-Remote-Group --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-client-ca-file=/var/run/kubernetes/request-header-ca.crt --requestheader-allowed-names=system:auth-proxy --proxy-client-cert-file=/var/run/kubernetes/client-auth-proxy.crt --proxy-client-key-file=/var/run/kubernetes/client-auth-proxy.key
```

注意

 1. 使用 `dlv` 参数 `--headless exec` 命令启动 `kube-apiserver`
 2. 为了远程调试，还需要给 kube-apiserver 指定服务监听地址，这里为 `:2345`

> 这里将原来的参数 `--cors-allowed-origins=127.0.0.1(:[0-9]+)?$,localhost(:[0-9]+)?$` 删除了，否则会提示bash脚本错误

# 远程调试(本地) 

上面我们已将远程调试环境搞完了，调试服务监听的是默认端口 `:2345` ,现在我们使用 GoLand 开发工具来进行远程调试。

我们按上一篇文章的方法配置一下 `Go Deubg` 信息 ，并设置一些断点。![image-20230728174843848](https://blogstatic.haohtml.com/uploads/2023/07/a881ee004b14b3f7f51c73a11755fcee-6.webp)

> 这里填写的远程调试服务为 kube-apiserver 监听地址信息，因此只能对本地的 kube-apiserver 文件进行调试。
>
>
> 有一点需要注意的是，如果本地文件进行了修改的话，没有办法像原来在本机一样立即能看到效果，这个只能在远程重新编译成二进制才可以，也就是是远程二进制对应的是本地的源代码，

![image-20230729012956772](https://blogstatic.haohtml.com/uploads/2023/07/2a033afdd8755baa61d8235375907181-6.webp)

可以看到在断点停止运行，我们点击 `绿色右剪头` 执行下一个断点。![image-20230729013221364](https://blogstatic.haohtml.com/uploads/2023/07/7f05c085b7a205877097174a65063ce6-6.webp)![image-20230729013726272](https://blogstatic.haohtml.com/uploads/2023/07/93d79fa209faa6820b956535fcb70f67-6.webp)

我们再看一下远程调试服务器终端输出![image-20230729014050598](https://blogstatic.haohtml.com/uploads/2023/07/5849eb23758e4f06a1a411d652e251ab-6.webp)

可以看到程序的标准输出被打印出来了。

也可以创建或删除一些 Pod 来观察调试信息，当然要找到对应打断点的地方。

apiserver 包含一个webservice服务，所以也可以通过web的方式访问apiserver。如

```shell
$ kubectl proxy --port 8080
$ curl http://localhost:8080/api/v1/namespaces/default/pods
```

# 总结 

启动 `kube-apiserver` 服务前要保证其依赖服务(etcd)没有问题才可以。同样对于其它k8s 组件如 `kube-controller` 或 `kube-scheduler`，它们均可以采用这种方法进行远程调试。

上面我们主要用到两个脚本，一个是 `./hack/install-etcd.sh` 用来下载 `etcd` 二进制文件到本地，另一个是 `./hack/local-up-cluster.sh` 用来启动k8s集群，这个脚本会执行一些任务，如启动 `etcd` 服务、生成证书等。

> 如果你是本地调试的话，则这两步同样需要在本地执行，待集群成功启动后，记录一下 kube-apiserver 进程的参数，然后再将这个进行kill掉。然后在Goland 里创建一个调试apiserver的配置，并将上面记录的参数添加上，开始调试即可。

而远程调试这种方法对于一些大型复杂的程序比较合适，而对于小型应用，如果条件允许的话建议还是本地调试，毕竟远程调试还是挺复杂的。

这里我使用的Linux 下的普通用户，过程中有时候会遇到权限问题，因此建议直接使用 `root` 用户权限，毕竟是自己的开发环境，能省事尽量省事。

> 本地集群暂时没有办法在macOS上启动，否则会提示以下错误
>
> kubelet is not currently supported in darwin, kubelet aborted.
>
> kubelet is not currently supported in darwin, kube-proxy aborted.

# 参考资料 

 * [https://github.com/cloudflare/cfssl](https://github.com/cloudflare/cfssl)
 * [https://github.com/go-delve/delve/tree/master/Documentation/usage](https://github.com/go-delve/delve/tree/master/Documentation/usage)
 * [https://www.jetbrains.com/help/go/attach-to-running-go-processes-with-debugger.html#attach-to-a-process-on-a-remote-machine](https://www.jetbrains.com/help/go/attach-to-running-go-processes-with-debugger.html#attach-to-a-process-on-a-remote-machine)
 * [https://github.com/kubernetes/community/blob/master/contributors/devel/development.md](https://github.com/kubernetes/community/blob/master/contributors/devel/development.md)
 * [https://github.com/kubernetes/community/blob/master/contributors/devel/running-locally.md](https://github.com/kubernetes/community/blob/master/contributors/devel/running-locally.md)

 [1]: https://blog.haohtml.com/archives/34402