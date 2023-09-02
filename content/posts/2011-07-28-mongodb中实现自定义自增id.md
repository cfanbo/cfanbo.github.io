---
title: mongodb中实现自定义自增ID
author: admin
type: post
date: 2011-07-28T04:15:41+00:00
url: /archives/10726
IM_contentdowned:
 - 1
categories:
 - nosql
tags:
 - MongoDB
 - nosql

---
PHP代码:

```
function get_autoincre_id($name, $db){
	$update = array('$inc'=>array("id"=>1));
	$query = array('table_name' => $name);
	$command = array(
		'findandmodify'=>'autoincre_system', 'update'=>$update,
		'query'=>$query, 'new'=>true, 'upsert'=>true
	);
	$id = $db->command($command);
	return $id['value']['id'];
}
```

其中上面的table_name可以用来区别多个表,这样可以灵活实现单独的几个表的自增ID值.

> $conn = new Mongo();
> $db = $conn->cms;
>
> news_max_id = get\_autoincre\_id(‘tbl_news’); //从1开始
>
> soft_max_id = get\_autoincre\_id(‘tbl_soft’); //从1开始

现在就可以直接在写入新记录的时候使用这个id值了.

其具体实现方式主要是利用MongoDB中 [findAndModify](http://www.mongodb.org/display/DOCS/findAndModify+Command) 命令，