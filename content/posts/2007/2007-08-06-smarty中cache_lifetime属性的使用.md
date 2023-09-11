---
title: smarty中$cache_lifetime属性的使用
author: admin
type: post
date: 2007-08-06T16:32:43+00:00
url: /archives/77
IM_contentdowned:
 - 1
categories:
 - 程序开发

---
Setting$cache_lifetimepercache

require(‘Smarty.class.php’);

$smarty=newSmarty;

$smarty->caching=2;//lifetimeispercache

//setthecache_lifetimeforindex.tplto5minutes

$smarty->cache_lifetime=300;

$smarty->display(‘index.tpl’);

//setthecache_lifetimeforhome.tplto1hour

$smarty->cache_lifetime=3600;

$smarty->display(‘home.tpl’);

//NOTE:thefollowing$cache_lifetimesettingwillnotworkwhen$caching=2.

//Thecachelifetimeforhome.tplhasalreadybeenset

//to1hour,andwillnolongerrespectthevalueof$cache_lifetime.

//Thehome.tplcachewillstillexpireafter1hour.

$smarty->cache_lifetime=30;//30seconds

$smarty->display(‘home.tpl’);

?>

If $compile_check isenabled,everytemplatefileandconfigfilethatisinvolvedwiththecachefileischeckedformodifica-
tion.Ifanyofthefileshavebeenmodifiedsincethecachewasgenerated,thecacheisimmediatelyregenerated.Thisisa
slightoverheadsoforoptimumperformance,set$compile_check toFALSE.