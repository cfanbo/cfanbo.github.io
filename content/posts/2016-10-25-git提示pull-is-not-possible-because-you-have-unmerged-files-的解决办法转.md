---
title: git提示“Pull is not possible because you have unmerged files.”的解决办法(转)
author: admin
type: post
date: 2016-10-25T03:17:47+00:00
url: /archives/17128
categories:
 - 程序开发
tags:
 - git

---
在git pull的过程中，如果有冲突，那么除了冲突的文件之外，其它的文件都会做为staged区的文件保存起来。

重现：

$ git pull

A    Applications/Commerce/BookingAnalysis.java
A    Applications/Commerce/ClickSummaryFormatter.java
M    Applications/CommerceForecasting/forecast/Forecast.java
A    Applications/CommerceForecasting/forecast/ForecastCurveProviderCategory.java
M    Applications/CommerceForecasting/forecast/ForecastProvider.java
M    Applications/CommerceForecasting/forecast/InputPropertyItem.java
……

A    Applications/LocalezeImporter/com/tripadvisor/feeds/SingleMenuLocalezeMatcher.java
A    Applications/LocalezeImporter/com/tripadvisor/feeds/TypeCategory.java

Pull is not possible because you have unmerged files.Please, fix them up in the work tree, and then use ‘git add/rm ’as appropriate to mark resolution, or use ‘git commit -a’.

通过git status你会发现下面古怪的事情：

zhonghua@pts/ttys000 $ git status
\# On branch sns
\# Your branch and ‘snsconnect/sns’ have diverged,
\# and have 1 and 52 different commit(s) each, respectively.
#
\# Changes to be committed:
#
#    new file:   src/config/features_daodao.ini
#    new file:   src/config/services.xml
#    new file:   src/config/svnroot/hooks/mailer.conf
#    new file:   src/config/svnroot/hooks/mailer.py
#    new file:   src/config/svnroot/hooks/post-commit
#    new file:   src/config/svnroot/hooks/pre-commit
#    new file:   src/config/svnroot/hooks/prerelease_notifications.py
#    new file:   src/config/svnroot/hooks/run_checks.py
…….

\# Untracked files:
#   (use “git add …” to include in what will be committed)
#
#    _build/
#    css/combined/
#    css/gen/
#    daodao-site.patch
#    daodao-site.patch1
#    js/combined/
#    js/gen/
#    lib/weibo/
#    src/bin/

Pull is not possible because you have unmerged files.

解决：

1.pull会使用git merge导致冲突，需要将冲突的文件resolve掉 git add -u, git commit之后才能成功pull.

2.如果想放弃本地的文件修改，可以使用

> git reset –hard FETCH_HEAD

FETCH_HEAD表示上一次成功git pull之后形成的commit点。然后git pull.
注意：

git merge会形成MERGE-HEAD(FETCH-HEAD) 。git push会形成HEAD这样的引用。HEAD代表本地最近成功push后形成的引用。

参考： [http://www.ithao123.cn/content-4783203.html](http://www.ithao123.cn/content-4783203.html)