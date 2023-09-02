---
title: bootstrap自动完成插件Typeahead.js的用法之ajax请求
author: admin
type: post
date: 2017-08-23T04:05:27+00:00
url: /archives/17456
categories:
 - 前端设计
tags:
 - jquery插件

---
官方对Typeahead.js的介绍太简单了，对于新手来说有些吃力，项目中使用了这个插件，困惑了两天，终于才搞定它的ajax用法。官方示例虽然可以使用，但从remote url 获取数据的时候，无法传递过多的自定义参数，如要根据省份、城市信息来获取当前城市的小区信息，这个时候我们就要想办法把省份和城市信息也同时发给后端才可以。这样可以过滤大部分的无用信息，产少数据量的产生。

```

```

js代码

```
$(function(){
/*
	var filterResult = new Bloodhound({
		datumTokenizer: Bloodhound.tokenizers.whitespace,
		queryTokenizer: Bloodhound.tokenizers.whitespace,
		initialize: false,
		remote: {
			url: '/getCommunityList?province_code=' + $("#province_code").val() + '&q=%QUERY',
			wildcard: '%QUERY'
		}
	});
*/

	$('#community_name').typeahead({
			highlight: true,
			hint: true,
			minLength: 1
		},
		{

			name: 'result-list',
			limit: 10,
			async: true,
			source: function (query, processSync, processAsync) {
					//processSync(['This suggestion appears immediately', 'This one too']);
					var params = {
						province_code: $("#province_code").val(),
						city_code: $("#city_code").val(),
						area_code: $("#area_code").val(),
						q: query
					};

					if (params.province_code == '' || params.city_code == '' || params.area_code == '' || params.q == '') {
						return false;
					}

					$.post('/getCommunityList', params, function(json) {
						if (json.status == '0') {
							layer.msg(json.info);
						} else {
							return processAsync(json.data);
						}
					});
                },
			display:function(item){
						return item.name
					},
			templates: {
			  suggestion: function (item){
							return '

  ' + item.name + '
';
						},
			  notFound: '

  未找到小区，请确认区域信息!
',
			  header: '

  可使用键盘↑↓选择。
',
//			  footer: '

  支持剪头移动选择项
'
			}
		}

	);

	//结果项被选中事件
	$('#community_name').bind('typeahead:select', function(ev, suggestion) {
		$("#address").html(suggestion.address);

//	  	console.log(suggestion);
	});

	//鼠标移动到选择项事件
	$('#community_name').bind('typeahead:cursorchange', function(ev, suggestion) {
//	  	console.log(suggestion);
	});
	$('#community_name').bind('typeahead:autocomplete', function(ev, suggestion) {
	  	console.log(suggestion);
	});

})

```

注意上面的 **async: true** 配置，还有source回调函数要使用三个参数。