---
title: Kubernetes学习资源
author: admin
type: post
date: 2018-10-11T05:38:11+00:00
url: /archives/18393
categories:
 - 系统架构
tags:
 - k8s
 - kubernetes

---
![](https://blogstatic.haohtml.com/uploads/2021/04/1f57f5934f4db11b6e9e473ffd043b03.jpeg)k8s guide

## 准备 

对于一个新手来说，第一步是必须了解什么是 [kubernetees](https://kubernetes.io/zh/docs/concepts/overview/what-is-kubernetes/)、 [设计架构](https://kubernetes.io/zh/docs/concepts/architecture/) 和相关 [概念](https://kubernetes.io/zh/docs/concepts/)。只有在了解了这些的情况下，才能更好的知道k8s中每个组件的作用以及它解决的问题。

## 安装工具 

 * [minikube](https://minikube.sigs.k8s.io/) 参考 [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)
 * [kind](https://kind.sigs.k8s.io/docs/) 参考 [https://kind.sigs.k8s.io/docs/user/quick-start/](https://kind.sigs.k8s.io/docs/user/quick-start/)

以上是安装k8s环境的两种推荐方法，这里更推荐使用kind。主要原因是 `minikube` 只支持单个节点，而 `kind` 可以支持多个节点，这样就可以实现在一台电脑上部署的环境与生产环境一样，方便大家学习。

要实现管理控制 Kubernetes 集群资源如pod、node、service等的管理，还必须安装一个命令工具 [kubectl](https://kubernetes.io/zh/docs/reference/kubectl/kubectl/) ，请参考： [https://kubernetes.io/zh/docs/tasks/tools/](https://kubernetes.io/zh/docs/tasks/tools/)

## 学习文档 

 * Kubernetes 文档 https://kubernetes.io/zh/docs/home/ 
 * Play with Kubernetes [https://labs.play-with-k8s.com/](https://labs.play-with-k8s.com/)
 * Kubernetes中文指南/云原生应用架构实践手册 –  https://jimmysong.io/kubernetes-handbook 