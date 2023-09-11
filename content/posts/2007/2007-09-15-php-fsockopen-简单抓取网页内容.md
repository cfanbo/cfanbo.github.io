---
title: PHP fsockopen 简单抓取网页内容
author: admin
type: post
date: 2007-09-15T14:25:20+00:00
url: /archives/131
IM_contentdowned:
 - 1
categories:
 - 程序开发

---
      这几天在做采集的东东,[php][1]提供了很多访问远程计算机内容的方法,文件系统的函数些都支持读取远程文件,而fsockopen是争对于socket接口的编程函数,在网上搜了一下发现用这个函数来读取http内容也比较多,但是没有一个比较完善和适合我的,在某个小偷程序上改改,轻而易举的完善fsockopen请求http协议内容,从而获取请求内容.代码如下:

function get_page_content($url){

$url = eregi_replace(‘^http://’, ”, $url);

$temp = explode(‘/’, $url);

$host = array_shift($temp);

$path = ‘/’.implode(‘/’, $temp);

$temp = explode(‘:’, $host);

$host = $temp[0];

$port = isset($temp[1]) ? $temp[1] : 80;

$fp = @fsockopen($host, $port, &$errno, &$errstr, 30);

if ($fp){

@fputs($fp, "GET $path HTTP/1.1\r\nHost: $host\r\nAccept: */*\r\nReferer:$url\r\nUser-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)\r\nConnection: Close\r\n\r\n");

}

$Content = ”;

while ($str = @fread($fp, 4096)){

$Content .= $str;

}

@fclose($fp);

//重定向

if(preg_match("/^HTTP\/\d.\d 301 Moved Permanently/is",$Content)){

if(preg_match("/Location:(.*?)\r\n/is",$Content,$murl)){

     return get_page_content($murl[1]);

}

}

//读取内容

if(preg_match("/^HTTP\/\d.\d 200 OK/is",$Content)){

preg_match("/Content-Type:(.*?)\r\n/is",$Content,$murl);

$contentType=trim($murl[1]);

    $Content=explode("\r\n\r\n",$Content,2);

$Content=$Content[1];

}

return $Content;

}

 [1]: /?tag=php