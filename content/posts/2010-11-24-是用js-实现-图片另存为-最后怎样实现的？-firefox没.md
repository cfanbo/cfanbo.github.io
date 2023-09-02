---
title: 是用js 实现 图片“另存为” 最后怎样实现的？ (FireFox没有测试过)
author: admin
type: post
date: 2010-11-24T03:19:29+00:00
url: /archives/6777
IM_contentdowned:
 - 1
categories:
 - 前端设计

---

```
使用JS实现单击连接保存图片
2种形式都可以

第一种：
```

>

```
<script>
function   SaveAs5(imgURL)
...{
  var   oPop   =   window.open(imgURL,"","width=1,   height=1,   top=5000,   left=5000");
  for(;   oPop.document.readyState   !=   "complete";   )
  ...{
    if   (oPop.document.readyState   ==   "complete")break;
  }
  oPop.document.execCommand("SaveAs");
  oPop.close();
}
</script>
```

```
<img   src="t_screenshot_17616.jpg"   id="DemoImg"   border="0"   onclick="SaveAs5(this.src)">

第二种：
```

>

```
<script>
function   SaveAs5(imgURL)
...{
  var   oPop   =   window.open(imgURL,"","width=1,   height=1,   top=5000,   left=5000");
  for(;   oPop.document.readyState   !=   "complete";   )
  ...{
    if   (oPop.document.readyState   ==   "complete")break;
  }
  oPop.document.execCommand("SaveAs");
  oPop.close();
}
</script>
```

```
<img   src="../t_screenshot_17616.jpg"   id="DemoImg"   border="0">
<a   href="#"     onclick="SaveAs5(document.getElementById('DemoImg').src)"> 点击这里下载图片 </a>
```