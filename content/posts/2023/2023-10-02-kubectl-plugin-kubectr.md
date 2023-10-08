---
title: kubectr 一款快速查看Pod容器的kubectl插件
url: "/posts/kubectl-plugin-kubectr"
date: 2023-10-02T10:14:18+08:00
type: post
toc: true
categories: 
- 程序开发
tags:
- kubectl
- krew
- kubectr
---

以前工作中经常需要查看Pod里容器相关信息，特别是容器镜像信息，以前一直是通过 `kubectl describe`命令查看的

```sh
$ kubectl describe my-pod
```

但由于输出的内容特别多，查看容器关键信息特别麻烦。印象最深的莫过于在部署 `istio`时，由于国内网络环境不稳定，经常性的遇到镜像下载失败的情况，当时极其的头疼。

于是最近花了一点时间，开发了一款快速查看 Pod 容器信息的插件 [kubectr](https://github.com/cfanbo/kubectr) 。

# 安装

安装方法主要有三种

## krew 安装（推荐）

```sh
$ kubectl krew install ctr
```

> 目前已提交到 [krew](https://github.com/kubernetes-sigs/krew) ，但由于官方审核速度较慢，此安装方法不敢保证可用



## 二进制安装

从 https://github.com/cfanbo/kubectr/releases 下载对应的平台版本，并解压到对应的 PATH  环境变量目录即可。

```sh
$ tar zxvf kubectr_linux_amd64.tar.gz
$ sudo mv kubectr /usr/local/bin/
$ kubectr -h
```



## 源码安装

```sh
$ git clone https://github.com/cfanbo/kubectr.git
$ make 
$ bin/kubectr -h
```



# 用法

共两种用法，一种是 krew 风格的插件用法 ，另一种是普通命令格式的用法。

## `krew` 插件用法

```sh
$ kubectl ctr csi-do-controller-0 -n kube-system

NAME           	  READY	  STATUS 	  RESTARTS     	  AGE  	  PORTS	  IMAGE                                             	  PULLPOLICY  	  TYPE
csi-provisioner	  1    	  Running	  5 (3d21h ago)	  3d21h	  -    	  registry.k8s.io/sig-storage/csi-provisioner:v3.5.0	  IfNotPresent	  container
csi-attacher   	  1    	  Running	  5 (3d21h ago)	  3d21h	  -    	  registry.k8s.io/sig-storage/csi-attacher:v4.3.0   	  IfNotPresent	  container
csi-snapshotter	  1    	  Running	  5 (3d21h ago)	  3d21h	  -    	  registry.k8s.io/sig-storage/csi-snapshotter:v6.2.2	  IfNotPresent	  container
csi-resizer    	  1    	  Running	  5            	  3d21h	  -    	  registry.k8s.io/sig-storage/csi-resizer:v1.8.0    	  IfNotPresent	  container
csi-do-plugin  	  0    	  Waiting	  1672 (3m ago)	  -    	  -    	  digitalocean/do-csi-plugin:v4.7.1                 	  Always      	  container
```



## 普通命令用法

```shell
$ kubectr ephemeral-demo

NAME          	  READY	  STATUS    	  RESTARTS	  AGE 	  PORTS	  IMAGE                    	  PULLPOLICY  	  TYPE
ephemeral-demo	  1    	  Running   	  0       	  1d4h	  -    	  registry.k8s.io/pause:3.1	  IfNotPresent	  container
debugger-kbm5m	  0    	  Terminated	  0       	  -   	  -    	  busybox:1.28             	  IfNotPresent	  ephemeralContainer
```

目前插件支持普通 `container`、`initContainer` 和 `ephemeralContainer`三类容器。

# 版本号

```sh
$ kubectl ctr -v
Version: v0.0.1
GitCommit: 0bd08fb
runtimeVersion: go1.21.1
ProjectURL: github.com/cfanbo/kubectr
```

# 帮助

```sh
$ kubectl ctr -h
display all containers in the pod.

You can invoke ctr through kubectl: "kubectl ctr podName"

Usage:
  ctr podName [flags]

Flags:
      --as string                      Username to impersonate for the operation. User could be a regular user or a service account in a namespace.
      --as-group stringArray           Group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --as-uid string                  UID to impersonate for the operation.
      --cache-dir string               Default cache directory (default "/Users/sxf/.kube/cache")
      --certificate-authority string   Path to a cert file for the certificate authority
      --client-certificate string      Path to a client certificate file for TLS
      --client-key string              Path to a client key file for TLS
      --cluster string                 The name of the kubeconfig cluster to use
      --context string                 The name of the kubeconfig context to use
      --disable-compression            If true, opt-out of response compression for all requests to the server
  -h, --help                           help for ctr
      --insecure-skip-tls-verify       If true, the server's certificate will not be checked for validity. This will make your HTTPS connections insecure
      --kubeconfig string              Path to the kubeconfig file to use for CLI requests.
  -n, --namespace string               If present, the namespace scope for this CLI request
      --request-timeout string         The length of time to wait before giving up on a single server request. Non-zero values should contain a corresponding time unit (e.g. 1s, 2m, 3h). A value of zero means don't timeout requests. (default "0")
  -s, --server string                  The address and port of the Kubernetes API server
      --tls-server-name string         Server name to use for server certificate validation. If it is not provided, the hostname used to contact the server is used
      --token string                   Bearer token for authentication to the API server
      --user string                    The name of the kubeconfig user to use
  -v, --version                        Displays the current version number
```



如果对此工具有疑问或bug、新功能需求反馈，可以在 https://github.com/cfanbo/kubectr/issues 中反馈。
