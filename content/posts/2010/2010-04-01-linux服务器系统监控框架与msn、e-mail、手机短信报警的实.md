---
title: 'Linux服务器系统监控框架与MSN、E-mail、手机短信报警的实现[原创]'
author: admin
type: post
date: 2010-04-01T16:27:24+00:00
url: /archives/3228
IM_data:
 - 'a:2:{s:58:"http://blog.s135.com/attachment/200806/server_monitor1.png";s:75:"http://blog.haohtml.com/wp-content/uploads/2011/03/d1b7_server_monitor1.png";s:58:"http://blog.s135.com/attachment/200806/server_monitor2.png";s:75:"http://blog.haohtml.com/wp-content/uploads/2011/03/b6cf_server_monitor2.png";}'
IM_contentdowned:
 - 1
categories:
 - 服务器

---
[文章作者：张宴 本文版本：v1.0 最后修改：2008.06.25 转载请注明原文链接： [http://blog.s135.com/read.php/354.htm](http://blog.s135.com/read.php/354.htm)]

最近，在我原有的“Linux服务器系统监控程序”基础上，完善了HTTP、TCP、MySQL主动监控与MSN、E-mail、手机短信报警。监控程 序以shell和PHP程序编写，以下为主要框架与部分代码：

一、系统监控接口程序（interface.php）具有的报警方式
１、MSN实时报警
①、监控程序每次检测到故障存在、或者故障恢复，都会发送短消息到管理员的MSN。

[![点击在新窗口中浏览此图片](http://blog.s135.com/attachment/200806/server_monitor1.png)](http://blog.s135.com/attachment/200806/server_monitor1.png)[![点击在新窗口中浏览此图片](http://blog.s135.com/attachment/200806/server_monitor2.png)](http://blog.s135.com/attachment/200806/server_monitor2.png)

发送MSN短消息用了一个PHP类： [sendMsg](http://www.fanatic.net.nz/2005/02/15/send-a-message-using-php/)，使用该PHP类发消息，必须将发送、接收双方的MSN加为联系人，发送中文时，先用iconv 将字符集转为UTF-8：

引用


$sendMsg->sendMessage(iconv(“GBK”, “UTF-8”, $message), ‘Times New Roman’, ‘008000’);


* * *

２、 手机短信报警
①、工作日早上10点之前，晚上6点之后，以及周六、周日，监控程序检测到故障，会调用手机短信接口，发送短信给管理员的手机。
②、如果监控程序多次检测到同一台服务器的同一类故障，只会在第一次检测到故障时发送一条“故障报警”短信。服务器故障恢复后，监控程序会再发送一条 “故障恢复”短信。

如果没有手机短信网关接口，可以试试中国移动通信的 [www.139.com](http://www.139.com/) 邮箱，具有免 费的邮件到达手机短信通知功能，可以将收到的邮件标题以短信的形式发送到手机上。

* * *

３、电子邮件报警
①、如果监控程序多次检测到同一台服务器的同一类故障，只会在第一次检测到故障时发送一封“故障报警”邮件。服务器故障恢复后，监控程序会再发送一封 “故障恢复”邮件。

系统监控接口程序interface.php（核心部分，仅提供部分代码）：

[view plain](http://blog.s135.com/#) [print](http://blog.s135.com/#) [?](http://blog.s135.com/#)

02. //HTTP服务器监控
03. if (htmlspecialchars($_POST[“menu”]) == “http”)
04. {
05. $date = htmlspecialchars($_POST[“date”]);
06. $ip = htmlspecialchars($_POST[“ip”]);
07. $port = htmlspecialchars($_POST[“port”]);
08. $status = htmlspecialchars($_POST[“status”]);//状态，0表示无法访问，1表示正常，2表示无法访问但能ping通
09. //…下一步处理（省略）…
10. }
12. //TCP服务器监控
13. if (htmlspecialchars($_POST[“menu”]) == “tcp”)
14. {
15. $date = htmlspecialchars($_POST[“date”]);
16. $ip = htmlspecialchars($_POST[“ip”]);
17. $port = htmlspecialchars($_POST[“port”]);
18. $status = htmlspecialchars($_POST[“status”]);//状态，0表示无法访问，1表示正常，2表示无法访问但能ping通
19. //…下一步处理（省略）…
20. }
22. //MySQL服务器监控
23. if (htmlspecialchars($_POST[“menu”]) == “mysql”)
24. {
25. $date = htmlspecialchars($_POST[“date”]);
26. $ip = htmlspecialchars($_POST[“ip”]);
27. $port = htmlspecialchars($_POST[“port”]);
28. $abstract = htmlspecialchars($_POST[“abstract”]);//故障摘要（必须为全角）
29. $info = htmlspecialchars($_POST[“info”]);//故障详细描述
30. $failback = htmlspecialchars($_POST[“failback”]);//如果服务器存活，此处接收的值为active
31. //…下一步处理（省略）…
32. }
33. ?>

* * *

二、主动探测监控（“监控机”主动探测“被监控机”）
１、HTTP服务器监控
脚本：/data0/monitor/http.sh

引用


#!/bin/sh

LANG=C

# 被监控服务器、端口列表

server_all_list=(\

192.168.1.1:80 \

192.168.1.2:80 \

192.168.1.3:80 \

)


date=$(date -d “today” +”%Y-%m-%d_%H:%M:%S”)


#采用 HTTP POST方式发送检测信息给接口程序interface.php，接口程序负责分析信息，决定是否发送报警MSN消息、手机短信、电子邮件。

send_msg_to_interface()

{

/usr/bin/curl -m 600 -d menu=http -d date=$date -d ip=$server_ip -d port=$server_port -d status=$status [http://127.0.0.1:8888/interface.php](http://127.0.0.1:8888/interface.php)

}


server_all_len=${#server_all_list[*]}

i=0

while  [ $i -lt $server_all_len ]

do

server_ip=$(echo ${server_all_list[$i]} | awk -F ‘:’ ‘{print $1}’)

server_port=$(echo ${server_all_list[$i]} | awk -F ‘:’ ‘{print $2}’)

if curl -m 10 -G [http://${server_all_list](http://$%7bserver_all_list/)[$i]}/ > /dev/null 2>&1

then

#status:    0,http down    1,http ok    2,http down but ping ok

status=1

echo “服务器${server_ip}，端口${server_port}能够正常访问！”

else

if curl -m 30 -G [http://${server_all_list](http://$%7bserver_all_list/)[$i]}/ > /dev/null 2>&1

then

status=1

echo “服务器${server_ip}，端口${server_port}能够正常访问！”

else

if ping -c 1 $server_ip > /dev/null 2>&1

then

status=2

echo “服务器${server_ip}，端口${server_port}无法访问，但是能够Ping通！”

else

status=0

echo “服务器${server_ip}，端口${server_port}无法访问，并且无法Ping通！”

fi

fi

fi

send_msg_to_interface

let i++

done

* * *

２、TCP服务器监控

脚本：/data0/monitor/tcp.sh


引用


#!/bin/sh

LANG=C

# 被监控服务器、端口列表

server_all_list=(\

192.168.1.4:11211 \

192.168.1.5:11211 \

192.168.1.6:25 \

192.168.1.7:25 \

)


date=$(date -d “today” +”%Y-%m-%d_%H:%M:%S”)


#采用HTTP POST方式发送检测信息给接口程序interface.php，接口程序负责分析信息，决定是否发送报警MSN消息、手机短信、电子邮件。

send_msg_to_interface()

{

/usr/bin/curl -m 600 -d menu=tcp -d date=$date -d ip=$server_ip -d port=$server_port -d status=$status [http://127.0.0.1:8888/interface.php](http://127.0.0.1:8888/interface.php)

}


server_all_len=${#server_all_list[*]}

i=0

while  [ $i -lt $server_all_len ]

do

server_ip=$(echo ${server_all_list[$i]} | awk -F ‘:’ ‘{print $1}’)

server_port=$(echo ${server_all_list[$i]} | awk -F ‘:’ ‘{print $2}’)

if nc -vv -z -w 3 $server_ip $server_port > /dev/null 2>&1

then

#status:    0,http down    1,http ok    2,http down but ping ok

status=1

echo “服务器${server_ip}，端口${server_port}能够正常访问！”

else

if nc -vv -z -w 10 $server_ip $server_port > /dev/null 2>&1

then

status=1

echo “服务器${server_ip}，端口${server_port}能够正常访问！”

else

if ping -c 1 $server_ip > /dev/null 2>&1

then

status=2

echo “服务器${server_ip}，端口${server_port}无法访问，但是能够Ping通！”

else

status=0

echo “服务器${server_ip}，端口${server_port}无法访问，并且无法Ping通！”

fi

fi

fi

send_msg_to_interface

let i++

done


* * *

３、MySQL服务器监控

①、MySQL是否能够连接

②、MySQL是否发生表损坏等错误

③、MySQL活动连接 数是否过多

④、MySQL从库是否同步正常

⑤、MySQL从库同步延迟时间是否过大

脚本：/data0 /monitor/mysql.php


[view plain](http://blog.s135.com/#) [print](http://blog.s135.com/#) [?](http://blog.s135.com/#)

002. //$server_list[]=”服务器地址:端口:帐号:密码”;
003. $server_list[]=“192.168.1.11:3306:root:password”;
004. $server_list[]=“192.168.1.12:3306:root:password”;
005. $server_list[]=“192.168.1.13:3306:root:password”;
007. $database=“mysql”;
009. $curl = new Curl_Class();
011. foreach ($server_listas$server) {
012. $status=1;//初始化， 正常状态
013. unset($data);
014. $data[“menu”] = “mysql”;
015. $data[“info”] = “”;
016. list($data[“ip”], $data[“port”], $username, $password) = explode(“:”, $server);
018. $connect = @mysql_connect($data[“ip”].“:”.$data[“port”], $username, $password);
019. if(! $connect)
020. {
021. $status=0;
022. $data[“info”] = $data[“info”] . “无法连接 MySQL服务器\r\n”;
023. }
025. $select = @mysql_select_db($database, $connect);
026. $result = @mysql_query(“show slave status”);
027. $rs_slave = @mysql_fetch_array($result);
028. $result = @mysql_query(“show global status like ‘Threads_running'”);
029. $rs_threads = @mysql_fetch_array($result);
030. if($rs_slave[“Slave_SQL_Running”] == “No”)
031. {
032. $status=0;//故障状态
033. $data[“abstract”] = “从库不同步”;
034. $data[“info”] = $data[“info”] . “Slave_SQL_Running = No\r\n”;
035. }
036. if($rs_slave[“Slave_IO_Running”] == “No”)
037. {
038. $status=0;
039. $data[“abstract”] = ” 从库不同步”;
040. $data[“info”] = $data[“info”] . “Slave_IO_Running = No\r\n”;
041. }
042. if($rs_slave[“Last_Error”] != “”)
043. {
044. $status=0;
045. $data[“abstract”] = ” 从库同步出错”;
046. $data[“info”] = $data[“info”] . “Last_Error = “.substr($rs_slave[“Last_Error”], 0, 40).“\r\n”;
047. }
048. if($rs_slave[“Seconds_Behind_Master”] > 180)
049. {
050. $status=0;
051. $data[“abstract”] = “从库同步延迟时间高达”.$rs_slave[“Seconds_Behind_Master”].“秒”;
052. $data[“info”] = $data[“info”] . “Seconds_Behind_Master = “.$rs_slave[“Seconds_Behind_Master”].“\r\n”;
053. }
054. if($rs_threads[“Value”] > 60)
055. {
056. $status=0;
057. $data[“abstract”] = “活动连接数多达”.$rs_threads[“Value”];
058. $data[“info”] = $data[“info”] . “Threads_running = “.$rs_threads[“Value”].“\r\n”;
059. }
061. $data[“date”] = date(“Y-m-d_H:i:s”);
062. if($status == 0)
063. {
064. $post = @$curl->post(“http://127.0.0.1:8888/interface.php”, $data);
065. echo“MySQL服务器“”.$data[“ip”].“:”.$data[“port”].“”发生故 障!\n”;
066. print_r($post);
067. }
068. else
069. {
070. $data[“failback”] = “active”;//服务器正常，发送通知信息
071. $post = @$curl->post(“http://127.0.0.1:8888/interface.php”, $data);
072. echo“MySQL服务器“”.$data[“ip”].“:”.$data[“port”].“”运行正 常!\n”;
073. print_r($post);
074. }
075. }
077. /**
078. *********************************************************************
079. * Curl_Class :curl 类
080. *********************************************************************/
081. class Curl_Class
082. {
083. function Curl_Class()
084. {
085. return true;
086. }
088. function execute($method, $url, $fields = ”, $userAgent = ”, $httpHeaders = ”,
089. $username = ”, $password = ”)
090. {
091. $ch = Curl_Class::create();
092. if (false === $ch)
093. {
094. return false;
095. }
097. if (is_string($url) && strlen($url))
098. {
099. $ret = curl_setopt($ch, CURLOPT_URL, $url);
100. }
101. else
102. {
103. return false;
104. }
105. //是否显示头部信息
106. curl_setopt($ch, CURLOPT_HEADER, false);
107. //
108. curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
110. if ($username != ”)
111. {
112. curl_setopt($ch, CURLOPT_USERPWD, $username . ‘:’ . $password);
113. }
115. $method = strtolower($method);
116. if (‘post’ == $method)
117. {
118. curl_setopt($ch, CURLOPT_POST, true);
119. if (is_array($fields))
120. {
121. $sets = array();
122. foreach ($fieldsas$key => $val)
123. {
124. $sets[] = $key . ‘=’ . urlencode($val);
125. }
126. $fields = implode(‘&’, $sets);
127. }
128. curl_setopt($ch, CURLOPT_POSTFIELDS, $fields);
129. }
130. else
131. if (‘put’ == $method)
132. {
133. curl_setopt($ch, CURLOPT_PUT, true);
134. }
136. //curl_setopt($ch, CURLOPT_PROGRESS, true);
137. //curl_setopt($ch, CURLOPT_VERBOSE, true);
138. //curl_setopt($ch, CURLOPT_MUTE, false);
139. curl_setopt($ch, CURLOPT_TIMEOUT, 600);
141. if (strlen($userAgent))
142. {
143. curl_setopt($ch, CURLOPT_USERAGENT, $userAgent);
144. }
146. if (is_array($httpHeaders))
147. {
148. curl_setopt($ch, CURLOPT_HTTPHEADER, $httpHeaders);
149. }
151. $ret = curl_exec($ch);
153. if (curl_errno($ch))
154. {
155. curl_close($ch);
156. returnarray(curl_error($ch), curl_errno($ch));
157. }
158. else
159. {
160. curl_close($ch);
161. if (!is_string($ret) || !strlen($ret))
162. {
163. return false;
164. }
165. return$ret;
166. }
167. }
169. function post($url, $fields, $userAgent = ”, $httpHeaders = ”, $username = ”,
170. $password = ”)
171. {
172. $ret = Curl_Class::execute(‘POST’, $url, $fields, $userAgent, $httpHeaders, $username,
173. $password);
174. if (false === $ret)
175. {
176. return false;
177. }
179. if (is_array($ret))
180. {
181. return false;
182. }
183. return$ret;
184. }
186. function get($url, $userAgent = ”, $httpHeaders = ”, $username = ”, $password =
187. ”)
188. {
189. $ret = Curl_Class::execute(‘GET’, $url, ”, $userAgent, $httpHeaders, $username,
190. $password);
191. if (false === $ret)
192. {
193. return false;
194. }
196. if (is_array($ret))
197. {
198. return false;
199. }
200. return$ret;
201. }
203. function create()
204. {
205. $ch = null;
206. if (!function_exists(‘curl_init’))
207. {
208. return false;
209. }
210. $ch = curl_init();
211. if (!is_resource($ch))
212. {
213. return false;
214. }
215. return$ch;
216. }
218. }
219. ?>

* * *

4、主动监控守护进程

脚本：/data0/monitor/monitor.sh


引用


#!/bin/sh

while true

do

/bin/sh /data0/monitor/http.sh > /dev/null 2>&1

/bin/sh /data0/monitor/tcp.sh > /dev/null 2>&1

/usr/local/php/bin/php /data0/monitor/mysql.php > /dev/null 2>&1

sleep 10

done


启动主动监控守护进程：


/usr/bin/nohup /bin/sh /data0/monitor/monitor.sh 2>&1 > /dev/null &


* * *

三、被动报告监控（“被监控机”采集数据发送给“监控机”）

１、磁盘空间使用量监控

２、磁盘Inode使用量监控

３、Swap交换空间使用量监控

４、系统负载监控

５、Apache进程数监控


被动监控这部分，在我的文章《 [写完“Linux服务 器监控系统 ServMon V1.1”](http://blog.s135.com/read.php/291.htm) 》中已经实现，就不再详细写出。