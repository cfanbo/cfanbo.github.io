---
title: Amazon EC2,Google App Engine,Microsoft Azure大比拼
author: admin
type: post
date: 2010-06-26T16:12:49+00:00
url: /archives/4049
IM_contentdowned:
 - 1
categories:
 - 前端设计

---
当Microsoft在PDC2008上携 [Azure平台](http://www.kuqin.com/system-analysis/20081030/25061.html) 驾 云而来时，天气预测注定将发生改变。将当前市场上分别来自Amazon、Google和Microsoft的三大主流产品作一比较会是一件非常有意思的事 儿。第一眼看上去的印象是它们彼此之间似乎实际上并不存在真正的相互竞争。

Ziff兄弟投资的副总裁Michael J. Miller [对云计算的三大角逐者进行了比较](http://blogs.pcmag.com/miller/2008/11/cloud_thinking_amazon_microsof.php)，这是他关于Amazon EC2的发现：

> 备受关注的云平台无疑就是[Amazon Web服务][1]了，它是多种工具 的集合，大部分都位于相当底层。其中又以Amazon弹性计算云(EC2)最为出众，这一Web服务能你将应用分配给任意多的“计算单元”。简单的说，一 个标准的单一实例包括了一个“虚拟核心”，1.7GB的内存，以及160GB的存储实例（只针对该对话的存储），价格为每小时10美分。在这个服务之上， 你可能会考虑使用该公司提供的“简单存储服务”(S3)，对于前50TB的数据，其价格是每GB每月15美分，之后价格递减，同时要收取一定的交易费用。 同时你也可能考虑使用该公司的“Simple DB”数据库，或者是用于存储消息的队列服务，包括一些附加的收费。
>
> Amazon平台的基本优势很简单：在你需要的时候，仅仅使用你所需要的存储量。

关于Google的App Engine，Michael如是说：

> Google的[App Engine][2]是新来 者。它仍处在免费的beta阶段，并且其工具集到目前为止仍有一定局限性。具我的理解，如果说Amazon给了你一台可以在其上安装许多软件的虚拟机的 话，Google更应该说是给了你一个基于Python语言，Django框架，Google的BigTalbe数据库/存储系统和Google文件系统 (GFS)的确定环境。目前，开发者可以免费获得500MB的存储，以及最多每月500万页面访问的计算能力，并且该公司对更活跃的站点宣布了收费策略。 举例来说，该公司表示开发者[每 CPU核心/小时可能会需要支付10到12美分][3]。
>
> 因为其与Google自己的操作环境联系非常紧密，所以对于了解这些框架的开发者而言，起步相对会容易一些。但一些开发者却选择了避开它，因为相比 Amazon的解决方案它显得过于局限了。

当谈到Microsoft的Azure时，Michael证实：

> 与Amazon Web服务相似，Azure实际上是由一个公共平台上的多种不同服务来组成的。…….NET服务是目前最受关注的部分，因为开发者初始时将用它来为平台作 开发。实际上，从我所参加的会议看来，将一个为.NET框架编写并用Visual Studio开发的应用将会很容易的迁移到“云”上。
>
> Azure的一大不同之处是在于，尽管Microsoft打算提供其自主的Azure托管服务，但该平台同样也是为运行于本地工作站和企业服务器而 设计的。这使得测试应用变得方便，也同样得以支持企业应用既能运行于公司的内部网也能运行于外部环境。

Michael将其比较概括为：

> 考虑这三个角逐者，你会发现它们每个都发挥着自己的强项。Amazon是市场的先行者，并利用因特网标准与开源平台打造了一个十分灵活的平台。 Google利用了其对于大型数据库的研究成果并借助其内部的开发方法创建了一个强大但略显局限的环境。而Microsoft凭借其在开发者方面的传统强 势与其宽泛的工具集提供了可能是最庞大的一系列服务。随着时间发展，我猜测我们会看到它们会开始互相靠拢－从Amazon引入Windows服务实例就是 一个预示。

道琼斯新闻电报的Jessica Hodgson就Microsoft参与这场游戏所产生的 [新财务等式](http://money.cnn.com/news/newsfeeds/articles/djf500/200811121627DOWJONESDJONLINE000806_FORTUNE5.htm) 撰文一篇。她引用Directions研究公司的分析员Matt Rosoff的言论：

> 我以为Microsoft想要做的是冻结这个市场。他们想让那些打算尝试按需产品的人们停下来，并等着看他们的产品将会如何。

根据Jessica的说法，对于云计算市场的价值有多种不同的意见：

> 虽然大家都同意云计算正在增长，关于它是否会取代本地授权软件，或是与其结伴同行的意见却各有千秋。Deutsche银行的分析师Tom Ernst表示，软件包模式的市场已达到饱和。不出五年，Ernst估计云计算服务将占据价值600亿美元的应用软件市场的半壁江山。与此相 反，Oracle公司的首席执行官(ORCL)Larry Ellison却嘲笑云计算是“胡言乱语”，并声称没什么公司可从中获益。
>
> Microsoft对于业务模型的承诺将会助长期待，推动那些不愿拥抱云计算的大公司采纳它。位于温哥华的Web开发公司Strangeloop Networks的共同创始人Richard Campbell表示，他的许多客户都在询问云计算，虽然出于对安全和可靠性的考虑很少真正转向它。

Jessica接着分析Microsoft的行动：

> Microsoft审慎的前进着，一边小心的保护着其既有的软件包模式特权，一边打量着云计算市场将如何发展。它采纳了一种混合的方式，因为它声称 多数消费者将会继续需要本地授权软件的产品，就是大部分分析师所支持的立场……。
>
> 如果Microsoft进展太慢，它将面临让诸如Google以及Amazon等革新者占据市场分额的风险。这些公司不存在Microsoft所面 对的利益两难境地，因为他们没有什么实体的软件特权需要去维护。
>
> “如果它(云计算)成为了主流业务的流行方式，很难看到他们如何去避免去蚕食自己桌面服务的销售，”Nickolas Carr表示，他曾任哈佛商业评论的编辑并出版了一本关于云计算的书。“这将是一场scale的游戏。”

**查看英文原文：**[Comparing Amazon’s EC2, Google’s App Engine and Microsoft’s Azure][4]
本文来自：

 [1]: http://aws.amazon.com/
 [2]: http://code.google.com/appengine/
 [3]: http://googleappengine.blogspot.com/2008/05/announcing-open-signups-expected.html
 [4]: http://www.infoq.com/news/2008/11/Comparing-EC2-App-Engine-Azure