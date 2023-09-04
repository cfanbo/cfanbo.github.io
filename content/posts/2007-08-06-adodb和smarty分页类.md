---
title: adodb和smarty分页类
author: admin
type: post
date: 2007-08-06T23:14:46+00:00
url: /archives/78
IM_contentdowned:
 - 1
categories:
 - 程序开发

---

```
class show_Pager

{

protected$_total;                          //记录总数

protected$pagesize;                       //每一页显示的记录数

public$pages;                         //总页数

protected$_cur_page;                    //当前页码

protected$offset;                      //记录偏移量

protected$pager_Links;                //url连接

protected$pernum = 5;                //页码偏移量，这里可随意更改

publicfunction __construct($total,$pagesize,$_cur_page)

{

$this->_total=$total;

$this->pagesize=$pagesize;

$this->_offset();

$this->_pager();

$this->cur_page($_cur_page);

$this->link();

}

publicfunction _pager()//计算总页数

{

return$this->pages = ceil($this->_total/$this->pagesize);

}

publicfunction cur_page($_cur_page) //设置页数

{

if (isset($_cur_page))

{

$this->_cur_page=intval($_cur_page);

}

else

{

$this->_cur_page=1; //设置为第一页

}

return$this->_cur_page;

}

//数据库记录偏移量

publicfunction _offset()

{

return$this->offset=$this->pagesize*($this->_cur_page-1);

}

//html连接的标签

publicfunction link()

{

if ($this->_cur_page == 1 && $this->pages>1)

{

//第一页

$this->pager_Links = "首 页 | 上一页 | .$PHP_SELF."?pager=".($this->_cur_page+1).">下一页 | .$PHP_SELF."?pager=$this->pages>尾 页";

}

elseif($this->_cur_page == $this->pages && $this->pages>1)

{

//最后一页

$this->pager_Links = ".$PHP_SELF."?pager=1>首 页 | .$PHP_SELF."?pager=".($this->_cur_page-1).">上一页 | 下一页 | 尾 页";

}

elseif ($this->_cur_page > 1 && $this->_cur_page <= $this->pages)

{

//中间

$this->pager_Links = ".$PHP_SELF."?pager=1>首 页 | .$PHP_SELF."?pager=".($this->_cur_page-1).">上一页 | .$PHP_SELF."?pager=".($this->_cur_page+1).">下一页 | .$PHP_SELF."?pager=$this->pages>尾 页";

}

return$this->pager_Links;

}

//html数字连接的标签

publicfunction num_link()

{

$setpage  = $this->_cur_page ? ceil($this->_cur_page/$this->pernum) : 1;

$pagenum   = ($this->pages > $this->pernum) ? $this->pernum : $this->pages;

if ($this->_total  <= $this->pagesize) {

$text  = ‘只有一页’;

} else {

$text = ‘页数:’.$this->pages.‘ ‘.$this->pagesize.‘个/页 ‘;

if ($this->_cur_page > 1) {

$text .= ‘[1]..’;

}

if ($setpage > 1) {

$lastsetid = ($setpage-1)*$this->pernum;

$text .= ‘.$lastsetid.‘>[<<]’;

}

if ($this->_cur_page > 1) {

$pre = $this->_cur_page-1;

$text .= ‘.$pre.‘>[<]’;

}

$i = ($setpage-1)*$this->pernum;

for($j=$i; $j<($i+$pagenum) && $j<$this->pages; $j++) {

$newpage = $j+1;

if ($this->_cur_page == $j+1) {

$text .= ‘ **[‘** **.($j+1).‘]**’;

} else {

$text .= ‘.$newpage.‘>[‘.($j+1).‘]’;

}

}

if ($this->_cur_page < $this->pages){

$next = $this->_cur_page+1;

$text .= ‘.$next.‘>[>]’;

}

if ($setpage < $this->_total) {

$nextpre = $setpage*($this->pernum+1);

if($nextpre<$this->pages)

$text .= ‘.$nextpre.‘>[>>]’;

}

if ($this->_cur_page < $this->pages) {

$text .= ‘...$this->pages.‘>[‘.$this->pages.‘]’;

}

}

return$text;

}

}
```



//$conn,$tpl 全局变量 传入 $table是数据表 $where 是条件语句 $order 是ADSC之类的,$pager_size是每页显示数,$pager是当前页

//返回后在摸板里面可以直接使用

<{section name=sec loop=$show}>  <{/section}>

数字标签用<{$numlink}>

```
class my_Pager extends show_Pager

{

function __construct($table,$where,$order,$pager_size,$pager)

{

global$conn;

global$tpl;

$sql="Select * FROM `$table` $where order by $order desc";

$nums=$conn->Execute($sql)->RecordCount();

$pager=new show_Pager($nums,$pager_size,$pager);

$show=$conn->SelectLimit($sql,$pager_size,$pager->_offset())->GetAll();

$tpl->assign("numlink",$pager->num_link());//数字标签

$tpl->assign("show",$show);//显示帖子

}

}

?>
```

