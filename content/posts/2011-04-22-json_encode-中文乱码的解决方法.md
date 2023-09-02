---
title: json_encode 中文乱码的解决方法
author: admin
type: post
date: 2011-04-22T11:04:10+00:00
url: /archives/9389
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - json

---
json 是一个很好的数据结构现在已经广泛用在网络数据传输上

php 自身待了两个和json 相关的函数
json_encode 和 json_decode

这两个函数的具体用法 网上有很多相关的文章
本文主要介绍 用json_encode 时 中文无法转换的解决方案

本文假设 文件所用的编码为gb2312；

先写出所需的数组

>

>  $json = array (

0 =>

array (

‘id’ => ’13’,

‘name’ => ‘乒乓球’,

),

1 =>

array (

‘id’ => ’17’,

‘name’ => ‘篮球’,

)

)

?>

>

如果直接用函数json_encode

>

>  echo json_encode($json);

?>

>

结果为：

>

>  [{“id”:”13″,”name”:null},{“id”:”13″,”name”:null}]

?>

>

可以看到汉字没有被转义  都为”**null**“,这是因为json仅仅转义encoding编码.

 所以上面语句应该先转换编码

>

>

>
>

>
>

> foreach ($ajax as $key=>$val)

{

$ajax[$key][‘name’]    = urlencode($val[‘name’]);

}

echo json_encode($json);
>

>
>

> ?>

>

>
>

>
>

>

**客户端js代码**



>

>

function getsort(obj)

{

$.ajax(

{

type : “GET”,

url : “baseUrl?>/index/getajax”,

data : “c=” obj.value,

success : function(json)

{

var json=eval(json);


>
>

> var html = ‘’; $.each(json, function(k) { html  = ‘’   decodeURI(json[k][‘name’])   ‘’; }); html  =””;

$(‘#sort’).html(html);

}

}

)

}


>

>
>

>
>

>
>

>
>

>
>

>
>

>
>

>


>
>

> 注意,这样修改还会提示错误,用上面的代码js会报错 说编码不符合标准
>

>
>

> 原因是因为js 中decodeURI 仅仅支持utf8 转码

所以  php 代码应该为下面的代码
>

>
>

>
>

>
>

> foreach ($ajax as $key=>$val)

{

$ajax[$key][‘name’]    = urlencode(iconv(‘gb2312′,’utf-8’,$val[‘name’]));

}

echo json_encode($json);
>

>
>

> ?>
>


>
>

> 在用的时候发现些标点符号之类的变成了%2c之类的符号，这个是由于url编码的问题引起的．只需要用unescape()就可以解决了．
>

>
>

> 注意：json处理换行的时候有问题，只能用

不能用

这个，或者通过其它的办法．
>