---
title: linux下git版本过底引起的无法git clone的解决方案
author: admin
type: post
date: 2018-03-12T01:21:23+00:00
url: /archives/17725
categories:
 - 服务器
tags:
 - git

---
刚安装的新系统，git版本为1.8.3，使用git clone命令的时候，提示“… Peer reports incompatible or unsupported protocol version”

只需要升级一下基本包即可。

```shell
sudo yum update nss curl  # nss为名称解析和认证服务 curl为网络请求库
```