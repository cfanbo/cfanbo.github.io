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

002. class show_Pager
003.     {
004. protected$_total;                          //记录总数
005. protected$pagesize;                       //每一页显示的记录数
006. public$pages;                         //总页数
007. protected$_cur_page;                    //当前页码
008. protected$offset;                      //记录偏移量
009. protected$pager_Links;                //url连接
010. protected$pernum = 5;                //页码偏移量，这里可随意更改
012. publicfunction __construct($total,$pagesize,$_cur_page)
013.         {
014. $this->_total=$total;
015. $this->pagesize=$pagesize;
016. $this->_offset();
017. $this->_pager();
018. $this->cur_page($_cur_page);
019. $this->link();
020.     }
022. publicfunction _pager()//计算总页数
023.     {
024. return$this->pages = ceil($this->_total/$this->pagesize);
025.     }
028. publicfunction cur_page($_cur_page) //设置页数
029.     {
030. if (isset($_cur_page))
031.            {
032. $this->_cur_page=intval($_cur_page);
033.            }
034. else
035.            {
036. $this->_cur_page=1; //设置为第一页
037.            }
038. return$this->_cur_page;
039.    }
041. //数据库记录偏移量
042. publicfunction _offset()
043.    {
044. return$this->offset=$this->pagesize*($this->_cur_page-1);
045.    }
047. //html连接的标签
048. publicfunction link()
049.   {
050. if ($this->_cur_page == 1 && $this->pages>1)
051.         {
052. //第一页
053. $this->pager_Links = "首 页 | 上一页 | .$PHP_SELF."?pager=".($this->_cur_page+1).">下一页 | .$PHP_SELF."?pager=$this->pages>尾 页";
054.         }
055. elseif($this->_cur_page == $this->pages && $this->pages>1)
056.         {
057. //最后一页
058. $this->pager_Links = ".$PHP_SELF."?pager=1>首 页 | .$PHP_SELF."?pager=".($this->_cur_page-1).">上一页 | 下一页 | 尾 页";
059.         }
060. elseif ($this->_cur_page > 1 && $this->_cur_page <= $this->pages)
061.         {
062. //中间
063. $this->pager_Links = ".$PHP_SELF."?pager=1>首 页 | .$PHP_SELF."?pager=".($this->_cur_page-1).">上一页 | .$PHP_SELF."?pager=".($this->_cur_page+1).">下一页 | .$PHP_SELF."?pager=$this->pages>尾 页";
064.         }
065. return$this->pager_Links;
066.   }
068. //html数字连接的标签
069. publicfunction num_link()
070.   {
071. $setpage  = $this->_cur_page ? ceil($this->_cur_page/$this->pernum) : 1;
072. $pagenum   = ($this->pages > $this->pernum) ? $this->pernum : $this->pages;
073. if ($this->_total  <= $this->pagesize) {
074. $text  = ‘只有一页’;
075.         } else {
076. $text = ‘页数:’.$this->pages.‘ ‘.$this->pagesize.‘个/页 ‘;
077. if ($this->_cur_page > 1) {
078. $text .= ‘[1]..’;
079.             }
080. if ($setpage > 1) {
081. $lastsetid = ($setpage-1)*$this->pernum;
082. $text .= ‘.$lastsetid.‘>[<<]’;
083.             }
084. if ($this->_cur_page > 1) {
085. $pre = $this->_cur_page-1;
086. $text .= ‘.$pre.‘>[<]’;
087.             }
088. $i = ($setpage-1)*$this->pernum;
089. for($j=$i; $j<($i+$pagenum) && $j<$this->pages; $j++) {
090. $newpage = $j+1;
091. if ($this->_cur_page == $j+1) {
092. $text .= ‘ **[‘** **.($j+1).‘]**’;
093.                 } else {
094. $text .= ‘.$newpage.‘>[‘.($j+1).‘]’;
095.                 }
096.             }
097. if ($this->_cur_page < $this->pages){
098. $next = $this->_cur_page+1;
099. $text .= ‘.$next.‘>[>]’;
100.             }
101. if ($setpage < $this->_total) {
102. $nextpre = $setpage*($this->pernum+1);
103. if($nextpre<$this->pages)
104. $text .= ‘.$nextpre.‘>[>>]’;
105.             }
106. if ($this->_cur_page < $this->pages) {
107. $text .= ‘...$this->pages.‘>[‘.$this->pages.‘]’;
108.             }
109.          }
110. return$text;
111.          }
112.     }
113. //$conn,$tpl 全局变量 传入 $table是数据表 $where 是条件语句 $order 是ADSC之类的,$pager_size是每页显示数,$pager是当前页
114. //返回后在摸板里面可以直接使用
115. <{section name=sec loop=$show}>  <{/section}>
116. 数字标签用<{$numlink}>
117. class my_Pager extends show_Pager
118.     {
119. function __construct($table,$where,$order,$pager_size,$pager)
120.         {
121. global$conn;
122. global$tpl;
123. $sql="Select * FROM `$table` $where order by $order desc";
124. $nums=$conn->Execute($sql)->RecordCount();
125. $pager=new show_Pager($nums,$pager_size,$pager);
126. $show=$conn->SelectLimit($sql,$pager_size,$pager->_offset())->GetAll();
127. $tpl->assign("numlink",$pager->num_link());//数字标签
128. $tpl->assign("show",$show);//显示帖子
129.         }
130.     }
131. ?>