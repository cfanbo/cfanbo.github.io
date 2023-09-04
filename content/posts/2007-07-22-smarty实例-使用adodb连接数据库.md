---
title: Smarty实例 – 使用ADODB连接数据库
author: admin
type: post
date: 2007-07-22T19:23:10+00:00
url: /archives/52
IM_contentdowned:
 - 1
categories:
 - 程序开发

---
今天就先来说说ADODB.说到ADODB,可能做过ASP的都知道WINDOWS平台的ADO组件,但我们这里的ADODB不是微软的那个数据库操作组件,而是由php语言写的一套数据库操作类库,先让我们来看看它倒底有什么样的优点.
1. 以标准的SQL语句书写的数据库执行代码在进行数据库移植时不用更改源程序,也就是说它可以支持多种数据库,包括ACCESS.

2. 提供与微软ADODB相似的语法功能.这一点对于从ASP转行到PHP的人们是一大福音,它的很多操作都与WINDOWS中的ADODB相似.

3. 可以生成[Smarty][1]循环需要的二维数组,这样会简化smarty开发.这一点是等会我给大家演示.
4. 支持数据库的缓存查询，最大可能的提高查询数据库的速度。
5. 其它的实用功能.
虽然说优点很多,但是由于这个类库非常的庞大,光它的主执行类就107K,所以如果大家考虑执行效率的话就要认真想想了.不过说实话,它的功能还是很强大的,有很多的很实用的功能,使用它的这些功能,可以非常方便的实现我们想要的功能.所以对于那些老板没有特殊要求时大家不防用用它.
**一、如何得到ADODB? 它的运行环境是什么？
** 从http://sourceforge.net/project/show…簆hp4.0.5以上。
**二、如何安装ADODB?**
解压下载回的压缩文件，注意：大家下载回来的格式为ADODB.tar.gz，这是linux的压缩格式，在windows下大家可以使用winrar对其进行解压，解压完成后将目录拷贝到指定的目录的adodb目录下，像我在例子中将它拷贝到了/comm/adodb/中。
**三、如何调用ADODB?**
使用include_once (“./comm/adodb/adodb.inc.php”);这行就不用说了吧？包含ADODB的主文件。
**四、如何使用ADODB?**
1.进行初始化：
ADODB采用$conn = ADONewConnection();这样的语句进行初始化,对ADODB进行初始化有两种方式：
第一种方式为：传统方式。我暂时称它为这个名称。它使用的建立一个新连接的方式很像php中的标准连接方式：
$conn = new ADONewConnection($dbDriver);
$conn->Connect($host, $user, $passwd, $db);
简单吧？如果使用过phplib中的db类应该对它很熟悉的。

第二种方式：采用dsn方式，这样是将数据库的连接语句写成一条语句来进行初始化，dsn的写法有为：$dsn = “DBType://User:Passwd@Host/DBName”; 其中DBType表示数据库类型，User表示用户名，Passwd为密码，Host为服务器名，DBName为数据库名，像这样我使用oracle数据库，用户名：oracleUser,密码为oraclePasswd,数据库服务器为localhost, 数据库为oradb的dsn这样写：
$dsn = “oracle://oracleUserraclePasswd@localhost/oradb”;
$conn = new ADONewConnection($dsn);
这种方式可能从ASP转行来的程序员会更感兴趣。

这两种方式都可以使用，要看个人习惯来选用了.

2. 相关的概念：
使用ADODB有两个基本的类，一是是ADOConnection类，另一个是ADORecordSet类，使用过ASP的人看到这两个类会明白它的含义，ADOConnection指的是数据库连接的类，而ADORecordSet指的是由ADOConnection执行查询语句返回的数据集类，相关的资料大家可以查询ADODB类的手册。

3.基本的函数：

关于ADOConnection类的相关方法有：
1.Connect：数据库连接方法，上边我们介绍过的。对于mysql还有PConnect，与PHP语言中的用法一样
2.Execute($sql):执行查询语句结果返回一个ADORecordSet类。
3.GetOne($sql):返回第一行的第一个字段
4.GetAll($sql):返回所有的数据。这个函数可是大有用处，记得不记的我在以前的教程中写关于新闻列表的输入时要将需要在页面显示的

新闻列表做成一个二维数组？就是这样的语句：

```php
while($db->next_record())
{
$array[] = array(“NewsID” => $db->f(“iNewsID”),
“NewsTitle” => csubstr($db->f(“vcNewsTitle”), 0, 20));
}
```



这一行是什么意思呢？就是将要显示的新闻例表生成
````php
$array[0] = array(“NewsID”=>1, “NewsTitle”=>”这里新闻的第一条”);
$array[1] = array(“NewsID”=>2, “NewsTitle”=>”这里新闻的第二条”);
…
````



这样的形式，但如果我们不需要对标题进行控制，在ADODB中我们就有福了，我们可以这样写：

```php
$strQuery = “select iNews, vcNewsTitle from tb\_news\_ch”;
$array = &$conn->GetAll($strQuery);//注意这条语句
$smarty->assign(“News_CH”, $array);
```



unset($array);
==================================================================================
当然，这里的$conn应该进行初始化过了，不知大家看明白了没有？原来我要手工创建的二维数据在这里直接使用GetAll就行了！！！这也是为什么有人会说ADODB+Smarty是无敌组合的原因之一了…
4.SelectLimit($sql, $numrows=-1, $offset=-1, $inputarrr=false): 返回一个数据集，大家从语句上也不难看出它是一条限量查询语句，与mysql语句中的limit 有异曲同工之效，来一个简单的例子：
$rs = $conn->SelectLimit(“select iNewsID, vcNewsTitle from tb\_news\_CH”, 5, 1);
看明白了吗？$rs中保存的是数据库中从第一记录开始的5条记录。我们知道，在oracle数据库不支持在SQL语句中使用limit，但是我们如果使用ADODB的话，那这个问题就容易解决多了！
5.Close():关闭数据库，虽然说PHP在页面结束时会自动关闭，但为了程序的完整大家还是要在页面结束进行数据库的关闭。

关于ADORecordSet.ADORecordSet为$conn->Execute($sql)返回的结果,它的基本函数如下:
1. Fields($colname):返回字段的值.
2. RecordCount():所包含的记录数.这个记录确定数据集的记录总数.
3. GetMenu($name, [$default\_str=”], [$blank1stItem=true], [$multiple\_select=false], [$size=0], [$moreAttr=”])非常好的一个函数,使用它可以返回一个name=$name的下拉菜单(或多选框)!!!当然,它是一个HTML的字符串,这是一个令人激动的好东西,$name指的是option的name属性,$default\_str是默认选中的字串,$blank1stItem指出第一项是否为空,$multiple\_select指出是否为多选框,而我们得到这个字串后就可以使用$smarty->(“TemplateVar”, “GetMenuStr”)来在模板的”TemplateVar” 处输入一个下拉列表(或是多先框)
4. MoveNext():来看一段代码:

  ```php
  $rs = &$conn->Exceute($sql);
  if($rs){
     while($rs->EOF){
     	$array[] = array(“NewsID” => $rs->fields[“iNewsID”],
     	“NewsTitle” => csubstr($rs->fields[“vcNewsTitle”]), 0, 20);
  
    	$rs->MoveNext();
    }
  }
  ```

  

明白了吗?很像MS ADODB中的那一套嘛!
5. MoveFirst(),MoveLast(), Move($to):一样的,看函数名大家就可以知道它是什么意思了.
6. FetchRow():返回一行,看代码:

  ```php
  $rs = &$conn->Exceute($sql);
  if($rs) {
    while($row = $rs->FetchRow())
    {
      $array[] = array(“NewsID” => $row[“iNewsID”],
      “NewsTitle” => csubstr($row[“vcNewsTitle”]), 0, 20);
    }
  }
  ```

  

  它实现了与4一样的功能,但看起来更符合PHP的习惯,而4的习惯看起来更像是MS ADODB的办法．

7.GetArray($num):返回数据集中的$num行数据,将其组合成二维数组.这个方法我们在例子index.php要用到.

8. Close():同mysql\_free\_result($rs);清除内容占用.

好了,初步的函数就介绍到这里,够我们用的啦!实际上ADODB还有很多实用的技术,包括格式化日期时间,格式化查询语句,输出表格,更高级点的Cache查询,带参查询等等,大家可以自行查看手册.

下面我们开始学习我们的程序,同样还是那个Web程序,我将其中的comm目录重新组织了一下,同时为了提高效率对Smarty重新进行了封装,mySmarty.class.php是封装后的类,它继承自Smarty,所以以后所有的程序文件中只调用新的类MySmarty,先来看看目录结构:
+Web (站点根目录)
|
|—-+comm (Smarty相关文档目录)
| |
| |—-+smarty (Smarty原始文件目录)
| |—-+adodb (adodb原始文目录)
| |—–mySmarty.class.php (扩展后的smarty文件)
| |—–csubstr.inc (截取中文字符)
|
|—-+cache (Smarty缓存目录，*nix下保证读写权限)
|
|—-+templates (站点模板文件存放目录)
| |
| |—-header.tpl(页面页头模板文件)
| |—-index.tpl(站点首页模板文件)
| |—-foot.tpl(页面页脚模板文件)
| |—-news.tpl (新闻页模板文件)
|
|
|—-+templates_c (模板文件编译后存放目录，*nix下保证读写权限)
|
|—-+css (站点CSS文件目录)
|
|—-+image (站点图片目录)
|
|—-+media (站点Flash动画存放目录)
|
|—-indexbak.htm (首页原始效果图)
|
|—-newsbak,htm (新闻页原始效果图)
|
|—-index.php (Smarty首页程序文件)
|
|—-news.php (Smarty新闻显示文件)
|
|—-newsList.php (显示新闻列表)
|
|—-例程说明.txt (本文档)

相对于前两个教程,有将comm目录重新组织了一下,其它的文件结构没有变化,整个站点相对于上两个教程来讲,改变的地方只有comm目录与index.php与news.php,同时增加了新闻列表,大家可以在index.php执行后的页面中点击”国内新闻”,”国际新闻”, “娱乐新闻”来分别查看各自的新闻列表, 我们先来看看index.php:

======================================================
index.php
======================================================
Connect(“localhost”, “root”, “”, “News”); //连接数据库

//这里将处理国内新闻部分
3. $strQuery = “Select iNewsID AS NewsID, vcNewsTitle AS NewsTitle FROM tb\_news\_CH orDER BY iNewsID DESC”;
4. $rs = &$conn->Execute($strQuery);
5. $smarty->assign(“News\_CH”, $rs->GetArray(NEWS\_NUM));
6. unset($rs);

//这里处理国际新闻部分
$strQuery = “Select iNewsID AS NewsID, vcNewsTitle AS NewsTitle FROM tb\_news\_IN orDER BY iNewsID DESC”;
$rs = &$conn->Execute($strQuery);
$smarty->assign(“News\_IN”, $rs->GetArray(NEWS\_NUM));
unset($rs);

//这里将处理娱乐新闻部分
$strQuery = “Select iNewsID AS NewsID, vcNewsTitle AS NewsTitle FROM tb\_news\_MU orDER BY iNewsID DESC”;
$rs = &$conn->Execute($strQuery);
$smarty->assign(“News\_MU”, $rs->GetArray(NEWS\_NUM));
unset($rs);

7. $conn->close();

//编译并显示位于./templates下的index.tpl模板
$smarty->display(“index.tpl”);
?>

同样,我在关键的地方加了数标,下面来说明一下它们的含义:

1. 建立一个连接对象$conn,大家在这里要注意的是它的初始不是以$conn = new ADONewConnection($dbType)这样的形式出现的,也就是说,ADONewConnection不是一个class,你不能使用new 对它进行初始化.看看它的源码你就会明白,这只不过是一个函数.

2. 这个就不用说了吧?打开一个News的数据库,主机为:localhost, 用户名为root, 密码为””

3. 一个查询语句,注意,这里要将查询的字段使用AS关键字来重新标识,名称为你在模板中设置的模板变量的名称.

4. 使用Execute来执行这个查询,结果返回一个RecordSet数据集

5. 这里有个方法:$rs->GetArray($num) 这个在上边介绍过,它是要从$rs这个数据集中返回$num行,结果为一个可被Smarty所识别的二维数据.这样ADODB就自动为我们构建起了这样的结构,而在我们以前的例子中,都是使用一个循环构建这样的数组的.

6. 这一句我看也不用说了吧?

7. 关闭内存中的相关资源.

大家可以看看,整个程序中再没有出现什么while语句,程序整体结构显的非常清楚,这就是为什么ADODB+Smarty是黄金组合的原因.不过话也说回来了,简单有简单的问题,不知大家想过没有,这里对显示的新闻标题的长度没有控制,也就是说,如果某条新闻标题的长度超出一行显示的范围,它就是自动折行到下一行,那么整个的版面就会变乱,所说大家自已适自己的情况来决定是否这样使用吧当然,你也可以使用像上一节中介绍的那样,使用一个循环语句重构这个二维数组,使它符合你的用途,怎么做大家自己去想吧,参考PHPLIB中的做法,上节我介绍过了…

再来看看新闻页吧

=============================================================
`news.php`

```
Connect(“localhost”, “root”, “”, “News”); //连接数据库

$NewsID = $_GET[“id”]; //获取新闻编号
$NewsType = $_GET[“type”]; //要显示的新闻类型
switch($NewsType)
{
case 1: ****$dbName = “tb\_news\_CH”;
break;
case 2:
$dbName = “tb\_news\_IN”;
break;
case 3:
$dbName = “tb\_news\_MU”;
break;
}

$strQuery = “Select vcNewsTitle AS NewsTitle, ltNewsContent AS NewsContent FROM ” . $dbName;

1. $row = &$conn->GetRow($strQuery); //返回一个一维数组,下标为模板变量名

$smarty->display($row);
unset($row);

$conn->Close();

?>
```



说明一下关键的地方,其实在news.php中也只有一个地方值的说明一下了.

1. $conn->GetRow($strQuery):这一句返回一个一维数组,返回的形式为:

$array = (“NewsTitle”=>”xxxx”, “NewsContent”=>”yyyyy…”)
明白如果使用$smarty($array)后Smarty会干什么吗?对了,就是相当于:
$smarty->assign(“NewsTitle”, “xxxx”);
$smarty->assign(“NewsContent”, “yyyyy…”);

简单吧,确实很简单

下面再来看看新闻列表:
================================================================
newsList.php

```php
Connect(“localhost”, “root”, “”, “News”); //连接数据库

$NewsID = $_GET[“id”]; //获取新闻编号
$NewsType = $_GET[“type”]; //要显示的新闻类型
switch($NewsType)
{
case 1:
$tbName = “tb_news_CH”;
break;
case 2:
$tbName = “tb_news_IN”;
break;
case 3:
$tbName = “tb_news_MU”;
break;
}

$strQuery = “Select iNewsID AS NewsID, vcNewsTitle AS NewsTitle FROM ” . $tbName;

$rs = &$conn->GetAll($strQuery);
$smarty->assign(“NewsType”, $NewsType); //这一句为新闻列表中的链接服务
$smarty->assign(“NewsList”, $rs);
 unset($rs);
 $conn->close();

$smarty->display(“newsList.tpl”);

?>
```



分别来说明一下:

1. GetAll($strQuery):这个函数可是个好东东,它的作用是将$strQuery查询到的所有数据组合成为一个能够被Smarty所识别的二维数组,记住:它返回的是一个二维数组而不是一个RecordSet,所在你可以程序中直接在3处使用.
2. 这里是为了给新闻标题做链接时要GET参数type=XX而做的

后记:
大家在使用ADODB时有几个地方要注意:
1. 初始化: 初始化的方式不是使用new,因为它不是一个对象
2. 方 法: 基本上每个方法都是以大写字母开头大小写混合的名称,这一点好像与*NIX的习惯有些不同,也不同于PHP的整体风格,所以注意这里的大小写问题.

[1]: http://smarty.php.net