---
title: terraform 中的 provider
author: admin
type: post
date: 2023-08-31T10:15:05+00:00
url: /archives/35179
categories:
 - 程序开发
tags:
 - terraform

---
本文主要对 [terraform][1] 中的 `Providers` 进行介绍，让刚刚接触 `terraform` 的用户对其有一个大概的了解，以下内容翻译自：https://developer.hashicorp.com/terraform/language/providers

# 什么是 Providers 

> **实践:** Try the [Perform CRUD Operations with Providers](https://developer.hashicorp.com/terraform/tutorials/configuration-language/provider-use?utm_source=WEBSITE&utm_medium=WEB_IO&utm_offer=ARTICLE_PAGE&utm_content=DOCS) tutorial.

Terraform 依赖于称为提供商的插件来与`云提供商`、`SaaS 提供商`和 `其他 API` 进行交互。 Terraform 配置必须声明它们需要哪些 `providers`，以便 `Terraform` 可以 **安装** 和 **使用** 它们。此外，某些提供商在使用之前需要进行配置（例如 `端点 URL` 或 `云区域`）。

# Providers 能做什么 

每一个 Providers 都会有一组 Terraform 可以管理的 [resource types][2] 和或 [data sources][3]。如我们经常使用的 [docker provider](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs), 它提供了一些 `Resources` 和 `Data sources`，使用的时候只需要查看一下 provider 提供的有哪些配置直接 。

每个 `resouce type` 都由 Provider 实现；如果没有Provider的话，Terraform 就无法管理任何类型的基础设施。 大多数提供商配置特定的基础设施平台（云或自托管）。提供商还可以提供本地实用程序来执行诸如为唯一资源名称生成随机数之类的任务。

# Providers 来自哪里 

providers 与 Terraform 本身分开分发，每个提供程序都有自己的发布节奏和版本号。 [Terraform Registry][4] 是公开可用的 Terraform providers，并托管大多数主要基础设施平台的提供程序。

# Provider 文档 

每个提供者都有自己的文档，描述其资源类型及其参数。

[Terraform Registry][4] 包含由 HashiCorp、第三方供应商和我们的 Terraform 社区开发的各种提供商的文档。使用提供商标题中的“文档”链接来浏览其文档。 注册表中的提供者文档已进行版本控制；您可以使用标题中的版本菜单来更改您正在查看的版本。 有关编写、生成和预览提供程序文档的详细信息，请参阅 [provider publishing documentation][5].。

# 如何使用 Provider 

providers 与 Terraform 本身分开发布并拥有自己的版本号。在生产中，我们建议在配置的提供程序要求块中限制可接受的提供程序版本，以确保 `terraform init` 不会安装与配置不兼容的较新版本的提供程序。

要使用来自给定提供程序的资源，您需要在配置中包含一些有关它的信息。详情请参阅以下页面：

 * [Provider Requirements][6] 记录了如何声明提供程序以便 Terraform 可以安装它们。
 * [Provider Configuration][7] 记录了如何配置提供程序的设置。
 * [Dependency Lock File][8] 记录了一个可以包含在配置中的附加 HCL 文件，该文件告诉 Terraform 始终使用一组特定的提供程序版本。

# Provider 安装 

 * Terraform Cloud 和 Terraform Enterprise 在每次运行时都会安装提供程序。
 * Terraform CLI 在 [初始化工作目录][9] 时查找并安装提供程序。它可以自动从 Terraform 注册表下载提供程序，或从本地镜像或缓存加载它们。如果您使用持久工作目录，则每当更改配置的提供程序时都必须重新初始化。 为了节省时间和带宽，Terraform CLI 支持可选的插件缓存。您可以使用 [the CLI configuration file][10] 中的`plugin_cache_dir` 设置启用缓存。

为了确保Terraform始终为给定的配置安装相同的提供程序版本，您可以使用Terraform CLI创建 [依赖项锁文件][8] ，并将其与配置一起提交到版本控制。如果存在锁定文件，Terraform Cloud、CLI和Enterprise在安装提供程序时都会遵守该文件。

> **实践:** Try the [Lock and Upgrade Provider Versions](https://developer.hashicorp.com/terraform/tutorials/configuration-language/provider-versioning?utm_source=WEBSITE&utm_medium=WEB_IO&utm_offer=ARTICLE_PAGE&utm_content=DOCS) tutorial.

# 如何查找 Provider 

要查找您使用的基础设施平台的提供程序，请浏览 [Terraform Registry][4] 的提供程序部分。

注册中心上的一些提供程序是由 HashiCorp 开发和发布的，一些是由平台维护者发布的，还有一些是由用户和志愿者发布的。提供商列表使用以下徽章来指示谁开发和维护给定提供商。

| Tier | Description | Namespace |
| --------- | ---------------------------------------------------------------------------------- | -------------------------------- |
| Official | 官方provider 由 HashiCorp 拥有和维护 | `hashicorp` 官方 |
| Partner | 合作伙伴提供商由第三方公司根据自己的 API 编写、维护、验证和发布。要获得合作伙伴提供商徽章，合作伙伴必须参加 [HashiCorp 技术合作伙伴计划][11]。 | 第三方机构, 如 `mongodb/mongodbatlas` |
| Community | 社区providers由 Terraform 社区的个人维护者、维护者小组或其他成员发布到 Terraform Registry。 | 维护者的个人或组织帐户, 如 `DeviaVir/gsuite` |
| Archived | 存档的提供商是不再由 HashiCorp 或社区维护的官方或合作伙伴提供商。如果 API 被弃用或兴趣较低，则可能会发生这种情况。 | `hashicorp` 或第三方 |

# 如何开发Provider 

提供程序是使用 Terraform Plugin SDK 用 Go 编写的。有关开发 Provider 的更多信息，请参阅：

 * The [Plugin Development][12] documentation
 * The [Call APIs with Terraform Providers][13] tutorials

[1]: https://www.terraform.io/
[2]: https://developer.hashicorp.com/terraform/language/resources
[3]: https://developer.hashicorp.com/terraform/language/data-sources
[4]: https://registry.terraform.io/browse/providers
[5]: https://developer.hashicorp.com/terraform/registry/providers/docs
[6]: https://developer.hashicorp.com/terraform/language/providers/requirements
[7]: https://developer.hashicorp.com/terraform/language/providers/configuration
[8]: https://developer.hashicorp.com/terraform/language/files/dependency-lock
[9]: https://developer.hashicorp.com/terraform/cli/init
[10]: https://developer.hashicorp.com/terraform/cli/config/config-file
[11]: https://www.hashicorp.com/ecosystem/become-a-partner/
[12]: https://developer.hashicorp.com/terraform/plugin
[13]: https://developer.hashicorp.com/terraform/tutorials/providers-plugin-framework?utm_source=WEBSITE&utm_medium=WEB_IO&utm_offer=ARTICLE_PAGE&utm_content=DOCS