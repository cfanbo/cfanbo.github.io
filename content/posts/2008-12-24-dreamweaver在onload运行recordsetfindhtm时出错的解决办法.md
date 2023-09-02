---
title: dreamweaver在onLoad运行RecordsetFind.htm时出错的解决办法
author: admin
type: post
date: 2008-12-24T03:01:36+00:00
excerpt: |
 |
 今天单位的Dreamweaver出错了，折腾了半天，重新装了8.02，出现下面的错误：
 在onLoad运行RecordsetFind.htm时,发生了以下JavaScript错误:
 在文件""RecordsetFind"":
 ReferenceError:findRs is not defined
url: /archives/735
IM_contentdowned:
 - 1
categories:
 - 前端设计

---
今天单位的Dreamweaver出错了，折腾了半天，重新装了8.02，出现下面的错误：
**在onLoad运行RecordsetFind.htm时**, **发生了以下** JavaScript错误:

 在文件””RecordsetFind””:

 ReferenceError:findRs is not defined

卸载掉了，删除安装目录下的文件夹，清除注册表相应的项目，重装，问题依旧，郁闷！
再次卸载，装老版的DW 2004, 也出现部分菜单打不开，点击就不停的抱错，汗！再次装8.02，还是不成，网上也搜不到任何解决方法。无意中删除C:Documents and SettingsAdministratorApplication DataMacromedia把Dreamweaver 8 这个文件夹，另外清寒要删除”Common”这个文件夹,重新打开dw ，居然ok了，呵呵，果真是天无绝人之路！