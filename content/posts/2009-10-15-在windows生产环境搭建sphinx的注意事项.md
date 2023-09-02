---
title: 在windows生产环境搭建sphinx的注意事项
author: admin
type: post
date: 2009-10-15T08:05:55+00:00
excerpt: |
 1、以服务的方式运行sphinx

 在开发环境中，只要执行”rake ultrasphinx:daemon:start“，就可以启动一台sphinx服务器。但如果在生产环境还能这么做么？把sphinx安装为服务无疑是个靠谱的办法，这样它可以像mongrel、apache一样随系统启动。sphinx自带了安装为windows服务的命令：

 searchd –install –config xxxx.conf

 不妨把这个加入到rake命令中，于是我hack了一下ultrasphinx插件的任务列表，加入了一个”rake ultrasphinx:daemon:install“命令。名为ultrasphinx.rake的文件我将稍后提供。

 既然把sphinx安装为服务，相应的start和stop命令，也需要改改，改为使用”net start/stop searchd”。
url: /archives/2529
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - coreseek
 - Sphinx

---
**1、以服务的方式运行sphinx**

在开发环境中，只要执行”_rake ultrasphinx:daemon:start_“，就可以启动一台sphinx服务器。但如果在生产环境还能这么做么？把sphinx安装为服务无疑是个靠谱的办法，这样它可以像mongrel、apache一样随系统启动。sphinx自带了安装为windows服务的命令：

> searchd –-install -–config xxxx.conf
>
> 相应的删除服务命令为：
>
> searchd –delete

不妨把这个加入到rake命令中，于是我hack了一下ultrasphinx插件的任务列表，加入了一个”_rake ultrasphinx:daemon:install_“命令。名为ultrasphinx.rake的文件我将稍后提供。
如果在启用服务的时候提示”发生系统错误1067″的话,则需要在安装服务的时候指定配置文件的路径,参考:[sphinx在windows下无法启动的解决办法](http://blog.haohtml.com/index.php/archives/2593) 如: d:\csft3.1\bin>searchd –install –config d:\csft3.1\bin\www.conf

既然把sphinx安装为服务，相应的start和stop命令，也需要改改，改为使用”net start/stop searchd”。

**2、主索引和增量索引**（delta）

对于大量的数据，经常重建主索引很不现实。我的项目中几十万条数据，重建一次索引需要将近10个小时，所以打算以每天多次增量索引+一次合并索引的形式去做。

但javaeye论坛有人 [发帖](http://www.javaeye.com/topic/203122 "关于sphinx和ultrasphinx") 说， 在windows上要编制索引必须先停止sphinx服务器，因为searchd服务会把索引文件挂住。实际上并非如此：不论在windows还是在 linux，sphinx服务器都会锁住索引文件，但只要在编制索引时加上–rotate参数，就可以不停止sphinx服务器来编制索引。sphinx 的 [官方文档](http://www.sphinxsearch.com/docs/current.html "sphinx官方文档") 解释：

> `--rotate` creates a second index, parallel to the first (in the same place, simply including `.new` in the filenames). Once complete, `indexer` notifies `searchd` via sending the `SIGHUP` signal, and `searchd` will attempt to rename the indexes (renaming the existing ones to include `.old` and renaming the `.new` to replace them), and then start serving from the newer files. Depending on the setting of [seamless_rotate][1], there may be a slight delay in being able to search the newer indexes.

大意是如果编制索引时加上–totate参数，将生成一份新的索引文件，然后向运行中的searchd发一个信号，让它去把新生成索引更名，然后在新的索引文件上提供服务，文档说切换的时候会有”a slight delay”，我想应该是可以接受的。

那么，回到开头，为什么javaeye会有人认为必须停止服务器呢？其实问题源自ultrasphinx.rake文件中 的”ultrasphinx\_daemon\_running?”这个函数。这个函数是用来检测sphinx服务器是否在运行的，但在windows下没法 用，所以每次都返回false，于是在进行index的时候，就不会加上rotate参数。我又hack了一下，直接让这个函数返回true了，但这就得 人工来保证sphinx一直在运行。

**3、设置任务计划**

我采用了主索引+增量索引的方式，增量索引为24小时内修改过的内容，增量索引可以每十分钟就运行一次，然后每天0点与主索引进行合并。在linux上有crontab，在windows上，只能用任务计划了。

写一个批处理文件(.bat)：

> cd your\_app\_direcrory
> echo “Indexing delta…”
> rake ultrasphinx:index:delta
> echo “Merging to main index…”
> rake ultrasphinx:index:merge
> echo “Daily task comleted.”

这是将增量索引合并到主索引的批处理文件。新建一个任务计划，选择运行此文件，然后设置为每天0点运行。单独做增量索引的批处理文件大家可以自己写。

经过这么一番折腾，sphinx已经可以在生产环境下用了。可能还需要一个监控的东西，不过我暂时还没想好。

我hack过的ultrasphinx.rake文件可以在此处下载：[ultrasphinx.rake][2]

如果大家要买独立服务器，还是建议买linux的，我当年一念之差，让现在徒增很多烦恼。

 [1]: http://www.sphinxsearch.com/docs/current.html#conf-seamless-rotate "9.4.11. seamless_rotate"
 [2]: http://www.blogkid.net/wp-content/uploads/2009/02/ultrasphinx.rar