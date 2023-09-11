---
title: 动态网页制作技巧PHP：详细彻底学习Smarty
author: admin
type: post
date: 2008-11-20T21:22:08+00:00
excerpt: |
 |
 {$smarty.now}变量用于访问当前时间戳.
 可以用 date_format调节器格式化输出. 例如{$smarty.nowdate_format:"%Y-%m-%d %H:%M:%S"}

 {$smarty.const}
 你可以直接访问PHP常量. 例如{$smarty.const._MY_CONST_VAL}

 {$smarty.capture}
 可以通过 {capture}..{/capture}结构 截取的输出可以使用{$smarty} 变量访问.

 {$smarty.config}
 {$smarty}变量 可以访问已经加载的config变量.
 例如 {$smarty.config.foo}就可以表示 {#foo#}.
url: /archives/629
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php
 - smarty

---
页面请求变量
以下是访问页面请求变量诸如get,post,cookies,server,enviroment和session变量的例子. 例如{$smarty.server.SERVER_NAME}取得服务器变量，{$smarty.env.PATH}取得系统环境变量path, {$smarty.request.username}取得get/post/cookies/server/env的复合变量。

{$smarty.now}变量用于访问当前时间戳.
可以用 date\_format调节器格式化输出. 例如{$smarty.nowdate\_format:”%Y-%m-%d %H:%M:%S”}

{$smarty.const}
你可以直接访问PHP常量. 例如{$smarty.const.\_MY\_CONST_VAL}

{$smarty.capture}
可以通过 {capture}..{/capture}结构 截取的输出可以使用{$smarty} 变量访问.

{$smarty.config}
{$smarty}变量 可以访问已经加载的config变量.
例如 {$smarty.config.foo}就可以表示 {#foo#}.

{$smarty.section}, {$smarty.foreach}
{$smarty} 变量可以访问’section’和’foreach’循环的属性.

{$smarty.template}
显示当前被处理的模板的名字.

{$smarty.version}
显示smarty模板的版本

{$smarty.ldelim}
显示左分隔符

{$smarty.rdelim}
显示右分隔符



**变量调节器**

变量调节器用于变量,自定义函数和字符串.

可以使用”符号和调节器名称应用调节器.

变量调节器由赋予的参数值决定其行为.

参数由’:’符号分开.

如果你用变量调节器调节数组变量,结果是数组的每个值都被调节.如果你想要调节器调节整个数组,你必须在调节器名字前加上@符号.

例如: {$articleTitle@count}(这将会在输出 $articleTitle 数组里的数目)

**capitalize**
将变量里的所有单词首字大写. 参数值boolean型决定带数字的词是否首字大写。默认不大写

**count_characters**
计算变量值里的字符数.参数值boolean型决定是否计算空格数。默认不计算空格

**cat**
将cat里的参数值连接到给定的变量后面.默认为空。

**count_paragraphs**
计算变量里的段落数量

**count_sentences**
计算变量里句子的数量

**count_Words**
计算变量里的词数

**date_format**
日期格式

第一个参数控制日期格式.
如果传给date_format的数据是空的,将使用第二个参数作为默认时间

%a – 星期几的简写

%A – 星期几的全写

%b – 月份的简写

%B – 月份的全写

%c – 日期时间06/12/05 11:15:10

%C – 世纪时间

%d – 一个月的第几号(从 01 到 31)

%D – 同 %m/%d/%y

%e – 一个月的第几号，号为单数则前面加一空格 (从 1 到 31)

%g – 世纪

%G – 世纪 [0000,9999]

%h – 同%b

%H – 24小时形式的小时(从00到23)

%I – 12小时形式的小时(从01到 12)

%j – 一年中的第几天(从 001 到 366)

%k – 24小时形式的小时，单数字前面加空格. (从 0 到 23)

%l – 12小时形式的小时，单数字前面加空格.(range 1 to 12)

%m – 月份 (range 01 to 12)

%M – 分

%n – 换行符

%p – 显示早上还是下午\`am’ 或 \`pm’

%r – a.m. 或 p.m.形式的时间

%R – 24小时形式的时间

%S – 秒

%t – tab符号

%T – 同%H:%M:%S

%u – 用 [1,7],表示星期几

%U – 计算是该年的第几个星期，从该年的第一个星期天开始计算

%V – 计算是该年的第几个星期, 从 01 到 53, 第一个星期必须至少有4天在这一年, 星期天作为这个星期的第一天

%w – 用数字的形式表示是星期的第几天, 星期天 为 0

%W – 用数字的形式是该年的第几个星期,从该年的第一个星期一开始计算

%x – 显示日期：月/日/年

%X – 显示时间：小时：分钟：秒

%y – 不包括世纪的年份

%Y – 包括世纪的年份

%Z – 时区

%% – 输出%

其中有些有时不能正常输出。

**default**
默认
为空变量设置一个默认值.
当变量为空或者未分配的时候,将由给定的默认值替代输出.

**escape**
转码
参数值为html,htmlall,url,quotes,hex,hexentity,javascript。默认是html转码

**indent**
缩进
在每行缩进字符串,第一个参数指定缩进多少个字符，默认是4个字符.第二个参数,指定缩进用什么字符代替。

**lower**
小写
This is used to lowercase a variable.
将变量字符串小写

**nl2br**
换行符替换成

**regex_replace**
正则替换
寻找和替换正则表达式.必须有两个参数，参数1是替换正则表达式. 参数2使用什么文本字串来替换

**replace**
替换
简单的搜索和替换字符串必须有两个参数，参数1是将被替换的字符串. 参数2是用来替换的文本

**spacify**
spacify是在字符串的每个字符之间插入空格或者其他的字符串. 参数表示将在两个字符之间插入的字符串，默认为一个空格。

**string_format** 字符串格式化
是一种格式化浮点数的方法.例如十进制数.使用sprintf语法格式化。参数是必须的，规定使用的格式化方式。%d表示显示整数，%.2f表示截取两个浮点数。

**strip** 去除(多余空格)
替换所有重复的空格,换行和tab为单个或者指定的字符串. 如果有参数则是指定的字符串。

**strip_tags** 去除所有html标签

**truncate** 截取

参数1，规定截取的字符数.默认是80个.
第二个参数指定在截取的那段字符串后加上什么字符.默认为…
第三个参数决定是否精确截取，默认情况下为false,则smarty不会分割单词。

**upper** 将变量改为大写

**wordwrap** 行宽约束
第一个参数指定段落的宽度(也就是多少个字符一行,超过这个字符数换行).默认80.
第二个参数指定在约束点使用什么字符(默认是换行符\n).
第三个参数决定是否精确截取字符，默认情况下是不精确截取，就是截取时不能分开单词。

**内建函数不能擅自修改。**
**capture**
capture函数的作用是收集模板输出的数据到一个变量里,而不是把它们输出到页面.例如任何在 {capture name=”foo”}和{/capture}之间的数据都被收到了由函数的名称属性指定的变量{$foo}里，或者{$smarty.capture.foo}里。如果函数没有名字属性,将使用”default”.每个{capture}都必须对应{/capture},也不能嵌套使用capture函数。

**config_load**
引用配置文件
file是必须的，说明要包含进来的配置文件名称，section说明要加载的部分的名称，scope被处理的变量的作用域.必须是local,parent或者global.
local的意思是变量将在本模板里被加载.
parent 的意思是变量将在本模板和上级模板被加载.
global的意思是变量将应用到所有的模板.默认为local。变量是否在上级模板可视,默认为no。如果scope属性已经有了,这个值将被忽略.

**foreach,foreachelse**
foreach循环是选择性的section循环.用于遍历关联数组.foreach的语法比section简单的多,但是作为一个折中它只能用于简单数组.
foreach必须的参数是from和item. from变量表示需要循环的数组的名称，item表示当前元素的变量名，key表示当前关键字的变量名，name表示访问foreach属性的foreach循环名。循环可以互相嵌套,被嵌套的循环之间的名字必须是独立的.foreachelse 在from变量没有值的时候被执行

**include**
用来引用其他的模板。
file属性是必须的用来表示所引用模板的名字，assign表示include文件将要分配的输出的变量。你可以自行用属性名=”属性值”的方式定义任意个局部变量。

**include_php**
用来在模板中引入php脚本。file是必须的用来表示php脚本的路径，once确定如果在模板中引用了php脚本多次，是否只装载一次。默认为true。

**insert**
用来包含php脚本中的函数，name是必须的，表示所插入的脚本的名称，注意如果名称是name，则包含的函数则是insert\_name(),所以所有要插入的函数要有前缀insert\_ 。如果用了assign属性，则insert的输出将会分配给模板变量而不会显示。 script表示要引用的脚本路径。这个程序产生的内容将不会被缓存，在每次调用该页时重新执行，适用于广告，投票，查询结果等互动的地方。

**if,elseif,else**
if语句和和条件同php差不多，但每个词之间必须用空格分割开。也有一些新的条件语句，列举如下：eq相等，ne、neq不相等，gt大于，lt小于，gte、ge大于等于，lte、le 小于等于，not非，mod求模。is [not] div by是否能被某数整除，is [not] even是否为偶数，$a is [not] even by $b即($a / $b) % 2 == 0，is [not] odd是否为奇，$a is not odd by $b即($a / $b) % 2 != 0

**php**
php标记可以让模板中能直接使用php语言。

**section,sectionelse**
section用来循环显示数组的数据，name和loop是必须的参数。name表示嵌套名. section 可以嵌套使用,但是名字必须各不相同。loop表示循环的次数. sectionelse在loop参数为空的输出。start用来规定循环开始的指针,如果值为负则从数组尾部计算开始的指针,默认为0.step表示循环的步数,为负则反向循环,默认为1.max设定循环的最大步数.show决定是否显示section.
section也有自己的变量处理section属性,用{$smarty.section.sectionname.varname} 来显示.