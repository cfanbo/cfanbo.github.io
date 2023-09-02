---
title: 在Golang中使用json
author: admin
type: post
date: 2016-03-24T07:10:30+00:00
url: /archives/16851
categories:
 - 程序开发
tags:
 - golang

---
> 由于要开发一个小型的web应用，而web应用大部分都会使用json作为数据传输的格式，所以有了这篇文章。

#### 包引用

 import (
 "encoding/json"
 "github.com/bitly/go-simplejson" // for json get
 )


#### 用于存放数据的结构体

 type MyData struct {
 Name string `json:"item"`
 Other float32 `json:"amount"`
 }


这里需要注意的就是后面单引号中的内容。

 `json:"item"`


这个的作用，就是Name字段在从结构体实例编码到JSON数据格式的时候，使用item作为名字。算是一种重命名的方式吧。

#### 编码JSON

 var detail MyData

 detail.Name = "1"
 detail.Other = "2"

 body, err := json.Marshal(detail)
 if err != nil {
 panic(err.Error())
 }


我们使用Golang自带的encoding/json包对结构体进行编码到JSON数据。

 json.Marshal(...)


#### JSON解码

由于Golang自带的json包处理解码的过程较为复杂，所以这里使用一个第三方的包simplejson进行json数据的解码操作。

 js, err := simplejson.NewJson(body)
 if err != nil {
 panic(err.Error())
 }

 fmt.Println(js)


完！

有关simplejson的更多用法见： [http://1.guotie.sinaapp.com/?p=400](http://1.guotie.sinaapp.com/?p=400)

更多用法参考： [http://blog.haohtml.com/archives/16849](http://blog.haohtml.com/archives/16849)

转自： [http://www.cnblogs.com/sitemanager/p/3419970.html](http://www.cnblogs.com/sitemanager/p/3419970.html)