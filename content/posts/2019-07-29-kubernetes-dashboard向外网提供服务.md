---
title: kubernetes dashboard向外网提供服务
author: admin
type: post
date: 2019-07-29T07:07:20+00:00
url: /archives/19201
categories:
 - 系统架构
tags:
 - k8s

---
目前新版本的 kubernetes dashboard （）安装了后，为了安全起见，默认情况下已经不向外提供服务，只能通过 [`http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/`][1] 本机访问。在我们学习过程中，总有些不方便，这时我们可以利用 `kubectl proxy` 命令来实现。

首先我们看一下此命令的一些想着参数

```
➜  ~ kubectl proxy -h
To proxy all of the kubernetes api and nothing else, use:

  $ kubectl proxy --api-prefix=/

To proxy only part of the kubernetes api and also some static files:

  $ kubectl proxy --www=/my/files --www-prefix=/static/ --api-prefix=/api/

The above lets you 'curl localhost:8001/api/v1/pods'.

To proxy the entire kubernetes api at a different root, use:

  $ kubectl proxy --api-prefix=/custom/

The above lets you 'curl localhost:8001/custom/api/v1/pods'

Examples:
  # Run a proxy to kubernetes apiserver on port 8011, serving static content from ./local/www/
  kubectl proxy --port=8011 --www=./local/www/

  # Run a proxy to kubernetes apiserver on an arbitrary local port.
  # The chosen port for the server will be output to stdout.
  kubectl proxy --port=0

  # Run a proxy to kubernetes apiserver, changing the api prefix to k8s-api
  # This makes e.g. the pods api available at localhost:8011/k8s-api/v1/pods/
  kubectl proxy --api-prefix=/k8s-api

Options:
      --accept-hosts='^localhost$,^127\.0\.0\.1$,^\[::1\]$': Regular expression for hosts that the proxy should accept.
      --accept-paths='^/.*': Regular expression for paths that the proxy should accept.
      --address='127.0.0.1': The IP address on which to serve on.
      --api-prefix='/': Prefix to serve the proxied API under.
      --disable-filter=false: If true, disable request filtering in the proxy. This is dangerous, and can leave you
vulnerable to XSRF attacks, when used with an accessible port.
  -p, --port=8001: The port on which to run the proxy. Set to 0 to pick a random port.
      --reject-methods='POST,PUT,PATCH': Regular expression for HTTP methods that the proxy should reject.
      --reject-paths='^/api/.*/pods/.*/exec,^/api/.*/pods/.*/attach': Regular expression for paths that the proxy should
reject.
  -u, --unix-socket='': Unix socket on which to run the proxy.
  -w, --www='': Also serve static files from the given directory under the specified prefix.
  -P, --www-prefix='/static/': Prefix to serve static files under, if static file directory is specified.

Usage:
  kubectl proxy [--port=PORT] [--www=static-dir] [--www-prefix=prefix] [--api-prefix=prefix] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```

这里我们只要关注其中的三个参数就可以了

```
--accept-hosts='^localhost$,^127\.0\.0\.1$,^\[::1\]$': Regular expression for hosts that the proxy should accept.
--address='127.0.0.1': The IP address on which to serve on.
--port=8001: The port on which to run the proxy. Set to 0 to pick a random port.
```

–accept-hosts 表示哪些客户端访问,默认只允许 localhost 和 127.0.0.1
–address 表示本机绑定的ip地址，如果值为0.0.0.0 则表示不限，通过任何ip都可以访问.
a
–port 表示代理的接口，如果值为0的话，则随机一个端口

这里为了外网访问，可设置如下

```
nohup kubectl proxy --address='0.0.0.0' --port=8888 --accept-hosts='^*$'
```

这时就实现了通过外网访问

[http://192.168.0.107:8888/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/](http://192.168.0.107:8888/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/)

其实说白了，只要你把基本的命令参数搞清楚了，实现起来就方便了，就看你基础牢不牢。

 [1]: http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/