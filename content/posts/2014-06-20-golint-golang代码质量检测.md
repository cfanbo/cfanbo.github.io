---
title: golint—golang代码质量检测
author: admin
type: post
date: 2014-06-20T02:50:54+00:00
url: /archives/15152
categories:
 - 程序开发
tags:
 - golang
 - golint

---
github: [https://github.com/golang/lint](https://github.com/golang/lint)

golint是类似javascript中的jslint的工具，主要功能就是检测代码中不规范的地方。golint用于检测go代码。

## 使用

```
$ go get github.com/golang/lint
$ go install github.com/golang/lint

```

`golint 文件名或者目录`
检测对应的代码。

golint会输出一些代码存在的问题：
比如：

```
recorder.go:55:5: exported var RecordBind should have comment or be unexported
recorder.go:158:1: exported function Record_ErrorRecord should have comment or be unexported
recorder.go:173:6: don't use underscores in Go names; type Data_MemStats should be DataMemStats
recorder.go:179:2: struct field FreeRam should be FreeRAM

```

上面的输出中文件recorder.go，179行，在struct中字段 `FreeRam` 应该是 `FreeRAM`,输出信息非常的详细

golint 会检测的方面：

 * 变量名规范
 * 变量的声明，像var str string = “test”，会有警告，应该var str = “test”
 * 大小写问题，大写导出包的要有注释
 * x += 1 应该 x++

等等
详细可以看官方库示例， [https://github.com/golang/lint/tree/master/testdata](https://github.com/golang/lint/tree/master/testdata)