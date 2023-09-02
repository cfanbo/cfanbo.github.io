---
title: kubernetes中apiserver的证书
author: admin
type: post
date: 2018-09-29T04:30:06+00:00
url: /archives/18312
categories:
 - 系统架构
tags:
 - kubernetes

---
在kubernetes中，与api server 通讯时一般都需要使用https证书，这些证书文件存在放 /etc/kubernetes/pki 目录中(ubuntu)。主要有以下几种

```
/etc/kubernetes/pki/ca.{crt,key}
```

如果你已有现成的证书也可以直接将证书复制到这个目录里即可。这时kubeadm就会跳过证书生成这个步骤。

证书生成后，kubeadm 接下来会为其它组件生成访问api server 所需要的配置文件，这些文件路径为: /etc/kubernetes/xxx.conf:

```
ls /etc/kubernetes/
admin.conf controller-manager.conf kubelet.conf scheduler.conf
```

这里可以看到这四个配置文件，分别 为不同的组件之间提供配置。

[![](https://blog.haohtml.com/wp-content/uploads/2018/09/kubernetes-master.jpg)][1]

这些配置文件中存储的是Master节点的ip地址、端口号、证书目录等信息。这样对应的客户端（scheduler，kubelet, controller-manager等）就可以直接加载并读取相应的配置文件来与kube-apiserver 建立安全连接，实现通讯。

附kubernetes架构图

[![](https://blog.haohtml.com/wp-content/uploads/2018/09/kubernetes-arvhitecture.jpg)][2]

 [1]: https://blog.haohtml.com/wp-content/uploads/2018/09/kubernetes-master.jpg
 [2]: https://blog.haohtml.com/wp-content/uploads/2018/09/kubernetes-arvhitecture.jpg