---
title: IDE 翻译插件 Ai Translate  新版本发布
date: 2024-10-17T15:35:55+08:00
type: post
toc: true
url: /posts/ai-translate-plugin
categories:
  - 其它
tags:
  - ide
  - vscode
  - jetbrains
  - plugin
---

IDE 插件 `AI Translate `经过几个小版本的迭代，目前已经支持市场上主流的 LLM 服务提供商，主要有以下几个服务提供商：

- OpenAI
- Anthropic
- DeepL
- 智谱 GLM
- 字节跳动
- DeepSeek
- Alibaba
- GitHub
- Gemini
- Ollama

除此之外，还有两个智能体服务商

- 阿里云百炼
-  扣子

基本已经满足了大多数开发者的基本需求了。

下载地址：

- JetBrains: https://plugins.jetbrains.com/plugin/25313-ai-translate
- VSCode: https://marketplace.visualstudio.com/items?itemName=cfanbo.ai-translate

特别是最近对 `Ollama`   的支持，允许用户使用本地的大模型提供服务，无需像其它几个 provider 一样，还需要注册会员和身份主证。只需要填写一个 URL 地址即可使用，非常的方便。

由于目前使用的 `prompt` 是统一的，因此不同 provider 的翻译效果可能不一样，后续有可能提供针对不同 provider 或 model 自定义prompt 的功能。



如果你想免费使用 `OpenAI`  模型的话，推荐注册 https://gh.io/models 服务，它提供了 `GPT_4o`、`GPT_4o mini`、`o1-mini`、`o1-preview` ，甚至还提供了两个 `Embedding model ` 。
