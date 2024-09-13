---
title: 一款基于LLM实现的IDE翻译插件 AI Translate
date: 2024-09-04T14:31:27+08:00
type: post
toc: true
url: /posts/llm-ai-translate-plugin-for-ide
categories:
  - 其它
tags:
  - ide
  - vscode
  - jetbrains
  - plugin
 
---

![logo.png](https://blogstatic.haohtml.com/uploads/2024/09/logo.png)

AI Translate 是一款基于LLM实现的IDE专用翻译插件，主要为开发人员在查看项目源码时，可以快速对一些代码英文注释进行中文翻译，这对于英文有些弱的同学很有帮助，下面讲一下为什么开发这款IDE插件，以及它比其它同类翻译插件有什么好处。



# 为什么开发这款翻译插件

在以前使用IDE看项目源码时，经常遇到长段的英文段落注释信息，理解起来经常会吃力。当时找了一些翻译插件，但都感觉多多少少有些缺陷。

主要表现以下几点：

1. 翻译质量差，有些翻译软件可能将一些专业术语翻译成了其它名词，整体翻译后的语句读起来很让人头疼
2. 由于在源码文件里，多数源码说明 信息都是以注释方式出现的，多数翻译软件无法做到忽略注释符后连贯起来翻译。
3. 目前市面比较好用的翻译插件“沉浸式翻译”，可惜只支持浏览器，并不支持IDE

曾有一段时间的做法是手动从IDE里复制信息、删除注释符、重新整理段落，再到网页翻译里进行翻译，有时翻译的效果不太好，可能同时会用两个翻译软件。

去年看kubernets源码时，再次被折腾一番，当时花了两三个小时开发了一款基于chrome浏览器的插件，主要实现功能是在软件翻译前，将注释符移除和重新整理段落，这样效果稍微好一些，毕竟节省了不少时间，只是仍未解决从IDE复制代码再进行翻译的问题。

前段时间看ETCD源码时，再次被这个折腾一番，最终还是决定开发一款IDE翻译插件，正好可以使用LLM来解决以前传统翻译软件翻译质量不太好的问题。主要改进两点，一个是有一些专业术语被翻译成了普通的单词，读起来很无语；另一个是有些单词没有必要翻译成中文，保持原样即可，但翻译软件硬是翻译成了中文。

调研了几款目前市面上流行的插件，好像都不支持IDE的这种翻译功能。

1. 沉浸式阅读插件功能十分强大，即可以使用传统翻译软件，也可以使用LLM进行翻译，只可惜的是当前只有浏览器插件，并不支持IDE的翻译功能。
2. Alibaba的“通义灵码”IDE插件，功能也十分强大，在IDE工具里直接集成LLM，实现用户交互，可惜的只竟然不支持翻译功能

既然没有轮子，那就造一个吧，插件开发也挺简单的，也花不了太多的时间。

> JetBrains  IDE 插件开发太麻烦了，每次都等于开启两个窗口，内存和CPU几乎吃完了，调试一次运气不好的话，花1个小时也正常，相对来说vscode 插件开发就容易的多了。

# 实现原理

插件的功能很简单，只做一件事：将用户在IDE里选择的文本信息发送到LLM提供的API接口，然后再将接口返回的信息在IDE控制终端输出。

至于这个翻译功能主要是在LLM应用端通过配置 `prompt` 来实现的，告诉LLM你是一个技术开发人员，熟练各种开发语言，将收到的信息整理后，翻译中文并输出就可以了。

在LLM端可以配置多种功能，如是否启用“记忆”功能和一些插件，以实现更多的功能。

![image-20240913165252183](https://blogstatic.haohtml.com//uploads/2024/09/image-20240913165252183.png)

插件功能

![image-20240913165424083](https://blogstatic.haohtml.com//uploads/2024/09/image-20240913165424083.png)

因此可以自定义prompt实现类似的功能，如 程序员变量命名、SQL解释、代码解释等。

如果IDE插件“通义灵码”有翻译功能，则完全满足当前需求了，遗憾就是没有这个翻译功能，这也正是开发这个插件的原因。

# 支持哪些IDE

目前主要支持 VSCode 编辑器和 JetBrains 公司的IDE产品。

- VSCode：https://marketplace.visualstudio.com/items?itemName=cfanbo.ai-translate  ([GitHub](https://github.com/cfanbo/ai-translate))
- JetBrains Marketplace: https://plugins.jetbrains.com/plugin/25313-ai-translate ([GitHub](https://github.com/cfanbo/intellij-ai-translate))



# 使用教程

使用插件前，需要填写通义千问应用的 `APPID` 和  `APPKEY`，如果还没有开通的话，需要开通后(https://bailian.aliyun.com/)，创建应用后可获取这两项信息。

使用方法很简单，共两步：

1. 在  IDE 里选择要翻译的内容，一般指注释信息
2. 右键选择“AI 翻译“ 菜单即可，也可以使用快捷键 `Ctrl+Alt+T`

如果使用的是 `JetBrains` 系统IDE的话，注意快捷键是否有冲突，可以自行设置快捷键。







