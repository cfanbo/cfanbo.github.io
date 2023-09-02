---
title: windows下使用docker login命令提示403错误的解决办法
author: admin
type: post
date: 2016-02-02T07:38:18+00:00
url: /archives/16526
categories:
 - 服务器
tags:
 - docker

---
windows平台下使用docker，在对image打包上传( [https://hub.docker.com/](https://hub.docker.com/))时 （ [https://docs.docker.com/windows/step_six/](https://docs.docker.com/windows/step_six/)），发现登录docker的时候一直提示403错误

```
docker login --username=youruser --password=yourpassword --email=youremail
```

网友分析 是windows和unix仓库提交网址不一致，引起的问题

```
docker login --username=youruser --password=yourpassword --email=youremail https://index.docker.io/v1/
```

官方给出的提交url，不过这里用windows的url还是登录失败，一样的错误信息，用unix的url可以登录成功，但在push 的时候，却又提示未认证，暂时未找到解决办法。

 * Windows:
 * Unix:

具体可以看上面柳月给的网址或者：

` [https://github.com/docker/hub-feedback/issues/473](https://github.com/docker/hub-feedback/issues/473)

[https://github.com/docker/docker/issues/18019](https://github.com/docker/docker/issues/18019) `

[http://stackoverflow.com/questions/33748919/why-does-docker-login-fail-in-docker-quickstart-terminal-but-work-from-within](http://stackoverflow.com/questions/33748919/why-does-docker-login-fail-in-docker-quickstart-terminal-but-work-from-within)