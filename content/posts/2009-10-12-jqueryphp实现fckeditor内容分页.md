---
title: jQuery+PHP实现FCKEditor内容分页
author: admin
type: post
date: 2009-10-12T09:53:33+00:00
excerpt: php中用jquery+fckeditor来实现内容分页功能，实际上是所谓的假分页)
url: /archives/2497
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - FCKEditor
 - jQuery
 - php

---
如题，用jQuery+PHP实现FCKEditor内容分页，如下：

PHP分页函数：
/\***\*****\*\\*\*FCKEditor分页处理\*\*\***\****/
function pageBreak($content)
{
//把文章内容按照

分割成数组
$content  = $content;
$pattern  = “/

<\/span><\/div>/”;
$strSplit = preg\_split($pattern, $content, -1, PREG\_SPLIT\_NO\_EMPTY); //将文章内容分割成数组
$count    = count($strSplit);   //分割后的数组单元数目
$outStr   = “”; //返回的字串
$i        = 1;

if ($count > 1 ) {
$outStr   = “
”;
foreach($strSplit as $value) {
if ($i <= 1) {
$outStr .= “

$value

”;
} else {
$outStr .= “

$value

”;
}
$i++;
}

$outStr .= “

”;
for ($i = 1; $i <= $count; $i++) {
$outStr .= “$i
”;
}
$outStr .= “

”;
return $outStr;
} else {
return $content;
}
}

jQuery代码：

CSS样式代码：
/\*文章分页\*/
#page_break {

}
#page_break .collapse {
display: none;
}
#page_break .num {
padding: 10px 0;
text-align: center;
}
#page_break .num li{
display: inline;
margin: 0 2px;
padding: 3px 5px;
border: 1px solid #FF7300;
background-color: #fff;

color: #FF7300;
text-align: center;
cursor: pointer;
font-family: Arial;
font-size: 12px;
overflow: hidden;
}
#page_break .num li.on{
background-color: #FF7300;

color: #fff;
font-weight: bold;
}