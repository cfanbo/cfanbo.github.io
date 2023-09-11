---
title: 'Smarty高级特性 – 用户自定义函数使用 SMARTY:: register_function;'
author: admin
type: post
date: 2010-05-14T03:27:00+00:00
url: /archives/3586
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php
 - smarty

---
Smarty高级 特性 – 用户自定义函数 使用 SMARTY:: register_function;

前言：
很久不用smarty了，因为大多数项目都是比较轻量型的。前段时间 笔者接了个还算可以的项目，下面有几个程序员 ，与一个美工组为项目团队。为了做快速的布署应用 ，也为了小组成员能形成一个统一的view层的控制，选择了smarty。发现smarty果然还是那么的强大，那么的很黄很暴力。
作者：无喱头

故事背景：
小张是个很漂亮的美工MM，与无喱头搭档已经很多年。请不要误解，无喱头是有老婆女儿的，他们之前没有任何的暧昧关系，仅仅是同事，或者是上下级。
在两人的多年合作过程中，在很多地方，已经形成了一种默契。在很多时候，喱头提供封装好的php函数，然后通过一些技术 上的修改，可以直接使用小张在模板里引入php函数，这样可以很方便的把模板切成很多小块，便于维护。并且由于可以自定义一些关键字，小张可能很快的取出 想要的一些数据 。
比如：
{phpsoho “sort=article&order=ID DESC&limit=10&tplfolder=article&tplname=article.list”}，无喱头为小 张同学提供了类似于上面的自定义函数，并且明确的给出了使用文档：
Sort: 类别(文章?下载?图片？) article|download|picture
Order:排序方式 id,hot,? DESC|ASC
Limit:取出多少条？ Limit number|limit start,end
…….
当然，不仅仅是这些，也不仅仅只有phpsoho这个自定义的函数来定义。小张可能很快的通过这些她能看得懂的文档（打印稿）来很方便的进行界面的操作。
由于种种原因，现在准备使用SMARTY。现在问题来了，怎么让SMARTY也支持用户自定义的函数呢？
解 决 方案：
SMARTY手册是这样介绍register_function函数的

Use this to dynamically register template function plugins. Pass in the template function name, followed by the PHP function name that implements it.
The php-function callback impl can be either (a) a string containing the function name or (b) an array of the form array(&$object, $method) with &$object being a reference to an object and $method being a string containing the mehod-name or (c) an array of the form array(&$class, $method) with $class being a classname and $method being a class method of that class.

其实，那么多英语我也不太看得懂，经过两天的摸索，终于还是有了一些心得的。

**基础篇**

我们可以直接使用SMARTY::register_function来注册一个程序 员自定义的php函数，只要这个函数所在的文件 已经被引入，就可以正确使用它。
DEMO 1：

```
$smarty = set_smarty();
$smarty->register\_function(‘example’,’format\_data’);
$smarty->assign(‘time’,time());
$smarty->display(‘demo.htm’);
```



自定义函数如下：

```
function format_data( $params )
{
extract($params);
echo date($format,$time);
}
```



模板文件如下：

```
{example time=”$time” format=”Y-m-d”}
```



不难看出，我们很轻松的就布署使用了我们的自定义函数了。有一点我们需要注意到的是，在format_data函数里，我们是以一个数组对值进行传入的， 而在模板文件里，我们可以任意的建一些关键字，只要你能记得住。当然，除了SMARTY的保留字之外。
而后我们通过extract把这个数组里面的值释放放出来，当然这种方案是有多种多样的，你可以使用list，也可以使用explode，总之你能想到的办法 就是好办法，只要你用得熟悉。

**进阶篇**

有一种情况很快发生了，小张总是搞不清楚我到底注册了多少个自定义函数供她使用，她总是忘记一些关键字的定义，比如在一个取出热门的连载小说里，她不知道 有没有hot这个关键词。因为这种自定义的函数太多了，她不得不经常的拿着我给她打印出来的技术文档苦着脸查找，我很内疚。于是，另一种解决方案应运而 生。
本文开始时，曾经举了了像{phpsoho “sort=article&order=ID DESC&limit=10&tplfolder=article&tplname=article.list”}这样的字符 串在模板里的例子。或者你会说，头哥在骗人呀，这根本跟第一个例子是一样的嘛，无非是传来的值不一样而已。那么，咱们接着往下看。。。
DEMO 2:
我们假装smarty已经被实例 化，并且资源名为$smarty.
首先我们注册一个自定义函数到smarty中，这个跟上例一样，完全是为了骗几个字。

$smarty->register\_function( ‘phpsoho’,’tags\_extends’ );
//引入文件
include\_once \\_\_ROOT\_PATH\\_\_ . ‘/include/tags.func.php’;

这里需要注意的是，我们注册的到phpsoho句柄中的函数为tags_extends，函数名是什么意思不做一一解释，具体可以看PHP函数命名规范这 样的文档。
```
function tags_extends( $params )
{
$action = $params[‘action’];
$optional = $params[‘optional’];

/**

* 关键词检测
  */
  if ( !$action )
  {
  Error::putMsg(‘tags\_extends\_error’,’tags_extends action is null.’);
  }

if( !function_exists($action) )
{
Error::putMsg( ‘function\_exists’,’function null. class file.’.$extend\_tags_file . ‘. classname:’.$action );
}
$action($optional);
}
```



模板文件：
[php]
{phpsoho action=”getAffiche” optional=”fileds=id,title,post time&titlelen=17&and[stauts]=1&limit=5&order=id DESC&tpl=afiche.right”}
[/php]
上面的函数执行流程是这样子的：
在模板文件里面，我们使用phpsoho这个资源句柄把请求定位到’tags_extends’函数中，在这个函数中，我们做了几种检查:
Check Action:检查动作是否被设置 ，这个为必须的关键字.其实我是以这个action来进行下面的函数定位操作的.
Check function_exists：检查相应执行的函数是否存在或者是否已经被加载进入
这里我们不用检查optional，因为我们会在下一步里进行另一步检查.
$action($optional)被执行，也就是说上例中的getAffiche被引入了，并且设置$optional的值为 fileds=id,title,posttime&titlelen=17&and[stauts]=1&limit=5& amp;order=id DESC&tpl=afiche.right，里面的一些一一对应的关系，我就不一一过多的讲解。我是有文档给小张的，你们谁想要，可以联系她。 她可是美女哟~！
回过头来，我们看$action($optional)，为什么我说看$action而不是getAffiche呢？因为谁知道你将来会让他引入什么函数 呢？（狂笑，得意的笑）
$action()DEMO：
[php]
function getAffiche($optional)
{
global $db;
parse_str($optional);
$res = $db->getFetchArray(‘litou_affiche’,$fileds,$and,$order,$limit);

模板文件：
[php]
{phpsoho action=”getAffiche” optional=”fileds=id,title,post time&titlelen=17&and[stauts]=1&limit=5&order=id DESC&tpl=afiche.right”}
[/php]
上面的函数执行流程是这样子的：
在模板文件里面，我们使用phpsoho这个资源句柄把请求定位到’tags_extends’函数中，在这个函数中，我们做了几种检查:
Check Action:检查动作是否被设置 ，这个为必须的关键字.其实我是以这个action来进行下面的函数定位操作的.
Check function_exists：检查相应执行的函数是否存在或者是否已经被加载进入
这里我们不用检查optional，因为我们会在下一步里进行另一步检查.
$action($optional)被执行，也就是说上例中的getAffiche被引入了，并且设置$optional的值为 fileds=id,title,posttime&titlelen=17&and[stauts]=1&limit=5& amp;order=id DESC&tpl=afiche.right，里面的一些一一对应的关系，我就不一一过多的讲解。我是有文档给小张的，你们谁想要，可以联系她。 她可是美女哟~！
回过头来，我们看$action($optional)，为什么我说看$action而不是getAffiche呢？因为谁知道你将来会让他引入什么函数 呢？（狂笑，得意的笑）
$action()DEMO：
[php]
function getAffiche($optional)
{
global $db;
parse_str($optional);
$res = $db->getFetchArray(‘litou_affiche’,$fileds,$and,$order,$limit);

/**

* 声明并发送到模板
*/
$smarty = set_smarty();
$smarty->assign(‘titlelen’,$titlelen);
$smarty->assign(‘affiche’,$res);
$smarty->display( $tpl .$GLOBALS[‘tplEx’]);
}
[/php]
大家看到了吗？原来如此简单。。。
原来我们在函数的内容完成了指定内容的一些操作。当然，我这个完全是为了写这篇文章临时写出来的，但你们可以在这里面加上一些php的高级特性，比如：缓 存，在你的字符上加入cache=3600(一般我们使用的是秒，就是一个小时)，那你可以通过上面的一些关键词定义引入你的cache，并且可以根据时 间来更新后引入，加上过滤或者其它任何你想要的功能 。
总之，只要你有想法，一切皆有可能。
知识点：
[php]
parse_str, Parses the string into variables
void parse_str ( string $str [, array &$arr ] )
[/php]
这是个很好用的函数，包括它的家族兄弟们 parse\_ini\_file及parse_url，具体用法自己去测试。
汗，快下班了，估计今天写不完高级使用了。。。先简单介绍一下，留个悬念，以后再过来编辑一下。

**高级篇**

小张很开心。这样她仅仅只需要使用这一个函数就行了，并且里面需要的记的也不多，A4纸一张不到。但苦了我呀，小张经常这样说：头哥，我需要取出一些图片 资料，并且要求横排四张，竖排八张@$#^%$%\*&^\*&(*&()，，，，天啊，我做错了吗？我不得不给她写N多的函数，让 她使用…
每次看到小张一脸诡笑的样子，我心里在发汗，唉。不行，我要再进行深入的研究…