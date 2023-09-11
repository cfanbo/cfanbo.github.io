---
title: nodejs中package.json文件说明
author: admin
type: post
date: 2016-03-12T12:04:54+00:00
url: /archives/16777
categories:
 - 程序开发
tags:
 - nodejs

---
推荐： [http://jingpin.jikexueyuan.com/article/34254.html](http://jingpin.jikexueyuan.com/article/34254.html)

package.json 中包含各种所需模块以及项目的配置信息（名称、版本、许可证等）meta 信息。

package.json文件可以通过npm init 来创建

#### 包含可配置项

 * name 名称
 * 应用描述 description
 * 版本号 version
 * 应用的配置项 config
 * 作者 author
 * 资源仓库地址 respository
 * 授权方式 licenses
 * 目录 directories
 * 应用入口文件 main
 * 命令行文件 bin
 * 项目应用运行依赖模块 dependencies
 * 项目应用开发环境依赖 devDependencies
 * 运行引擎 engines
 * 脚本 script

简单模式

==========================

```
{

    name: "myApp",

    version :"0.0.1"

}
```

完整模式

===========================

```
{

  "name": "myApp",
  "version": "0.0.0",
  "author" : "simple",
  "description" : "Nodejs Package json介绍",
  "keywords" : "javascript, nodejs",
  "respository" : {
      "type" :"git",
      "url" :"http://path/to/url"
    },

  "bugs" : {
      "url" : "http://path/to/bug",
      "email" : "bug@example.com"
    },
 "contributors" : [

  {"name" : "zhangsan", "email" : "zhangsan@example.com"

  ]

  "license" : "MIT",
  "engines" : { "node" : "0.10.x"},
  "script" : {
    "start" : "node index.js"
  },
  "private": true,
  "scripts": {
  "start": "node ./bin/www"
  },

  "dependencies": {
    "express": "~4.9.0",
    "body-parser": "~1.8.1",
    "cookie-parser": "~1.3.3",
    "morgan": "~1.3.0",
    "serve-favicon": "~2.1.3",
     "debug": "~2.0.0",
    "jade": "~1.6.0"
  },

  "devDependencies": {
    "bower" : "~1.2.8",
    "grunt" : "~0.4.1",
    "grunt-contrib-concat" : "~0.3.0",
    "grunt-contrib-jshint" : "~0.7.2",
    "grunt-contrib-uglify" : "~0.2.7",
    "grunt-contrib-clean"  : "~0.5.0",
    "browserify" : "2.36.1",
    "grunt-browserify" : "~1.3.0"
  }
}
```

#### 1.scripts

运行指定脚本命令。

#### 2.

#### npm install express –save

#### npm install express –save-dev

#### 上面代码表示单独安装express模块，

–save参数表示将该模块写入dependencies属性，

–save-dev表示将该模块写入devDependencies属性。

#### 3.关于指定版本号

_（1）波浪号~（tilde）+指定版本：比如~1.2.2，表示安装1.2.x的最新版本（不低于1.2.2），但是不安装1.3.x，也就是说安装时不改变大版本号和次要版本号。_

package.json 文件用处：


直接转到当前项目目录下用命令npm install 或npm install –save-dev安装即可，自动将package.json中的模块安装到node-modules文件夹下。


—————————-


主要有两个疑惑:

1.package.json是提供给nodes和nam使用的，如果您开发的是一个应用的话，则此文件可以不需要，而如果你开发的是一个模块的话，则需要此文件来记录相关的依赖关系。

2.package.json是npm安装模块时的依据(目前理解的). 那如果是一个应用项目而不是一个功能性的模块, 那我也需要编写package.json吗?


1. node,npm 都要用。

    1.1 node在调用require的时候去查找模块，会按照一个次序去查找，package.json会是查找中的一个环节。见阮一峰的require分析 [http://www.ruanyifeng.com/blog/2015/05/require.html](http://www.ruanyifeng.com/blog/2015/05/require.html)

    1.2 npm用的就比较多，其中的 “dependencies” 字段就是本模块的依赖的模块清单。每次npm update的时候，npm会自动的把依赖到的模块也下载下来。当npm install 本模块的时候，会把这里提到的模块都一起下载下来。通过package.json,就可以管理好模块的依赖关系。

2. 如果是应用，不必编写package.json


———————–