---
title: FreeBSD好工具：Screen
author: admin
type: post
date: 2009-04-25T11:20:54+00:00
excerpt: |
 |
 非常非常爽的一个工具，看了书之后研究了一下，非常爽

 # screen
 //以下^A表示同按“Ctrl + A”键
 # ^A c //Create，开出新的 window
 # ^A n //Next，切换到下个 window
 # ^A p //Previous，前一个 window
url: /archives/1294
IM_contentdowned:
 - 1
categories:
 - 服务器

---

非常非常爽的一个工具，看了书之后研究了一下，非常爽

**# screen**

//以下^A表示同按“Ctrl + A”键

# ^A c //Create，开出新的 window

# ^A n //Next，切换到下个 window

# ^A p //Previous，前一个 window

# ^A ^A //在两个 window 间切换

# ^A w //Windows，列出已开启的 windows 有那些

# ^A 0…9 //切换到第 0..9 个 window

# ^A t //Time，显示目前的时间，与系统的 load

# ^A K //kill window，强制关掉目前的 window

# ^A ? //Help，显示简单说明

# ^A d //detach，将目前的 screen session (可能含有多个 windows) 丢到背景执行

_当_ _按了 ^A d_ _把 screen session detach_ _掉后，会回到还没进 screen_ _时的状态，此时在 screen session ?_ _每个 window_ _内跑的 process (_ _无论是前景/_ _背景)_ _都在继续执行，即使 logout_ _也不影响。_

**# screen -ls** //显示所有的 screen sessions

**# screen -r [keyword]** //挑个 screen session 回来 (捡回来)