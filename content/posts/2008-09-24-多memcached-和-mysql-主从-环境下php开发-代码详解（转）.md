---
title: '多memcached 和 mysql 主从 环境下PHP开发: 代码详解（转）'
author: admin
type: post
date: 2008-09-24T11:48:29+00:00
excerpt: |
 |
 一般的大站通常做法是 拿着内存当数据库来用(memcached). 和很好的读 写分离 备份机制 (mysql 的主从)

 在这样的环境下我们怎么进行PHP开发呢. 本人不太会讲话.所以还是帖代码吧.

 刚在linux 的 VIM 里写的一个 demo 调试通过. 也同时希望大家拍砖 , 使用PHP5 写的. PHP4写出来怕大家说我落后了
url: /archives/383
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - memcached
 - mysql
 - php

---
## 多memcached 和 mysql 主从 环境下PHP开发: 代码详解

4点了.今天是最后一天在这间公司.心情不是很好.

所以写下东西发泄下.
    一般的大站通常做法是    拿着内存当数据库来用(memcached).     和很好的读  写分离  备份机制 (mysql 的主从)

在这样的环境下我们怎么进行PHP开发呢.  本人不太会讲话.所以还是帖代码吧.

刚在linux  的 VIM  里写的一个  demo    调试通过.  也同时希望大家拍砖 ,  使用PHP5  写的. PHP4写出来怕大家说我落后了

##### PHP代码:

`<span style="color: #000000;"><br />
<span style="color: #0000bb;"><?php<br />
$memcached </span><span style="color: #007700;">= array(  </span><span style="color: #ff8000;">//用memcached 的 多 进程模拟 多台memcached 服务器     cn     en   为  内存服务器名<br />
     </span><span style="color: #dd0000;">'cn'</span><span style="color: #007700;">=>array(</span><span style="color: #dd0000;">'192.168.254.144'</span><span style="color: #007700;">,</span><span style="color: #0000bb;">11211</span><span style="color: #007700;">),<br />
     </span><span style="color: #dd0000;">'en'</span><span style="color: #007700;">=>array(</span><span style="color: #dd0000;">'192.168.254.144'</span><span style="color: #007700;">,</span><span style="color: #0000bb;">11212</span><span style="color: #007700;">)<br />
     );<br />
</span><span style="color: #0000bb;">$mysql    </span><span style="color: #007700;">= array( </span><span style="color: #ff8000;">// mysql 的主从 我的环境是 ： xp 主    linux 从  mysql 5  php5<br />
     </span><span style="color: #dd0000;">'master'</span><span style="color: #007700;">=>array(</span><span style="color: #dd0000;">'192.168.254.213'</span><span style="color: #007700;">,</span><span style="color: #dd0000;">'root'</span><span style="color: #007700;">,</span><span style="color: #dd0000;">'1'</span><span style="color: #007700;">,</span><span style="color: #dd0000;">'mydz'</span><span style="color: #007700;">),<br />
     </span><span style="color: #dd0000;">'slave_1'</span><span style="color: #007700;">=>array(</span><span style="color: #dd0000;">'192.168.254.144'</span><span style="color: #007700;">,</span><span style="color: #dd0000;">'root'</span><span style="color: #007700;">,</span><span style="color: #dd0000;">'1'</span><span style="color: #007700;">,</span><span style="color: #dd0000;">'mydz'</span><span style="color: #007700;">)  </span><span style="color: #ff8000;">//可以灵活添加多台从服务器<br />
     </span><span style="color: #007700;">);<br />
</span><span style="color: #0000bb;">?></span><br />
</span><br />
`

服务器配置文件: 十分方便的 切换主从.  当主换了  从可以迅速切换为主.  支持 多从服务器   .


复制PHP内容到剪贴板

##### PHP代码:

`<span style="color: #000000;"><br />
<span style="color: #0000bb;"><?php<br />
</span><span style="color: #007700;">class </span><span style="color: #0000bb;">Memcached<br />
</span><span style="color: #007700;">{<br />
private </span><span style="color: #0000bb;">$mem</span><span style="color: #007700;">;<br />
public </span><span style="color: #0000bb;">$pflag</span><span style="color: #007700;">=</span><span style="color: #dd0000;">''</span><span style="color: #007700;">; </span><span style="color: #ff8000;">// memcached pconnect tag<br />
</span><span style="color: #007700;">private function </span><span style="color: #0000bb;">memConnect</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$serkey</span><span style="color: #007700;">){<br />
require </span><span style="color: #dd0000;">'config.php'</span><span style="color: #007700;">;<br />
</span><span style="color: #0000bb;">$server </span><span style="color: #007700;">= </span><span style="color: #0000bb;">$memcached</span><span style="color: #007700;">;<br />
</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">mem </span><span style="color: #007700;">= new </span><span style="color: #0000bb;">Memcache</span><span style="color: #007700;">;<br />
</span><span style="color: #0000bb;">$link </span><span style="color: #007700;">= !</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">pflag </span><span style="color: #007700;">? </span><span style="color: #dd0000;">'connect' </span><span style="color: #007700;">: </span><span style="color: #dd0000;">'pconnect' </span><span style="color: #007700;">;<br />
</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">mem</span><span style="color: #007700;">-></span><span style="color: #0000bb;">$link</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$server</span><span style="color: #007700;">[</span><span style="color: #0000bb;">$serkey</span><span style="color: #007700;">][</span><span style="color: #0000bb;">0</span><span style="color: #007700;">],</span><span style="color: #0000bb;">$server</span><span style="color: #007700;">[</span><span style="color: #0000bb;">$serkey</span><span style="color: #007700;">][</span><span style="color: #0000bb;">1</span><span style="color: #007700;">]) or </span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">errordie</span><span style="color: #007700;">(</span><span style="color: #dd0000;">'memcached connect error'</span><span style="color: #007700;">);</p>
<p>}</p>
<p>public function </span><span style="color: #0000bb;">set</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$ser_key</span><span style="color: #007700;">,</span><span style="color: #0000bb;">$values</span><span style="color: #007700;">,</span><span style="color: #0000bb;">$flag</span><span style="color: #007700;">=</span><span style="color: #dd0000;">''</span><span style="color: #007700;">,</span><span style="color: #0000bb;">$expire</span><span style="color: #007700;">=</span><span style="color: #dd0000;">''</span><span style="color: #007700;">){<br />
</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">memConnect</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">tag</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$ser_key</span><span style="color: #007700;">));<br />
if(</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">mem</span><span style="color: #007700;">-></span><span style="color: #0000bb;">set</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$ser_key</span><span style="color: #007700;">,</span><span style="color: #0000bb;">$values</span><span style="color: #007700;">,</span><span style="color: #0000bb;">$flag</span><span style="color: #007700;">,</span><span style="color: #0000bb;">$expire</span><span style="color: #007700;">)) return </span><span style="color: #0000bb;">true</span><span style="color: #007700;">;<br />
else return </span><span style="color: #0000bb;">false</span><span style="color: #007700;">;<br />
} </p>
<p>public function </span><span style="color: #0000bb;">get</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$ser_key</span><span style="color: #007700;">){<br />
</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">memConnect</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">tag</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$ser_key</span><span style="color: #007700;">));<br />
if(</span><span style="color: #0000bb;">$var</span><span style="color: #007700;">=</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">mem</span><span style="color: #007700;">-></span><span style="color: #0000bb;">get</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$ser_key</span><span style="color: #007700;">)) return </span><span style="color: #0000bb;">$var</span><span style="color: #007700;">;<br />
else return </span><span style="color: #0000bb;">false</span><span style="color: #007700;">; <br />
} <br />
private function </span><span style="color: #0000bb;">tag</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$ser_key</span><span style="color: #007700;">){<br />
</span><span style="color: #0000bb;">$tag</span><span style="color: #007700;">=</span><span style="color: #0000bb;">explode</span><span style="color: #007700;">(</span><span style="color: #dd0000;">'_'</span><span style="color: #007700;">,</span><span style="color: #0000bb;">$ser_key</span><span style="color: #007700;">);<br />
return </span><span style="color: #0000bb;">$tag</span><span style="color: #007700;">[</span><span style="color: #0000bb;">0</span><span style="color: #007700;">];<br />
}<br />
private function </span><span style="color: #0000bb;">errordie</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$errmsg</span><span style="color: #007700;">){<br />
die(</span><span style="color: #0000bb;">$errmsg</span><span style="color: #007700;">);<br />
}<br />
}<br />
</span><span style="color: #0000bb;">?></span><br />
</span><br />
`

简单的封装了 memcached  的操作. 详细的时间不多.我要离开公司了


在memcached 的多服务器上.  我的实现思路是这样的:   在把信息添加到 内存服务器的时候.我选择了手工设置添加到那个服务器.而不用传统的根据ID自动分配.

这样可以更灵活点.


以内存服务器名 为表示   比如 存  $arr 这个信息到  en 这台 内存服务器 我就这样写   $mem->set(‘en_’.$arr);    明白了吧


##### PHP代码:

`<span style="color: #000000;"><br />
<span style="color: #0000bb;"><?php<br />
</span><span style="color: #007700;">class </span><span style="color: #0000bb;">Mysql<br />
</span><span style="color: #007700;">{<br />
private   </span><span style="color: #0000bb;">$mysqlmaster</span><span style="color: #007700;">;<br />
private   </span><span style="color: #0000bb;">$myssqlslave</span><span style="color: #007700;">;<br />
private static </span><span style="color: #0000bb;">$auid</span><span style="color: #007700;">=</span><span style="color: #0000bb;">0</span><span style="color: #007700;">;<br />
public function </span><span style="color: #0000bb;">__construct</span><span style="color: #007700;">(){<br />
require </span><span style="color: #dd0000;">'config.php'</span><span style="color: #007700;">;<br />
</span><span style="color: #0000bb;">$msg </span><span style="color: #007700;">= </span><span style="color: #0000bb;">$mysql</span><span style="color: #007700;">;<br />
<br />
</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">mysqlmaster </span><span style="color: #007700;">= new </span><span style="color: #0000bb;">mysqli</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$msg</span><span style="color: #007700;">[</span><span style="color: #dd0000;">'master'</span><span style="color: #007700;">][</span><span style="color: #0000bb;">0</span><span style="color: #007700;">],</span><span style="color: #0000bb;">$msg</span><span style="color: #007700;">[</span><span style="color: #dd0000;">'master'</span><span style="color: #007700;">][</span><span style="color: #0000bb;">1</span><span style="color: #007700;">],</span><span style="color: #0000bb;">$msg</span><span style="color: #007700;">[</span><span style="color: #dd0000;">'master'</span><span style="color: #007700;">][</span><span style="color: #0000bb;">2</span><span style="color: #007700;">],</span><span style="color: #0000bb;">$msg</span><span style="color: #007700;">[</span><span style="color: #dd0000;">'master'</span><span style="color: #007700;">][</span><span style="color: #0000bb;">3</span><span style="color: #007700;">]); </span><span style="color: #ff8000;">//master mysql<br />
</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">mysqlslave  </span><span style="color: #007700;">= </span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">autotranscat</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$msg</span><span style="color: #007700;">); </span><span style="color: #ff8000;">// slave mysql</p>
<p>  </span><span style="color: #007700;">if(</span><span style="color: #0000bb;">mysqli_connect_errno</span><span style="color: #007700;">()){<br />
</span><span style="color: #0000bb;">printf</span><span style="color: #007700;">(</span><span style="color: #dd0000;">"Connect failed: %s\n"</span><span style="color: #007700;">,</span><span style="color: #0000bb;">mysqli_connect_error</span><span style="color: #007700;">());<br />
exit();<br />
}<br />
if(!</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">mysqlmaster</span><span style="color: #007700;">-></span><span style="color: #0000bb;">set_charset</span><span style="color: #007700;">(</span><span style="color: #dd0000;">"latin1"</span><span style="color: #007700;">) && !</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">mysqlslave</span><span style="color: #007700;">-></span><span style="color: #0000bb;">set_charset</span><span style="color: #007700;">(</span><span style="color: #dd0000;">"latin1"</span><span style="color: #007700;">)){<br />
exit(</span><span style="color: #dd0000;">"set charset error"</span><span style="color: #007700;">);<br />
}<br />
}</p>
<p>private function </span><span style="color: #0000bb;">autotranscat</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$mysql</span><span style="color: #007700;">){<br />
</span><span style="color: #0000bb;">session_start</span><span style="color: #007700;">();<br />
</span><span style="color: #0000bb;">$_SESSION</span><span style="color: #007700;">[</span><span style="color: #dd0000;">'SID'</span><span style="color: #007700;">]!=</span><span style="color: #0000bb;">0 </span><span style="color: #007700;">|| </span><span style="color: #0000bb;">$_SESSION</span><span style="color: #007700;">[</span><span style="color: #dd0000;">'SID'</span><span style="color: #007700;">]=</span><span style="color: #0000bb;">0   </span><span style="color: #007700;">;<br />
if(</span><span style="color: #0000bb;">$_SESSION</span><span style="color: #007700;">[</span><span style="color: #dd0000;">'SID'</span><span style="color: #007700;">] >=</span><span style="color: #0000bb;">count</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$mysql</span><span style="color: #007700;">)-</span><span style="color: #0000bb;">1</span><span style="color: #007700;">) </span><span style="color: #0000bb;">$_SESSION</span><span style="color: #007700;">[</span><span style="color: #dd0000;">'SID'</span><span style="color: #007700;">] = </span><span style="color: #0000bb;">1</span><span style="color: #007700;">;<br />
else </span><span style="color: #0000bb;">$_SESSION</span><span style="color: #007700;">[</span><span style="color: #dd0000;">'SID'</span><span style="color: #007700;">]++;<br />
</span><span style="color: #0000bb;">$key </span><span style="color: #007700;">= </span><span style="color: #dd0000;">'slave_'</span><span style="color: #007700;">.</span><span style="color: #0000bb;">$_SESSION</span><span style="color: #007700;">[</span><span style="color: #dd0000;">'SID'</span><span style="color: #007700;">];<br />
echo(</span><span style="color: #0000bb;">$_SESSION</span><span style="color: #007700;">[</span><span style="color: #dd0000;">'SID'</span><span style="color: #007700;">]);<br />
return new </span><span style="color: #0000bb;">mysqli</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$mysql</span><span style="color: #007700;">[</span><span style="color: #0000bb;">$key</span><span style="color: #007700;">][</span><span style="color: #0000bb;">0</span><span style="color: #007700;">],</span><span style="color: #0000bb;">$mysql</span><span style="color: #007700;">[</span><span style="color: #0000bb;">$key</span><span style="color: #007700;">][</span><span style="color: #0000bb;">1</span><span style="color: #007700;">],</span><span style="color: #0000bb;">$mysql</span><span style="color: #007700;">[</span><span style="color: #0000bb;">$key</span><span style="color: #007700;">][</span><span style="color: #0000bb;">2</span><span style="color: #007700;">],</span><span style="color: #0000bb;">$mysql</span><span style="color: #007700;">[</span><span style="color: #0000bb;">$key</span><span style="color: #007700;">][</span><span style="color: #0000bb;">3</span><span style="color: #007700;">]);<br />
}</p>
<p>public function </span><span style="color: #0000bb;">mquery</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$sql</span><span style="color: #007700;">){ </span><span style="color: #ff8000;">//insert  update <br />
</span><span style="color: #007700;">if(!</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">mysqlmaster</span><span style="color: #007700;">-></span><span style="color: #0000bb;">query</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$sql</span><span style="color: #007700;">)){<br />
return </span><span style="color: #0000bb;">false</span><span style="color: #007700;">;<br />
}<br />
}</p>
<p>public function </span><span style="color: #0000bb;">squery</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$sql</span><span style="color: #007700;">){<br />
if(</span><span style="color: #0000bb;">$result</span><span style="color: #007700;">=</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">mysqlslave</span><span style="color: #007700;">-></span><span style="color: #0000bb;">query</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$sql</span><span style="color: #007700;">)){<br />
return </span><span style="color: #0000bb;">$result</span><span style="color: #007700;">; <br />
}else{<br />
return </span><span style="color: #0000bb;">false</span><span style="color: #007700;">;<br />
};<br />
}<br />
public function </span><span style="color: #0000bb;">fetArray</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$sql</span><span style="color: #007700;">){<br />
if(</span><span style="color: #0000bb;">$result</span><span style="color: #007700;">=</span><span style="color: #0000bb;">$this</span><span style="color: #007700;">-></span><span style="color: #0000bb;">squery</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$sql</span><span style="color: #007700;">)){<br />
while(</span><span style="color: #0000bb;">$row</span><span style="color: #007700;">=</span><span style="color: #0000bb;">$result</span><span style="color: #007700;">-></span><span style="color: #0000bb;">fetch_array</span><span style="color: #007700;">(</span><span style="color: #0000bb;">MYSQLI_ASSOC</span><span style="color: #007700;">)){<br />
    </span><span style="color: #0000bb;">$resultraa</span><span style="color: #007700;">[] = </span><span style="color: #0000bb;">$row</span><span style="color: #007700;">;<br />
};<br />
return </span><span style="color: #0000bb;">$resultraa</span><span style="color: #007700;">;<br />
}<br />
}<br />
}<br />
</span><span style="color: #0000bb;">?></span><br />
</span><br />
`

这个是 mysqli 的封装.  也就是   读  从  写 主  的操作的封装.


复制PHP内容到剪贴板

##### PHP代码:

`<span style="color: #000000;"><br />
<span style="color: #0000bb;"><?php<br />
</span><span style="color: #007700;">require </span><span style="color: #dd0000;">'init.php'</span><span style="color: #007700;">;<br />
</span><span style="color: #0000bb;">$mem </span><span style="color: #007700;">= new </span><span style="color: #0000bb;">Memcached</span><span style="color: #007700;">;<br />
</span><span style="color: #ff8000;">/* $mem->set('en_xx','bucuo');<br />
echo($mem->get('en_xx'));<br />
$mem->set('cn_jjyy','wokao');<br />
echo($mem->get('cn_jjyy'));<br />
*/ <br />
</span><span style="color: #0000bb;">$sq </span><span style="color: #007700;">= new </span><span style="color: #0000bb;">Mysql</span><span style="color: #007700;">;<br />
</span><span style="color: #0000bb;">$sql </span><span style="color: #007700;">= </span><span style="color: #dd0000;">"insert into mybb(pid) values(200)"</span><span style="color: #007700;">;<br />
</span><span style="color: #0000bb;">$mdsql </span><span style="color: #007700;">= </span><span style="color: #0000bb;">md5</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$sql</span><span style="color: #007700;">);<br />
if(!</span><span style="color: #0000bb;">$result</span><span style="color: #007700;">=</span><span style="color: #0000bb;">$mem</span><span style="color: #007700;">-></span><span style="color: #0000bb;">get</span><span style="color: #007700;">(</span><span style="color: #dd0000;">'cn_'</span><span style="color: #007700;">.</span><span style="color: #0000bb;">$mdsql</span><span style="color: #007700;">)){<br />
</span><span style="color: #0000bb;">$sq</span><span style="color: #007700;">-></span><span style="color: #0000bb;">mquery</span><span style="color: #007700;">(</span><span style="color: #dd0000;">"insert into mybb(pid) values(200)"</span><span style="color: #007700;">); </span><span style="color: #ff8000;">//插入到主mysql<br />
</span><span style="color: #0000bb;">$result </span><span style="color: #007700;">= </span><span style="color: #0000bb;">$sq</span><span style="color: #007700;">-></span><span style="color: #0000bb;">fetArray</span><span style="color: #007700;">(</span><span style="color: #dd0000;">"select * from mybb"</span><span style="color: #007700;">); </span><span style="color: #ff8000;">//查询 是 从mysql<br />
</span><span style="color: #007700;">foreach(</span><span style="color: #0000bb;">$result </span><span style="color: #007700;">as </span><span style="color: #0000bb;">$var</span><span style="color: #007700;">){<br />
echo </span><span style="color: #0000bb;">$var</span><span style="color: #007700;">[</span><span style="color: #dd0000;">'pid'</span><span style="color: #007700;">];<br />
}<br />
</span><span style="color: #0000bb;">$mem</span><span style="color: #007700;">-></span><span style="color: #0000bb;">set</span><span style="color: #007700;">(</span><span style="color: #dd0000;">'cn_'</span><span style="color: #007700;">.</span><span style="color: #0000bb;">$mdsql</span><span style="color: #007700;">,</span><span style="color: #0000bb;">$result</span><span style="color: #007700;">); </span><span style="color: #ff8000;">//添加到 名为 cn 的 memcached 服务器  <br />
</span><span style="color: #007700;">}else{<br />
foreach(</span><span style="color: #0000bb;">$result </span><span style="color: #007700;">as </span><span style="color: #0000bb;">$var</span><span style="color: #007700;">){<br />
echo </span><span style="color: #0000bb;">$var</span><span style="color: #007700;">[</span><span style="color: #dd0000;">'pid'</span><span style="color: #007700;">];<br />
}<br />
}<br />
</span><span style="color: #0000bb;">?></span><br />
</span><br />
`

这个是使用程序.   看下就大概明白了.