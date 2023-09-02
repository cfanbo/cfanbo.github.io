---
title: php中ob_start函数 积累
author: admin
type: post
date: 2011-04-01T04:08:52+00:00
url: /archives/8855
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php

---
确实自己写不出来，只能看看别人的经验总结。

**PHP的ob_start();用法**

用PHP的ob_start();控制您的浏览器cache

Output Control 函数可以让你自由控制脚本中数据的输出。它非常地有用，特别是对于：当你想在数据已经输出后，再输出文件头的情况。输出控制函数不对使用 header() 或 setcookie(), 发送的文件头信息产生影响,只对那些类似于 echo() 和 PHP 代码的数据块有作用。
我们先举一个简单的例子，让大家对Output Control有一个大致的印象：

> Example 1.
>
> 程序代码:
>
> ob_start(); //打开缓冲区
> echo “Hellon”; //输出
> header(“location:”); //把浏览器重定向到
> ob\_end\_flush();//输出全部内容到浏览器
> ?>

所有对header()函数有了解的人都知道，这个函数会发送一段文件头给浏览器，但是如果在使用这个函数之前已经有了任何输出（包括空输出，比如空格，回车和换行）就会提示出错。如果我们去掉第一行的ob\_start()，再执行此程序，我们会发现得到了一条错误提示：”Header had all ready send by”！但是加上ob\_start，就不会提示出错，原因是当打开了缓冲区，echo后面的字符不会输出到浏览器，而是保留在服务器，直到你使用flush或者ob\_end\_flush才会输出，所以并不会有任何文件头输出的错误！

**一、 相关函数简介：**
1、Flush：刷新缓冲区的内容，输出。
函数格式：flush()
说明：这个函数经常使用，效率很高。
2、ob_start ：打开输出缓冲区
函数格式：void ob_start(void)
说明：当缓冲区激活时，所有来自PHP程序的非文件头信息均不会发送，而是保存在内部缓冲区。为了输出缓冲区的内容，可以使用ob\_end\_flush()或flush()输出缓冲区的内容。
3 、ob_get_contents ：返回内部缓冲区的内容。
使用方法：string ob\_get\_contents(void)
说明：这个函数会返回当前缓冲区中的内容，如果输出缓冲区没有激活，则返回 FALSE 。
4、ob_get_length：返回内部缓冲区的长度。
使用方法：int ob\_get\_length(void)
说明：这个函数会返回当前缓冲区中的长度；和ob\_get\_contents一样，如果输出缓冲区没有激活。则返回 FALSE。
5、ob_end_flush ：发送内部缓冲区的内容到浏览器，并且关闭输出缓冲区。
使用方法：void ob\_end\_flush(void)
说明：这个函数发送输出缓冲区的内容（如果有的话）。
6、ob_end_clean：删除内部缓冲区的内容，并且关闭内部缓冲区
使用方法：void ob\_end\_clean(void)
说明：这个函数不会输出内部缓冲区的内容而是把它删除！
7、ob_implicit_flush：打开或关闭绝对刷新
使用方法：void ob\_implicit\_flush ([int flag])
说明：使用过Perl的人都知道$|=x的意义，这个字符串可以打开/关闭缓冲区，而ob\_implicit\_flush函数也和那个一样，默认为关闭缓冲区，打开绝对输出后，每个脚本输出都直接发送到浏览器，不再需要调用 flush()
**二、深入了解：**

1. 关于Flush函数：
这个函数在PHP3中就出现了，是一个效率很高的函数，他有一个非常有用的功能就是刷新browser的cache.我们举一个运行效果非常明显的例子来说明flush.

> Example 2.
> 程序代码
>
>
> for($i = 1; $i <= 300; $i++ ) print(” “);
> // 这一句话非常关键，cache的结构使得它的内容只有达到一定的大小才能从浏览器里输出
> // 换言之，如果cache的内容不达到一定的大小，它是不会在程序执行完毕前输出的。经
> // 过测试，我发现这个大小的底限是256个字符长。这意味着cache以后接收的内容都会
> // 源源不断的被发送出去。
> For($j = 1; $j <= 20; $j++) {
> echo $j.”
> “;
> flush(); //这一部会使cache新增的内容被挤出去，显示到浏览器上
> sleep(1); //让程序”睡”一秒钟，会让你把效果看得更清楚
> }
> ?>

注：如果在程序的首部加入ob\_implicit\_flush()打开绝对刷新,就可以在程序中不再使用flush(),这样做的好处是：提高效率！

2. 关于ob系列函数：

我想先引用我的好朋友y10k的一个例子：

> Example 3.
>
> 比如你用得到服务器和客户端的设置信息，但是这个信息会因为客户端的不同而不同，如果想要保存phpinfo()函数的输出怎么办呢？在没有缓冲区控制之前，可以说一点办法也没有，但是有了缓冲区的控制，我们可以轻松的解决：
> 程序代码
>
> ob_start(); //打开缓冲区
> phpinfo(); //使用phpinfo函数
> $info=ob_get_contents(); //得到缓冲区的内容并且赋值给$info
> $file=fopen(”,’w’); //打开文件
> fwrite($file,$info); //写入信息到
> fclose($file); //关闭文件
> ?>

用以上的方法，就可以把不同用户的phpinfo信息保存下来，这在以前恐怕没有办法办到！其实上面就是将一些”过程”转化为”函数”的方法！
或许有人会问：”难道就这个样子吗？还有没有其他用途？”当然有了，比如笔者论坛的PHP 语法加亮显示就和这个有关（PHP默认的语法加亮显示函数会直接输出，不能保存结果，如果在每次调用都显示恐怕会很浪费CPU，笔者的论坛就把语法加亮函数显示的结果用控制缓冲区的方法保留了），大家如果感兴趣的话可以来看看

可能现在大家对ob\_start()的功能有了一定的了解，上面的一个例子看似简单，但实际上已经掌握了使用ob\_start()的要点。
<1>.使用ob\_start打开browser的cache，这样可以保证cache的内容在你调用flush(),ob\_end_flush()（或程序执行完毕）之前不会被输出。
<2>.现在的你应该知道你所拥有的优势：可以在任何输出内容后面使用header,setcookie以及session，这是ob\_start一个很大的特点；也可以使用ob\_start的参数，在cache被写入后，然后自动运行命令，比如ob\_start(“ob\_gzhandler”)；而我们最常用的做法是用ob\_get\_contents()得到cache中的内容，然后再进行处理……
<3>.当处理完毕后，我们可以使用各种方法输出，flush(),ob\_end\_flush(),以及等到程序执行完毕后的自动输出。当然，如果你用的是ob\_get\_contents()，那么就要你自己控制输出方式了。

来，让我们看看能用ob系列函数做些什么……

**一、 静态模版技术**

简介：所谓静态模版技术就是通过某种方式，使得用户在client端得到的是由PHP产生的html页面。如果这个html页面不会再被更新，那么当另外的用户再次浏览此页面时，程序将不会再调用PHP以及相关的数据库，对于某些信息量比较大的网站，例如sina,163,sohu。类似这种的技术带来的好处是非常巨大的。

我所知道的实现静态输出的有两种办法：
<1>.通过y10k修改的phplib的一个叫类实现。
<2>.使用ob系列函数实现。
对于第一种方法，因为不是这篇文章所要研究的问题，所以不再赘述。
我们现在来看一看第二种方法的具体实现：

> Example 4.
>
> 程序代码
>  ob_start();//打开缓冲区
> ?>
> php页面的全部输出
>  $content = ob\_get\_contents();//取得php页面输出的全部内容
> $fp = fopen(“”, “w”); //创建一个文件，并打开，准备写入
> fwrite($fp, $content); //把php页面的内容全部写入，然后……
> fclose($fp);
> ?>

这样，所谓的静态模版就很容易的被实现了……

**二、 捕捉输出**

以上的Example 4.是一种最简单的情况，你还可以在写入前对$content进行操作……
你可以设法捕捉一些关键字，然后去对它进行再处理，比如Example 3.所述的PHP语法高亮显示。个人认为，这个功能是此函数最大的精华所在，它可以解决各种各样的问题，但需要你有足够的想象力……

> Example 5.
>
> 程序代码
>  Function run_code($code) {
> If($code) {
> ob_start();
> eval($code);
> $contents = ob\_get\_contents();
> ob\_end\_clean();
> }else {
> echo “错误！没有输出”;
> exit();
> }
> return $contents;
> ?>
> }

以上这个例子的用途不是很大，不过很典型$code的本身就是一个含有变量的输出页面，而这个例子用eval把$code中的变量替换，然后对输出结果再进行输出捕捉，再一次的进行处理……

**Example 6. 加快传输**

> 程序代码
>  /*
> ** Title………: PHP4 HTTP Compression Speeds up the Web
> ** Version…….:
> ** Author……..: catoc
> ** Filename……:
> ** Last changed..: 18/10/2000
> ** Requirments…: PHP4 >= 4.0.1
> ** PHP was configured with –with-zlib[=DIR]
> ** Notes………: Dynamic Content Acceleration compresses
> ** the data transmission data on the fly
> ** code by sun jin hu (catoc)
> ** Most newer browsers since 1998/1999 have
> ** been equipped to support the HTTP 1.1
> ** standard known as “content-encoding.”
> ** Essentially the browser indicates to the
> ** server that it can accept “content encoding”
> ** and if the server is capable it will then
> ** compress the data and transmit it. The
> ** browser decompresses it and then renders
> ** the page.
> **
> ** Modified by John Lim (jlim@)
> ** based on ideas by Sandy McArthur, Jr
> ** Usage……..:
> ** No space before the beginning of the first ‘ ** ————Start of file———-
> ** | ** | include(”);
> ** |? >
> ** |
> ** |… the page …
> ** |
> ** | ** | gzdocout();
> ** |? >
> ** ————-End of file———–
> */
> ob_start();
> ob\_implicit\_flush(0);
> function CheckCanGzip(){
> global $HTTP\_ACCEPT\_ENCODING;
> if (headers\_sent() || connection\_timeout() || connection_aborted()){
> return 0;
> }
> if (strpos($HTTP\_ACCEPT\_ENCODING, ‘x-gzip’) !== false) return “x-gzip”;
> if (strpos($HTTP\_ACCEPT\_ENCODING,’gzip’) !== false) return “gzip”;
> return 0;
> }
> /\* $level = compression level 0-9, 0=none, 9=max \*/
> function GzDocOut($level=1,$debug=0){
> $ENCODING = CheckCanGzip();
> if ($ENCODING){
> print “nn”;
> $Contents = ob\_get\_contents();
> ob\_end\_clean();
> if ($debug){
> $s = “

Not compress length: “.strlen($Contents);
> $s .= ”
> Compressed length: “.strlen(gzcompress($Contents,$level));
> $Contents .= $s;
> }
> header(“Content-Encoding: $ENCODING”);
> print “x1fx8bx08x00x00x00x00x00”;
> $Size = strlen($Contents);
> $Crc = crc32($Contents);
> $Contents = gzcompress($Contents,$level);
> $Contents = substr($Contents, 0, strlen($Contents) – 4);
> print $Contents;
> print pack(‘V’,$Crc);
> print pack(‘V’,$Size);
> exit;
> }else{
> ob\_end\_flush();
> exit;
> }
> }
> ?>