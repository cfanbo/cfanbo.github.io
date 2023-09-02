---
title: 日志文件分析工具—AWStats在IIS中的配置
author: admin
type: post
date: 2007-07-19T18:19:24+00:00
url: /archives/42
IM_contentdowned:
 - 1
categories:
 - 服务器

---

[AWStats](http://sourceforge.net/projects/awstats/) 是sourceforge.net上很有名的Web/Mail/FTP服务器日志文件分析工具，可以运行在windows系统上分析IIS日志文件，本文讲的是AWStats在windows下的安装及配置。


运行环境说明：


操作系统Microsoft Windows Server 2003 SP2简体中文企业版

Web服务器IIS 6.0

Perl：ActivePerl 5.8.8.820

AWStats 6.7


#### 一、IIS配置

1.启用IIS日志记录：打开windows运行对话框(Windows+R)，输入inetmgr,打开Internet 信息服务（IIS）管理器控制台界面,在控制台左边“网站”项目上点击鼠标右键，打开“网站属性”设置窗口，在“网站”标签中，将“启用日志记录”前的复选框选中，再点击“应用”按钮，使设置生效。


2.日志格式设置：活动日志格式选择“W3C扩展日志文件格式,再点击“属性”按钮，进入日志记录属性配置界面，新日志计划选择“每天”，勾选“文件名和创建使用当地时间”，日志文件目录默认为C:WINDOWSsystem32LogFiles，由于Web服务器的长期运行会使日志文件会变得非常大，因此建议不要将日志文件存放在默认的目录中，应该保存到特定的目录中，确保磁盘空间充足，并做好备份和维护工作，如果您不在意以前的数据丢失与否或者您仅仅是在本机做测试，就没必要更改默认目录了，在本项目中，日志文件保存路径设定为：D:siteLogFiles，在设置了日志文件目录后，日志文件其实并不是直接保存在该目录下，系统会在设定的目录中根据需要建立不同的子目录，分别保存不同的日志文件，在下图中，日志文件名：W3SVCXexyymmdd.log是站点的日志的实际存储路径。其中W3SVCX中的X表示不同的WEB站点的标识符，为数字，组合后目录名称为W3SVC1，W3SVC2等，文件名为字母ex加上年月日。实际日志文件名例如：W3SVC1ex070712.log。


3.日志记录属性高级设置：在设置了新日志计划以及日志文件命名规则以后，还需要对日志文件包含的内容进行配置，选择“高级”标签，进入“扩展日志选项”的界面。 勾选以下12个项目，项目必须完全一致。


- date(日期)

- time(时间)

- c-ip(客户端IP地址)

- cs-username(用户名)

- cs-method(方法)

- cs-uri-stem(URI资源)

- cs-uri-query(URI查询)

- sc-status(协议状态)

- sc-bytes(发送的字节数)

- cs-version(协议版本)

- cs(User-Agent)(用户代理)

- cs(Refer)(引用站点)


4.应用配置方案：您可以对服务器上的所有站点进行相同的配置，也可以分别对每个站点进行不同的配置！停止IIS,删除原有的日志文件，然后启动IIS,并访问一次站点，即可生成新格式下的IIS日志记录文件。


#### 二、 Perl语言运行环境

AWStats软件是使用动态语言Perl开发的应用程序，因此服务器上必须具有Perl运行环境，我们这里使用ActivePerl 5.8.x软件，它的安装配置比较简单。


1.下载软件：


- 版本：ActivePerl-5.8.8.820

- 文件名：ActivePerl-5.8.8.820-MSWin32-x86-274739.zip

- 官方网站： [http://www.activestate.com/](http://www.activestate.com/)
- 完整路径： [http://www.activestate.com/store/download.aspx?prdGUID=81fbce82-6bd5-49bc-a915-08d58c2648ca](http://www.activestate.com/store/download.aspx?prdGUID=81fbce82-6bd5-49bc-a915-08d58c2648ca "进入各个版本的Perl解释器下载页面")

2.安装ActivePerl ：


- 安装用户必须是Administrators组的用户，否则安装不能成功或者不完整，导致软件不能正常运行。

- Perl环境变量：如果已经存在PERLLIB, PERL5LIB 或者 PERL5OPT这几个环境变量，必须在安装ActivePerl之前使它们无效，否则这些变量会在安装处理过程中导致Perl模块的版本不兼容问题。

- 在安装ActivePerl之前，请先停止Web服务器，安装完毕以后再启动WEB服务器。

- MSI安装包:双击安装文件，直接进行安装。ZIP安装包，直接拷贝到想要放置的目录。如果已经安装过其他版本，请首先卸载，然后再安装新版本。不要直接在以前版本上安装。 本方案中将ActivePerl安装到C:Perl目录下。


3.配置：


- **启用Perl服务扩展:** 安装ActivePerl以后，还需要配置WEB服务扩展，使得IIS能够支持perl脚本，打开IIS, 选择左边窗口目录树中的“Web 服务扩展”项，则右边窗口中显示出系统已经安装的服务扩展及状态（默认多为禁止），对于ActivePerl 5.8.x在Windows Server 2003上的默认安装，我们可以看到以下两个项目： Perl CGI Extension 禁止、Perl ISAPI Extension 禁止。默认情况下它们处于禁止状态，需要将它们的状态改变为“允许”，请分别选择这两个服务扩展，点击“允许”按钮，启用它们，使得perl脚本程序可以被IIS执行。

- **添加Perl服务扩展：** 如果没有以上两项服务扩展(使用ZIP包安装或者安装其他版本的ActivePerl)，那么我们需要手工添加这两项配置信息。 选择管理控制台左边窗口上的“Web 服务扩展”，点击鼠标右键，选择“添加一个新的Web服务扩展（A）…”，在“扩展名”中输入一个完整说明名称(不是文件的扩展名)，例如：Perl CGI Extension，点击“添加”按钮，在“文件路径”中输入进行扩展处理的文件的完整路径以及参数，例如：C:PerlbinPerl.exe  “%s”  %s ，并勾选“设置扩展状态为允许”

- **配置IIS脚本程序映射：** ActivePerl安装完毕以后，应该会在IIS主目录配置中添加脚本程序映射，如果没有，那么在你希望配置Perl脚本的虚拟目录、应用程序或网站上点击鼠标右键，选择“属性”，打开网站属性界面，选择“主目录”标签，点击“配置”按钮，（如果配置按钮不可用，点击创建按钮，创建一个默认的配置方案），进行应用程序配置。如果在应用程序扩展列表中，没有发现.pl的扩展名项，那么点击“添加”按钮，进行应用程序扩展名映射。在“可执行文件”中输入和前面添加Web服务扩展中相同的命令行C:PerlbinPerl.exe  “%s”  %s ，在“扩展名”中输入.pl，在“动作”中输入：GET,HEAD,POST


#### 三、  AWStats的安装与配置

1、下载AWStats


- 版本：6.7稳定版

- 文件名：awstats-6.7.zip

- 官网： [http://awstats.sourceforge.net/](http://awstats.sourceforge.net/ "连接到awstats-官方网站")
- 下载地址： [http://sourceforge.net/projects/awstats/](http://sourceforge.net/projects/awstats/ "下载AWStats"),也可以直接 [下载AWStats6.7Zip版本](http://nchc.dl.sourceforge.net/sourceforge/awstats/awstats-6.7.zip)

2.安装AWStats


- 文件安装在D:AWStats

- 安装目录说明：
  - x:aswtatsdocs软件的相关文档）

  - x:aswtatstools(软件的相关工具)

  - x:aswtatswwwroot(Web日志分析统计程序及相关文件)

  - x:aswtatswwwrootcgi-bin(分析结果主显示程序 )

  - x:aswtatswwwrootclasses(软件的相关类文件)

  - x:aswtatswwwrootcss(样式表)

  - x:aswtatswwwrooticon(该软件所用图片)

  - x:aswtatswwwrootjs(javascript脚本)

3、IIS相关配置


- 映射虚拟目录AWStats：在AWSTATS 软件安装以后，需要将主程序所在的路径wwwroot 映射成网站的一个虚拟目录，当然，也可以将该目录下的所有子目录复制到你的网站中，但是这样对于管理很不方便，而且权限设置也存在安全问题，因此，我们选择将wwwroot映射成虚拟目录。启动“Internet 信息服务管理器”，选择你的网站，如“默认网站”，右键，“新建虚拟目录”，虚拟目录名称我们命名为AWStats，路径为安装程序所在的路径，如 x:awstatswwwroot。

- 虚拟目录相关属性设置：虚拟目录建立完毕以后，还需要对虚拟目录的属性进行设置。选择建立的虚拟目录，点击鼠标右键，选择“属性”,将“记录访问”、“索引资源”前的复选框选项去掉，表示对于该虚拟目录的访问不记录进访问日志中，并且全文索引不对该虚拟目录进行索引。 将“执行权限”选择为“脚本和可执行文件”，该选项必须如此，否则不能显示统计信息。


4.AWStats运行配置


- 创建网站的AWStats配置信息：网站配置文件存放在x:AWStatswwwrootcgi-bin目录下，默认文件名称为：awstats.model.conf，我们将它复制成一个新配置文件：awstats.myvirtualhostname.conf，其中myvirtualhostname是网站的域名或者IP，因此，我们创建了配置文件：awstats.www.mysite.com.cn.conf。

- 编辑配置文件中相应配置信息：
  - **LogType：** 日志文件的类型，W—web日志、M—email日志、F—FTP日志。LogType=W

  - **LogFile：** 日志文件存储的路径或位置，以及文件名，在前面我们进行IIS配置时，将日志文件存储在D:siteLogFiles目录下，我们查看得到MYSITE网站的标识为W3SVC1(不同的安装会导致此标识不同，因此需要根据实际查看得到)，日志文件命名为exYYMMDD.log。LogFile=”D:siteLogFilesW3SVC1ex%YY-0%MM-0%DD-0.log”

  - **LogFormat：** 日志文件的格式，它必须与log文件中的格式完全一致(查看log文件中的#Fields字段)，栏目多少与顺序也必须一致，因此，IIS日志记录格式必须按照前面介绍的配置进行。LogFormat=”date time cs-method cs-uri-stem cs-uri-query cs-username c-ip cs-version cs(User-Agent) cs(Referer) sc-status sc-bytes”

  - **DirData** **：** 由于网站访问的日志有可能极大，每次统计可能需要耗时几小时，因此统计信息不是即时的，而是以天为单位，因此分析得到的统计数据需要存放到专用的数据文件中。注意，必须首先手工建立该数据存放目录。DirData=”D:AWStatswwwrootcgi-bindata”

  - **CreateDirDataIfNotExists：** 如果分析结果数据保存路径不存在，是否创建该路径，1表示立即创建。CreateDirDataIfNotExists=0

  - **DirIcons：** AWStats需要使用的图片文件的存储路径，由于我们将x:awstatswwwroot设置为虚拟路径，因此，在此处

     DirIcons=”../icon”

  - **SiteDomain：** 网站完整域名，SiteDomain=”www.mysite.com.cn”

  - **AllowFullYearView：** 是否运行以年为单位分析日志。AWStats默认是以月为单位分析日志数据，如果需要以年为单位进行查看分析，需要按照下面设置。AllowFullYearView=3

  - **DefaultFile：** 网站主页名称。DefaultFile=”index.asp”

  - **Lang：** 网站分析统计结果界面的语言，cn表示中文。Lang=”cn”

  - **NotPageList：** NotPageList=”css js class gif jpg jpeg png bmp ico swf zip pdf xml htc”
- 插件安装以及设置：
  - **Plugin: TimeZone：** 由于采用W3C标准格式记录日志，日期时间记录的是格林威治时间，因此需要根据时区进行调整，否则得到的统计结果就是错误的。 该插件的配置在网站配置文件中（与上面的参数配置相同）。LoadPlugin=”timezone +8″

  - **IP地址插件：** 如果需要对访问用户的IP地址进行分析，需要使用IP地址插件，安装该插件以后，可以按照国家进行统计，它的安装稍微复杂一些。 我们使用GeoIPfree插件，首先在配置文件中打开该插件。 LoadPlugin=”geoipfree”

  - **下载GeoIPfree软件：** 该软件为免费软件， 网址为： [http://search.cpan.org/~gmpassos/Geo-IPfree-0.2/](http://search.cpan.org/~gmpassos/Geo-IPfree-0.2/)，文件为：Geo-IPfree-0.2.tar.gz

  - **安装配置GeoIPfree软件：** 首先使用winzip等解压缩该软件，再进行软件的编译链接，在命令行执行：perl makefile.pl，再将lib目录下的geo目录复制到Perl的库文件目录中或者AWStats的插件目录中。其中Perl的库目录为x:Perllib ，AWStats插件目录为x:AWStatswwwrootcgi-binplugins。
- 初始化统计信息：
  - 在以上配置信息以及插件都安装完毕以后，首次运行之前的需要手工在命令行进行统计信息初始化工作。

  - 在命令行，切换目录到x:awstatswwwrootcgi-bin，运行 (如果pl没有与perl解释程序相关联，请加上x:Perlbinperl.exe )awstats.pl –config=www.mysite.com.cn –update

  - 按照awstats.www.mysite.com.cn.conf网站配置信息进行初次统计分析，它会使用前一天的日志文件进行分析，如果日志文件数据量很大，分析可能持续几小时。

  - 每日定时执行统计分析：AWStats是以天为单位进行统计分析的，统计分析需要用户执行，因此我们需要使用操作系统的计划任务使它每日定时进行，定时时间为每日00：05。 首先在x:awstatswwwrootcgi-bin目录中创建批处理文件update.bat，文件内容为：Perl.exe awstats.pl –config=www.mysite.com.cn –update

  - 如果有多个网站的日志需要进行统计分析，那么创建一个全部更新的命令更加方便。在x:awstatstools目录下，创建一个批处理文件updateall.bat，文件内容为：Perl.exe awstats_updateall.pl now -awstatsprog=../wwwroot/cgi-bin/awstats.pl -configdir=../wwwroot/cgi-bin

  - 在Windows Server 2003命令行中创建计划任务：schtasks /create /tn “AWStats Update Statistics” /tr d:awstatswwwrootcgi-binupdate.bat /sc daily /st 00:05或者schtasks /create /tn “AWStats UpdateAll Statistics” /tr d:awstatstoolsupdateall.bat /sc daily /st 00:05或者或者使用计划任务向导创建计划，每日00:05分执行批处理程序，更新统计信息，注意在执行计划任务时需要指定运行该计划的用户和密码，最好为该任务创建一个专用用户帐号，并为适当的目录指定相应的权限。
- 查看统计信息：AWStats的日志统计分析信息以Web方式提供给用户，查看方式为：http://www.mysite.com.cn/awstats/cgi-bin/awstats.pl?config=www.mysite.com.cn