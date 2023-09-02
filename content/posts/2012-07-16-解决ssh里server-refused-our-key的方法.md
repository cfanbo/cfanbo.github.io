---
title: 解决SSH里“Server Refused Our Key”的方法
author: admin
type: post
date: 2012-07-16T06:22:22+00:00
url: /archives/13170
IM_data:
 - 'a:3:{s:69:"http://img.ph.126.net/Yjn4c35IWXPiALFVFbIYqA==/615867249059712387.jpg";s:78:"http://blog.haohtml.com/wp-content/uploads/2012/07/e079_615867249059712387.jpg";s:70:"http://img.ph.126.net/sYukklNPklb1sJuukZ-4jA==/2623628257936821436.jpg";s:79:"http://blog.haohtml.com/wp-content/uploads/2012/07/1106_2623628257936821436.jpg";s:70:"http://img.ph.126.net/RKOvh55slGao7NhKXpoS1A==/1040612988916092573.jpg";s:79:"http://blog.haohtml.com/wp-content/uploads/2012/07/ed2b_1040612988916092573.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - putty
 - ssh

---
/\***\***\***\***\***\***\***\***\***\***\***\***\***\***\*****
title:解决SSH里“Server Refused Our Key”的方法
author:insun
blog:http://yxmhero1989.blog.163.com/
\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\****/
=========================================================================

在公司使用虚拟机研究爬虫抓网页和相关数据，要连接linux虚拟机。

putty.exe 该软件可连接服务器，用来连接远程的linux服务器和虚拟机，或者用来设置代理。

[![](http://blog.haohtml.com/wp-content/uploads/2012/07/615867249059712387.jpg)](http://blog.haohtml.com/wp-content/uploads/2012/07/615867249059712387.jpg)

网关设置正确的话，应该可以不用密钥可以login的。若在其他地方才要ppk密钥key。

winscp406setup.exe  该软件用来在pc和服务器中传送文件

[![](http://blog.haohtml.com/wp-content/uploads/2012/07/putty_2.jpg)](http://blog.haohtml.com/wp-content/uploads/2012/07/putty_2.jpg)

[![](http://blog.haohtml.com/wp-content/uploads/2012/07/winscp.jpg)](http://blog.haohtml.com/wp-content/uploads/2012/07/winscp.jpg)



输入root后出现“Disconnected：No supported authentication methods available”
命令行里输入 ipconfig /flushdns这个试一下，自己研究去哈哈

====================================================================================

找了个：Server Refused Our Key

[http://jerrykwok.wordpress.com/2010/04/07/server-refused-our-key/](http://jerrykwok.wordpress.com/2010/04/07/server-refused-our-key/)

最近想用 Public Key Authentication 方法連接公司的 server，可是一直都碰上 “Server Refused Our Key” 的問題。經過一輪 Google 後，大致明白原來問題出於用 puttygen 產生的 Public Key 裡頭有 OpenSSH 認不出的一堆無謂文字；所以別要用 puttygen 裡 “Save public key” button 來產生 Public Key，而改為手動儲存。


**以下是手動產生 Public Key 的方法：**

1) 用 puttygen 產生 keys


2) 儲存 Private Key 在本機任何一個安全的地方（說完也覺得有點多餘）


3) 在 puttygen 頂部， “Public key for pasting into OpenSSH authorized_keys file” 下面的方格裡，那堆以 “ssh-rsa …” 為首的 codes 直接 Copy and paste 到一個純文字檔案裡


4) 在純文字檔案裡，把每行的 new line character 刪除掉，也就是說，讓整段 code 都在同一行裡


5) 把純文字檔儲存，檔案名稱可隨意（當然你不會想用中文檔名）。這裡就假設為 id_dsa


6) 用你喜歡的方法，把 id_dsa 抄到 server 上，home directory 下的 .ssh


7) cat id_dsa >> authorized_keys。如果你本來就沒有 authorized_keys，就改為 cat > authorized_keys


8) 要留意整個 .ssh 的 persmission，為了方便，我會把 .ssh 及以下的 sub-directories 改為700


至此，你的 Public Key 已經完成了，現在你可以再試試用 Putty 連線你的 server 試試看了。


摘自： [http://yxmhero1989.blog.163.com/blog/static/112157956201161214048431/](http://yxmhero1989.blog.163.com/blog/static/112157956201161214048431/)