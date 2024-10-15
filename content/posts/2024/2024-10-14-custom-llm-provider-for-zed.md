---
title: 为 zed IDE 设置自定义 LLM provider
date: 2024-10-14T15:24:01+08:00
type: post
toc: true
url: /posts/custom-llm-provider-for-zed
categories:
  - 其它
tags:
  - ide
  - zed
  - llm
---

 在`Zed` IDE中，默认只支持以下几种 providers :

- [Zed AI (Configured by default when signed in)](https://zed.dev/docs/assistant/configuration#zed-ai)
- [Anthropic](https://zed.dev/docs/assistant/configuration#anthropic)
- [GitHub Copilot Chat](https://zed.dev/docs/assistant/configuration#github-copilot-chat) [1](https://zed.dev/docs/assistant/configuration#1)
- [Google AI](https://zed.dev/docs/assistant/configuration#google-ai) [1](https://zed.dev/docs/assistant/configuration#1)
- [Ollama](https://zed.dev/docs/assistant/configuration#ollama)
- [OpenAI](https://zed.dev/docs/assistant/configuration#openai)

这对于国内开发者来说，由于政策原因，想使用起来可能借助一些科学上网的方法，这就有点麻烦了。另外国内几种大模型公司也都提供了一定的免费额度的 tokens，如果可以在Zed  里集成国内几家的大模型，也是一个不错的主意。

几个月前通过自定义`Endpoint` 方法，试图绕过官方不允放国内用户使用的问题，但没有成功。今天重新试了一下设置方法仍是无效，本想打算在官方仓库里开发一个自定义provider的功能，就是感觉着有点麻烦，但有点担心个人电脑过旧，编译是一个问题，于是重新在issue里找到一个解决办法 https://github.com/zed-industries/zed/pull/13276，就是设置起来有点麻烦。

本文将其记录设置方法整理一下。

# 配置 settings.json

首先配置 settings.json 里的 `assistant`  和 `language_models` 。

![image-20241015153632924](https://blogstatic.haohtml.com//uploads/2024/09/image-20241015153632924.png)

这里以 DeepSeek 为例，完整的内容如下

```json
{
    "assistant": {
    "default_model": {
      "provider": "openai",
      "model": "deepseek-chat"
    },
    "enabled": true,
    "provider": {
      "name": "openai",
      "default_model": {
        "custom": {
          "max_tokens": 32000,
          "name": "deepseek-chat"
        }
      },
      "available_models": [
        {
          "custom": {
            "name": "deepseek-chat",
            "max_tokens": 32000
          }
        },
        {
          "custom": {
            "name": "deepseek-code",
            "max_tokens": 32000
          }
        }
      ],
      "api_url": "https://api.deepseek.com/v1"
    },
    "version": "2"
  },
  "language_models": {
    "openai": {
      "version": "1",
      "api_url": "https://api.deepseek.com/v1",
      "available_models": [
        {
          "provider": "openai",
          "name": "deepseek-chat",
          "max_tokens": 32000
        },
        {
          "provider": "openai",
          "name": "deepseek-code",
          "max_tokens": 32000
        }
      ]
    }
  },
}
```

 上面的配置有些配置项看起来有些重复，这个没有办法，不要做任何删除操作。其中 `assistant.default_model` 里的值会在后面选择不同模型时动态的更新。

# 配置 OpenAI KEY

在 IDE 右下角点击 `Assistant Panel`

 ![image-20241015154203809](https://blogstatic.haohtml.com//uploads/2024/09/image-20241015154203809.png)

再点击右上角的 ”三“ 图标，选择  `Configuration` ，在最下方找到 `OpenAI` 配置项，将 DeepSeek 的 `KEY` 粘贴到填写框，然后按 **回车键** 确认配置，这时会在 `OpenAI` 配置项的右侧多出来一个 配置项，点击创建一个 对话下下文窗口

![image-20241015154552831](https://blogstatic.haohtml.com//uploads/2024/09/image-20241015154552831.png)

这时会看到一个新的对话窗口，然后在右上角选择一个我们刚刚添加的模型

<img src="https://blogstatic.haohtml.com//uploads/2024/09/Screen%20Shot%202024-10-15%20at%2015.49.06.png" alt="Screen Shot 2024-10-15 at 15.49.06" style="zoom:50%;" />

这时我们提问一个问题，并按 `command+enter` 组合键，可以看到它的回答，表示我们的配置正常。

<img src="https://blogstatic.haohtml.com//uploads/2024/09/image-20241015155727117.png" alt="image-20241015155727117" style="zoom:50%;" />



> 第一个问题，返回的内容总是与问题完全一样，后面的就正常了，不清楚什么原因。

至此配置结束。

# 说明

1. 上面的配置有点复杂，其中有些配置项看着有点重复，不过确实满足了我们的需要，况且对于配置操作一般就一次，也就无所谓了
2. 这里以 DeepSeek 为例，我们可以切换成阿里的 Qwen 模型，或 Doubao 模型
