---
title: 在smarty中增加类似foreach的功能自动加载数据
author: admin
type: post
date: 2010-05-14T03:43:13+00:00
url: /archives/3591
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php
 - smarty

---
在smarty中使用自定义插件来加载数据（见：）， 在使用的时候还是感觉不够方便，灵机一动就想写成类似foreach那种标签：

第一步：在Smarty_Compiler.class.php的_compile_tag函数中增加：

[view\
\
plain](http://blog.csdn.net/yycai/archive/2009/12/28/5092770.aspx#) [copy\
\
to clipboard](http://blog.csdn.net/yycai/archive/2009/12/28/5092770.aspx#) [print](http://blog.csdn.net/yycai/archive/2009/12/28/5092770.aspx#) [?](http://blog.csdn.net/yycai/archive/2009/12/28/5092770.aspx#)

01. //加载

     数据的开始标签
02. case‘load’:
03. $this->_push_tag(‘load’);
04. return$this->_complie_load_start($tag_args);
05. break;
07. //加载数据的结束标签
08. case‘/load’:
09. $this->_pop_tag(‘load’);
10. return“”;
11. break;

第二步：增加一个方法：

[view\
\
plain](http://blog.csdn.net/yycai/archive/2009/12/28/5092770.aspx#) [copy\
\
to clipboard](http://blog.csdn.net/yycai/archive/2009/12/28/5092770.aspx#) [print](http://blog.csdn.net/yycai/archive/2009/12/28/5092770.aspx#) [?](http://blog.csdn.net/yycai/archive/2009/12/28/5092770.aspx#)

01. /**
02. * 加载数据
03. * @param $tag_args
04. */
05. function _complie_load_start($tag_args)
06. {
07. $key = substr(md5($tag_args), 8, 16); //根据参数生成一个特殊的变量名
08. $attrs = $this->_parse_attrs($tag_args);
10. //这里可以增加更多的处理
11. $class = (!isset($attrs[‘class’]) || emptyempty($attrs[‘class’])) ? ‘cls_crud’ : trim($attrs[‘class’]);
12. (!isset($attrs[‘table’]) || emptyempty($attrs[‘table’])) && exit(‘`table` is empty!’);
13. $db = $class::factory(array(‘table’ => substr($attrs[‘table’], 1, -1)));
15. //定义新变量
16. $this->_tpl_vars[$key] = $db->get_block_list(array(substr($attrs[‘where’], 1, -1)), $attrs[‘limit’]);
17. $tag_args = “from=\${$key} “ . $tag_args;
19. //调用foreach标签处理函数进行处理
20. return$this->_compile_foreach_start($tag_args);
21. }

这样就可以在模板中使用load这个标签了。用法例

如：

[view\
\
plain](http://blog.csdn.net/yycai/archive/2009/12/28/5092770.aspx#) [copy\
\
to clipboard](http://blog.csdn.net/yycai/archive/2009/12/28/5092770.aspx#) [print](http://blog.csdn.net/yycai/archive/2009/12/28/5092770.aspx#) [?](http://blog.csdn.net/yycai/archive/2009/12/28/5092770.aspx#)

1. {load table=“test” where=“`id`<100” limit=10 item=rec}
2. …
3. {/load}

[http://code.google.com /p/cyy0523xc/source/browse/trunk/php/在smarty中增加类似foreach的功能自动加载数据.txt](http://code.google.com/p/cyy0523xc/source/browse/trunk/php/%E5%9C%A8smarty%E4%B8%AD%E5%A2%9E%E5%8A%A0%E7%B1%BB%E4%BC%BCforeach%E7%9A%84%E5%8A%9F%E8%83%BD%E8%87%AA%E5%8A%A8%E5%8A%A0%E8%BD%BD%E6%95%B0%E6%8D%AE.txt)
来源: [http://blog.csdn.net/yycai/archive/2009/12/28/5092770.aspx](http://blog.csdn.net/yycai/archive/2009/12/28/5092770.aspx)