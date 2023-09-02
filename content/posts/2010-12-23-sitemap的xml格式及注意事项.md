---
title: Sitemap的XML格式及注意事项
author: admin
type: post
date: 2010-12-23T08:53:41+00:00
url: /archives/7117
IM_contentdowned:
 - 1
categories:
 - 前端设计

---
这篇文章介绍的比较全的:

此文档介绍适用于 Sitemap 协议的 XML 架构。

Sitemaps 协议格式由 XML 标记组成。Sitemap 的所有数据数值应为实体转义过的。文件本身应为 UTF-8 编码。

Sitemap 必须：

 * 以 `< [urlset](http://www.sitemaps.org/zh_CN/protocol.php#urlsetdef) >` 开始标记作为开始，以 `` 结束标记作为结束。
 * 在 `` 标记中指定命名空间（协议标准）。
 * 每个网址包含一个`< [url](http://www.sitemaps.org/zh_CN/protocol.php#urldef) >` 条目作为 XML 父标记。
 * 在每个 `` 父标记中包含一个 `< [loc](http://www.sitemaps.org/zh_CN/protocol.php#locdef) >` 子标记条目。

其他所有标记均为可选，搜索引擎不同，对可选标记的支持也各不相同。有关详情，请参阅各个搜索引擎的文档。

而且，Sitemap 中的所有网址都必须来自于同一个主机，如 www.example.com 或 store.example.com。有关详细信息，请参阅[Sitemap 文件位置][1] 。

## XML 标记定义

以下对可用 XML 标记进行说明。

 属性

 说明
 `<urlset>`
 必填

 压缩此文件并提供当前协议标准作为参考。
 `<url>`
 必填

 每个网址条目的父标记。剩余标记为此标记的子标记。
 `<loc>`
 必填

 该页的网址。如果您的网络服务器需要网址的话，此网址应以协议开始（例如：http）并以斜杠结尾。该值必须少于 2,048 个字符。
 `<lastmod>`
 可选

 该文件上次修改的日期。此日期应采用 [W3C Datetime](http://www.w3.org/TR/NOTE-datetime) 格式。如果需要，此格式允许省略时间部分，并使用 YYYY-MM-DD。

请注意，此标记不同于服务器可返回的 If-Modified-Since (304) 标头，搜索引擎可能会以不同的方式使用这两个来源的信息。

`<changefreq>`
 可选

 页面可能发生更改的频率。此值为搜索引擎提供一般性信息，可能与搜索引擎抓取页面的频率不完全相关。有效值为：

- always

- hourly

- daily

- weekly

- mothly

- yearly

- never


“always”值应当用于描述随每次访问而改变的文档。而“never”值则应当用于描述存档的网址。


请注意，抓取工具会将此标记的值视为 _提示_ 而不是命令。尽管搜索引擎抓取工具在做 决定时会考虑此信息，但对于标记为“hourly”页面的抓取频率可能低于每小时一次，而对于标记为“yearly”页面的抓取频率可能高于每年一次。抓 取工具也可能会定期抓取标记为“never”的网页，以便能够处理对这些网页的未预期更改。

`<priority>`
 可选

 此网址的优先级是相对于您网站上其他网址的优先级而言的。有效值范围从 0.0 到 1.0。该值不会影响您的网页与其他网站上网页的比较结果，而只是告知搜索引擎您认为哪些网页对抓取工具来说最为重要。

一个网页的默认优先级为 0.5。


请注意，为网页指定的优先级并不会影响网址在搜索引擎结果页上的排名。搜索引擎在同一网站上选择不同网址时会使用此信息，因此，您可以使用此标记增加最重要的网页在搜索索引中显示的可能性。


另请注意，为网站中的所有网址都指定高优先级并不会带来什么好处。因为优先级是相对的，只用于在您网站的网址之间进行选择。

## 实体转义

Sitemap 文件必须以 UTF-8 编码（通常在保存文件时可以这么做）。对于所有的 XML 文件，任何数据数值（包括网址）都应对下表中列出的字符使用实体转义码。


 字符

 转义码

 & 符号

 &
 `&`
 单引号

 ‘
 `'`
 双引号

 “
 `"`
 大于

 >
 `>`
 小于

 <
 `<`

此外，所有网址（包括 Sitemap 的网址）都必须经过网址转义并编码，以便它们所在网络服务器可以进行读取。不过，如果您使用任何类型的脚本、工具或日志文件来生成网址（除手动输入之外的 任何方法），通常系统已经替您完成了这部分工作。请仔细检查，确保网址符合 [RFC-3986](http://asg.web.cmu.edu/rfc/rfc3986.html) URI 标准、 [RFC-3987](http://www.ietf.org/rfc/rfc3987.txt) IRI 标准，以及 [XML 标准](http://www.w3.org/TR/REC-xml/) 。


### XML Sitemap 示例

下例显示了一个 XML 格式的 Sitemap。示例中的 Sitemap 包含少量的网址，每个网址都使用不同的一组可选参数。


> ```
> <?xml version="1.0" encoding="UTF-8"?>
> <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
>    <url>
>       <loc>http://www.example.com/</loc>
>       <lastmod>2005-01-01</lastmod>
>       <changefreq>monthly</changefreq>
>       <priority>0.8</priority>
>    </url>
>    <url>
>       <loc>http://www.example.com/catalog?item=12&desc=vacation_hawaii</loc>
>       <changefreq>weekly</changefreq>
>    </url>
>    <url>
>       <loc>http://www.example.com/catalog?item=73&desc=vacation_new_zealand</loc>
>       <lastmod>2004-12-23</lastmod>
>       <changefreq>weekly</changefreq>
>    </url>
>    <url>
>       <loc>http://www.example.com/catalog?item=74&desc=vacation_newfoundland</loc>
>       <lastmod>2004-12-23T18:00:15+00:00</lastmod>
>       <priority>0.3</priority>
>    </url>
>    <url>
>       <loc>http://www.example.com/catalog?item=83&desc=vacation_usa</loc>
>       <lastmod>2004-11-23</lastmod>
>    </url>
> </urlset>
>
> ```

## 使用 Sitemap 索引文件（对多个 Sitemap 文件进行分组）

您可以提供多个 Sitemap 文件，但每个 Sitemap 文件包含的网址不得超过 **50,000** 个，并且文件不得超过 **10MB**（10,485,760 字节）。如果您愿意，可以使用 gzip 压缩 Sitemap 文件，以减少带宽要求；但是解压缩后的 Sitemap 文件不得超过 10MB。如果要列出 50,000 个以上的网址，您需要创建多个 Sitemap 文件。


如果您确实提供多个 Sitemap，则应当在 Sitemap 索引文件中列出每个 Sitemap 文件。Sitemap 索引文件中最多可列出 50,000 个 Sitemap，文件不得超过 10MB（10,485,760 字节），并且是可以压缩的。您可以具有多个 Sitemap 索引文件。Sitemap 索引文件的 XML 格式与 Sitemap 文件的 XML 格式非常相似。


Sitemap 索引文件必须：


- 以 `<sitemapindex>` 开始标记作为开始，以 `</sitemapindex>` 结束标记作为结束。

- 每个 Sitemap 包含一个`<sitemap>` 条目作为 XML 父标记。

- 每个 `<sitemap>` 父标记包含一个 `<loc>` 子标记条目。


可选的 `<lastmod>` 标记同样适用于 Sitemap 索引文件。


**注意：** Sitemap 索引文件只能指定与其位于同一网站的 Sitemap。例如，http://www.yoursite.com/sitemap_index.xml 可包含 http://www.yoursite.com 上的Sitemap，但不能包含 http://www.example.com 或 http://yourhost.yoursite.com 上的 Sitemap。


与 Sitemap 一样，Sitemap 索引文件也必须为 UTF-8 编码。


### XML Sitemap 索引示例

下例显示包含两个 Sitemap 的 Sitemap 索引文件：


> ```
> <?xml version="1.0" encoding="UTF-8"?>
> <sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
>    <sitemap>
>       <loc>http://www.example.com/sitemap1.xml.gz</loc>
>       <lastmod>2004-10-01T18:23:17+00:00</lastmod>
>    </sitemap>
>    <sitemap>
>       <loc>http://www.example.com/sitemap2.xml.gz</loc>
>       <lastmod>2005-01-01</lastmod>
>    </sitemap>
> </sitemapindex>
>
> ```

**注意：** 与 XML 文件中的所有值一样，Sitemap 网址必须经过 [实体转义](http://www.sitemaps.org/zh_CN/protocol.php#escaping) 。


### Sitemap 索引 XML 标记定义

 属性

 说明
 `<sitemapindex>`
 必填

 压缩文件中所有 Sitemap 的相关信息。
 `<sitemap>`
 必填

 压缩个别 Sitemap 的相关信息。
 `<loc>`
 必填

 识别 Sitemap 的位置。

此位置可以为 Sitemap、Atom 文件、RSS 文件或简单的文本文件。

`<lastmod>`
 可选

 识别相对 Sitemap 文件的修改时间。它与该 Sitemap 中列出的任一网页的更改时间不相符。lastmod 标记的值应采用 [W3C 日期时间](http://www.w3.org/TR/NOTE-datetime) 格式。

通过提供最近修改的时间戳，您可以让搜索引擎抓取工具只检索索引中的 Sitemap 子集，也就是说，抓取工具只检索某个特定日期之后修改的 Sitemap。通过这一递增的 Sitemap 提取机制，可以快速发现超大型网站上的新网址。

您可以提供纯文本文件，其中每行包含一个网址。此文本文件需要遵循以下指南：


- 文本文件每行都必须有一个网址。网址中不能有换行。

- 您必须指定完整的网址，包括 http。

- 每个文本文件最多可包含 50,000 个网址，并且不得超过 10MB（10,485,760 字节）。如果网站所包含的网址超过 50,000 个，则可以将列表分割成多个文本文件，然后分别添加每个文件。

- 文本文件需使用 UTF-8 编码。在保存文件时您可指明此项（例如，在记事本中，此项会在“另存为”对话框中的编码菜单中列出）。

- 文本文件不应包含网址列表以外的任何信息。

- 此文本文件不应包含任何标题或注脚信息。

- 如果愿意，您可以使用 gzip 压缩 Sitemap 文本文件，以减少带宽要求。

- 您可以随意为此文本文件命名。请检查并确保您的网址符合 [RFC-3986](http://www.ietf.org/rfc/rfc3986.txt) 标准中的 URI 规定和 [RFC-3987](http://www.ietf.org/rfc/rfc3987.txt) 标准中的 IRI 规定。

- 您应该将文本文件上传至您希望搜索引擎抓取的最高级别的目录，并确保在文本文件中未列出位于更高级别目录的网址。


## Sitemap 文件位置

Sitemap 文件的位置决定该 Sitemap 中可以包含的网址组。位于 http://example.com/catalog/sitemap.xml 的 Sitemap 文件可以包含任何以 http://example.com/catalog/ 开头的网址，但不能包含以 http://example.com/images/ 开头的网址。


## 验证您的 Sitemap

下列 XML 架构定义可以出现在 Sitemap 文件中的元素和属性。可从以下链接下载此架构：


**对于 Sitemap：** [http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd](http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd)

**对于 Sitemap 索引文件：** [http://www.sitemaps.org/schemas/sitemap/0.9/siteindex.xsd](http://www.sitemaps.org/schemas/sitemap/0.9/siteindex.xsd)

有多种工具可帮助您根据此架构来验证您的 Sitemap 结构。在下面的每一个位置您都可以找到 XML 相关的工具列表：


[http://www.w3.org/XML/Schema#Tools](http://www.w3.org/XML/Schema#Tools)

[http://www.xml.com/pub/a/2000/12/13/schematools.html](http://www.xml.com/pub/a/2000/12/13/schematools.html)

Sitemap 协议可让您告知搜索引擎您希望将那些内容编入索引。要告知搜索引擎您要编入索引的内容，请使用 robots.txt 文件或 robots 元标记。有关如何从搜索引擎中排除内容的详情，请参阅 [robotstxt.org](http://www.sitemaps.org/zh_CN/protocol.php) 。


**总结:**

1、文件随小但这是一套完整的方法和规范，他是对外开放的窗口。


2、灵活掌握其精髓，搭建网站的map你会对网站了解深入骨髓。


 [1]: http://www.sitemaps.org/zh_CN/protocol.php#location