---
title: linux/unix命令备忘:ln -s 建立文档连结
author: admin
type: post
date: 2009-01-10T10:41:27+00:00
url: /archives/823
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ln

---
1 . **使用方式** ：

> ln [option] source_file **dist\_file\_link_name**   （source\_file是待建立链接文件的源文件，dist\_file是新创建的链接文件）
> -f 建立时，将同档案名删除.
> -i 删除前进行询问.

两个参数的位置经常记错，只需要记住命令和显示结果的位置正好相反。
写 ln 命令时第一个参数就是一个普通的文件名，第二个参数是链接名
而使用 ls -al 命令查看时，则是 链接名在前，实际文件名在后，中间用 -> 连接（硬连接也是一个文件嘛，肯定按普通文件名来显示了）

例如建立一个 abc.txt 文件的的软连接，并取名为 abc-link.txt
\# ln -s abc.txt  abc-link.txt
\# ls -al
lrwxrwxrwx 1 sxf sxf 7 4月 12 14:14 abc-link.txt -> abc.txt
-rw-rw-r– 1 sxf sxf 4 4月 12 14:14 abc.txt

如果这里删除了原来物理文件名 abc.txt， 则对应的软链接文件名虽然用ls可以看到，但其实无效的，可以手动删除。如果删除的是链接文件的话，则恢复原来样子。

例如建立文件 a.txt 的硬连接，并取名为 a-hardlink.txt
\# ln a.txt a-hardlink.txt
\# ls -al | grep txt
-rw-rw-r– 2 sxf sxf 0 4月 12 14:24 a-hardlink.txt
-rw-rw-r– 2 sxf sxf 0 4月 12 14:24 a.txt

可以看到显示的结果与软软件不一样，这里显示的是两个文件，并不能一眼看出来两者的关系。

不过你可以通过 stat 命令查看，它们的 Inode 是一样的，并且 Links 值为2。

这里如果你删除 a.txt 的话，则其对应的硬链接文件仍是可编辑的。

**2. 软链接与硬链接的区别（通俗）：**
硬链接可认为是一个文件拥有两个文件名;

而软链接则是系统新建一个链接文件，此文件指向其所要指的文件。


此外，软链接可对文件和文件夹。。而硬链接仅针对文件。


**3. 软链接与硬链接的区别（讲解）：**

Linux 软连接与硬连接

对于一个文件来说，有唯一的索引接点与之对应，而对于一个索引接点号，却可以有多个文件名与之对应。因此，在磁盘上的同一个文件可以通过不同的路径去访问该文件。注意在Linux下是一切皆文件的啊，文件夹、新加的硬盘 …都可以看着文件来处理的啊。

假设这里网站 [www.haohtml.com](http://www.haohtml.com) 的网站根目录为**/home/haohtml/www**,想将另一个目录/data/docs里的内容通过 [www.haohtml.com/tools](http://www.haohtml.com/tools) 访问.这里通过建立符号连接实现.

**ln -s   /data/docs    **/home/haotml/www/tools****

现在通过web  访问原来的文件了。

注意建立符号的连接的操作权限！

**附：ln 文档连结 详细说明**
命令格式：

**ln -s originname newname ( Hard link )**

同一文档，可拥有一个以上之名称，可将文档做数个连结.例子 ：

ln -s file1 file2 　　将名称 file2，连结至文档 file1.

**用途说明：**

1. 为一个文件建立别名，方便系统调用，而不需要修改任何配置。

2. 为一个目录建立别名，方便系统调用，而不需要修改任何配置。

**删除链接**

删除符号链接，有创建就有删除

```
$rm -rf symbolic_name
```

注意不是rm -rf symbolic_name/

# linux删除软/硬链接会不会删除原始文件？

linux的软链接和硬链接删除都不会影响原始文件，但是修改的话都会影响原始文件。
1、linux的软链接相当于windows里的快捷方式，快捷方式虽然删除了，但源文件还是存在的。
2、硬链接有可能会（即当硬链接数为1时，这时再删除，就真的删没了。如果硬链接数大于1，则删除硬链接只是使连接数减1，而不会真正删除文件）。它的特点就是，链接文件和原始文件只要有一个存在，文件就会存在，不会消失。（你删除源文件，依然可以在连接文件里打开）但是软链接可以跨系统，这点硬链接不行。