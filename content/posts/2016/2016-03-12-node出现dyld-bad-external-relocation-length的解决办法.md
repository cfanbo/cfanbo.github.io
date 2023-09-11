---
title: 'node出现“dyld: bad external relocation length”的解决办法'
author: admin
type: post
date: 2016-03-12T14:15:03+00:00
url: /archives/16793
categories:
 - 程序开发

---
mac下有时候在执行 npm的过程中，我们强制中止了操作或者一些命令出现问题会提示“dyld: bad external relocation length”这个错误，这个时候只要将未下载完的文件删除即可，我这里使用 n 4.4.3下载的时候，网络出现异常，提示这个错误，只要删除 /usr/local/n/versions/node/4.4.3 这个目录即可。

如果还是不行的话，可以试着执行一下

```
brew doctor
```

命令，根据提示操作就可以了。

我遇到的情况是使用node版本控制n安装新版本号的时候，好像安装包下载的不完整，但提示安装成功了，在最后提示这个错误。试了好多方法卸载重装也不行，最后根据 brew doctor 的提示，执行了

```
brew link --overwrite node
```

彻底解决了。