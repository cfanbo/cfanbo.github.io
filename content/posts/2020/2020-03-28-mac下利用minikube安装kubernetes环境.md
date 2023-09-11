---
title: mac下利用minikube安装Kubernetes环境
author: admin
type: post
date: 2020-03-28T03:44:44+00:00
url: /archives/19920
categories:
 - 系统架构
tags:
 - k8s

---
本机为mac环境，安装有brew工具，所以为了方便这里直接使用brew来安装minikube工具。同时本机已经安装过VirtualBox虚拟机软件。

minikube是一款专门用来创建k8s 集群的工具。

## 一、安装minikube 

参考 , 在安装minkube之前建议先了解一下minikube需要的环境。

1. 先安装一个虚拟化管理系统，如果还未安装，则在 HyperKit、VirtualBox 或 VMware Fusion 三个中任选一个即可，这里我选择了VirtualBox。

如果你想使用hyperkit的话，可以直接执行 **brew install hyperkit** 即可。

对于支持的driver_name有效值参考, 目前**docker**尚处于实现阶段。

```
$ brew install minikube
```

查看版本号

```
$ minikube version
```

minikube version: v1.8.2
commit: eb13446e786c9ef70cb0a9f85a633194e62396a1

安装kubectl命令行工具

```
$ brew install kubectl
```

## 二、启动minikube 创建集群 

```
$ minikube start --driver=virtualbox
```

如果国内的用户安装时提示失败”VM is unable to access k8s.gcr.io, you may need to configure a proxy or set –image-repository”,
则指定参数_–image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers_

```
 $ minikube start --driver=virtualbox --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
```

```
😄 minikube v1.8.2 on Darwin 10.15.3
✨ Using the virtualbox driver based on existing profile
✅ Using image repository registry.cn-hangzhou.aliyuncs.com/google_containers
💾 Downloading preloaded images tarball for k8s v1.17.3 …
⌛ Reconfiguring existing host …
🏃 Using the running virtualbox “minikube” VM …
🐳 Preparing Kubernetes v1.17.3 on Docker 19.03.6 …
> kubelet.sha256: 65 B / 65 B [————————–] 100.00% ? p/s 0s
> kubeadm.sha256: 65 B / 65 B [————————–] 100.00% ? p/s 0s
> kubectl.sha256: 65 B / 65 B [————————–] 100.00% ? p/s 0s
> kubeadm: 37.52 MiB / 37.52 MiB [—————] 100.00% 1.01 MiB p/s 37s
> kubelet: 106.42 MiB / 106.42 MiB [————-] 100.00% 2.65 MiB p/s 40s
> kubectl: 41.48 MiB / 41.48 MiB [—————] 100.00% 1.06 MiB p/s 40s
🚀 Launching Kubernetes …
🌟 Enabling addons: default-storageclass, storage-provisioner
🏄 Done! kubectl is now configured to use “minikube”
```
另外**在minikube start** 有一个选项是 **–image-mirror-country=’cn’** 这个选项是专门为中国准备的，还有参数 **–iso-url**，官方文档中已经提供了阿里云的地址,这个选项会让你使用阿里云的镜像仓库，我这里直接指定了镜像地址。

对于大部分国内无法访问到的镜像 k8s.gcr.io 域名下的镜像都可以在 找到。

查看集群状态

```
$ minikube status
```

如果输出结果如下，则表示安装成功
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

如果你以前安装过minikube,在 minikube start 的时候返回错误 machine does not exist，则需要执行 minikube delete 清理本地状态。

这时创建完成后，会在VirtualBox软件里出现一个虚拟系统。![](https://blog.haohtml.com/wp-content/uploads/2020/03/k8s-virtualbox.png)

另外minikube 也支持 –driver=none参数

## 三、进阶篇 

参考 [https://kubernetes.io/docs/tutorials/hello-minikube/#before-you-begin](https://kubernetes.io/docs/tutorials/hello-minikube/#before-you-begin)

打开Kubernetes仪表板

```
$ minikube dashboard
```
```
🔌 Enabling dashboard …
🤔 Verifying dashboard health …
🚀 Launching proxy …
🤔 Verifying proxy health …
🎉 Opening http://127.0.0.1:59915/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser…
```

在浏览器中自动打开上方网址， 就可以看到 k8s 控制台。同时也可以根据上方网址教程来学习。![](https://blog.haohtml.com/wp-content/uploads/2020/03/Kubernetes-Dashboard-1024x562.png)

具体的操作，你可以查看 Dashboard 项目的[官方文档][1]。

## 四、停止集群 

```
$ minikube stop
```

## 五、删除集群 

```
$ minikube delete
```

## 六、升级集群 

```
$ brew update
$*  brew upgrade minikube
```
 
mac 的请参考[ https://minikube.sigs.k8s.io/docs/start/macos/#upgrading-minikube]( https://minikube.sigs.k8s.io/docs/start/macos/#upgrading-minikube)

## 参考资料 
* https://kubernetes.io/docs/setup/learning-environment/minikube/
* https://kubernetes.io/docs/tasks/tools/install-minikube/
* https://kubernetes.io/docs/tutorials/hello-minikube/#before-you-begin

 [1]: https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/