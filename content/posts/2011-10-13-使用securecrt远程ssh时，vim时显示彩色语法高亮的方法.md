---
title: SecureCRT远程ssh使VIM语法加亮
author: admin
type: post
date: 2011-10-13T05:14:15+00:00
url: /archives/11676
IM_contentdowned:
 - 1
categories:
 - 其它
tags:
 - SecureCRT
 - vim

---
使用SecureCRT登录linux服务器用VIM时显示彩色语法高亮的方法

1：在$HOME 目录下 vim ~/.vimrc 建立一个文件
2：在最后面添两句：syntax on 和 set nocp ，然后保存
3：在SecureCRT中设置 选项->会话选项->终端->仿真->终端：Linux
4：重新登录linux服务器，打开 vim，现在就可自动对语法进行加亮了。

[![](http://blog.haohtml.com/wp-content/uploads/2011/10/vim.jpg)][1]

 [1]: http://blog.haohtml.com/wp-content/uploads/2011/10/vim.jpg