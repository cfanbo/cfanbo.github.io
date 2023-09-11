---
title: 一个docker-compose微服务脚本，自用
author: admin
type: post
date: 2018-07-05T06:45:06+00:00
url: /archives/17925
categories:
 - 系统架构
tags:
 - docker

---
容器为swoole+php7

docker-compose.yml

```
version: '3.6'
services:
  redis:
    image: redis
  web:
    image: cfanbo/swoole4_php7:v1
    depends_on:
      - redis
    links:
      - redis
    volumes:
      - /Users/sxf/sites/msgserve:/usr/src/myapp
    command: "php /usr/src/myapp/src/wx_push_server.php start"
```

对于 wx_push_server.php文件里redis的主机地址应该写成docker-compose配置文件里的容器服务名(redis)