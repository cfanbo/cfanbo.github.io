---
title: 分布式文件系统MFS(moosefs)实现存储共享
author: admin
type: post
date: 2010-06-26T15:50:23+00:00
url: /archives/4024
IM_data:
 - 'a:4:{s:52:"http://www.kuqin.com/upimg/allimg/090410/1755520.jpg";s:52:"http://www.kuqin.com/upimg/allimg/090410/1755520.jpg";s:52:"http://www.kuqin.com/upimg/allimg/090410/1755521.jpg";s:52:"http://www.kuqin.com/upimg/allimg/090410/1755521.jpg";s:52:"http://www.kuqin.com/upimg/allimg/090410/1755522.jpg";s:52:"http://www.kuqin.com/upimg/allimg/090410/1755522.jpg";s:52:"http://www.kuqin.com/upimg/allimg/090410/1755523.jpg";s:52:"http://www.kuqin.com/upimg/allimg/090410/1755523.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 前端设计

---
由于用户数量的不断攀升,我对访问量大的应用实现了可扩展、高可靠的集群部署（即lvs+keepalived的方式），但仍然有用户反馈访问慢的 问题。通过排查个服务器的情况，发现问题的根源在于共享存储服务器NFS。在我这个网络环境里，N个服务器通过nfs方式共享一个服务器的存储空间，使得 NFS服务器不堪重负。察看系统日志，全是nfs服务超时之类的报错。一般情况下，当nfs客户端数目较小的时候，NFS性能不会出现问题；一旦NFS服 务器数目过多，并且是那种读写都比较频繁的操作，所得到的结果就不是我们所期待的。

下面是某个集群使用nfs共享的示意图：![](http://www.kuqin.com/upimg/allimg/090410/1755520.jpg)

这种架构除了性能问题而外，还存在单点故障，一旦这个NFS服务器发生故障，所有靠共享提供数据的应用就不再可用，尽管用rsync方式同步数据到 另外一个服务器上做nfs服务的备份，但这对提高整个系统的性能毫无帮助。基于这样一种需求，我们需要对nfs服务器进行优化或采取别的解决方案，然而优 化并不能对应对日益增多的客户端的性能要求，因此唯一的选择只能是采取别的解决方案了；通过调研，分布式文件系统是一个比较合适的选择。采用分布式文件系 统后，服务器之间的数据访问不再是一对多的关系（1个NFS服务器，多个NFS客户端），而是多对多的关系，这样一来，性能大幅提升毫无问题。

到目前为止，有数十种以上的分布式文件系统解决方案可供选择，如lustre,hadoop,Pnfs等等。我尝试了 PVFS,hadoop,moosefs这三种应用，参看了lustre、KFS等诸多技术实施方法，最后我选择了moosefs（以下简称MFS）这种 分布式文件系统来作为我的共享存储服务器。为什么要选它呢？我来说说我的一些看法：

1、 实施起来简单。MFS的安装、部署、配置相对于其他几种工具来说，要简单和容易得多。看看lustre 700多页的pdf文档，让人头昏吧。

2、 不停服务扩容。MFS框架做好后，随时增加服务器扩充容量；扩充和减少容量皆不会影响现有的服务。注：hadoop也实现了这个功能。

3、 恢复服务容易。除了MFS本身具备高可用特性外，手动恢复服务也是非常快捷的，原因参照第1条。

4、 我在实验过程中得到作者的帮助，这让我很是感激。

![](http://www.kuqin.com/upimg/allimg/090410/1755521.jpg)

**MFS文件系统的组成**

1、 元数据服务器。在整个体系中负责管理管理文件系统，目前MFS只支持一个元数据服务器master，这是一个单点故障，需要一个性能稳定的服务器来充当。 希望今后MFS能支持多个master服务器，进一步提高系统的可靠性。

2、 数据存储服务器chunkserver。真正存储用户数据的服务器。存储文件时，首先把文件分成块，然后这些块在数据服务器chunkserver之间复 制（复制份数可以手工指定，建议设置副本数为3）。数据服务器可以是多个，并且数量越多，可使用的“磁盘空间”越大，可靠性也越高。

3、 客户端。使用MFS文件系统来存储和访问的主机称为MFS的客户端，成功挂接MFS文件系统以后，就可以像以前使用NFS一样共享这个虚拟性的存储了。

**元数据服务器安装和配置**

元数据服务器可以是linux,也可以是unix,你可以根据自己的使用习惯选择操作系统,在我的环境里,我是用freebsd做为MFS元数据的 运行平台。GNU源码，在各种类unix平台的安装都基本一致。

**（一） 安装元数据服务**

1、下载GNU源码 wget http://www.moosefs.com/files/mfs-1.5.12.tar.gz

2、解包 tar zxvf mfs-1.5.12.tar.gz

3、切换目录 cd mfs-1.5.12

4、创建用户 useradd mfs –s /sbin/nologin

5、配置 ./configure –prefix=/usr/local/mfs –with-default-user=mfs –with-default-group=mfs

6、编译安装 make ; make install

**（二） 配置元数据服务**

元数据服务器的配置文件是mfsmaster.cfg,我在安装MFS时指定了前缀，因此这个文件的位置在/usr/local/mfs/etc /mfsmaster.cfg.我们打开这个配置文件，看看都有哪些内容：

![](http://www.kuqin.com/upimg/allimg/090410/1755522.jpg)

尽管每行都被注释掉了，但它们却是配置文件的默认值，要改变这些值，需要取消注释，然后明确指定其取值。接下来说明一下其中一些项目的含义。

◆ LOCK_FILE = /var/run/mfs/mfsmaster.pid 文件锁所在的位置，它的功能是避免启动多次启动同一个守护进程。由于系统中本来不存在目录 /var/run/mfs，因此需要手动创建 mkdir /var/run/mfs，然后更改其属主 chown –R mfs:mfs /var/run/mfs 这样MFS 服务就能对这个目录有创建/写入 mfsmaster.pid 文件的权限了。

◆ DATA_PATH = /usr/local/mfs/var/mfs 数据存放路径，只元数据的存放路径。那么这些数据都包括哪些呢？进目录看看，大致分3种类型的文件：

![](http://www.kuqin.com/upimg/allimg/090410/1755523.jpg)

这些文件也同样要存储在其他数据存储服务器的相关目录。

◆ MATOCS\_LISTEN\_PORT = 9420 MATOCS–master to chunkserver，即元数据服务器使用9420这个监听端口来接受数据存储服务器chunkserver端的连接。

◆ MATOCU\_LISTEN\_PORT = 9421 元数据服务器在9421端口监听，用以接受客户端对MFS进行远程挂接（客户端以mfsmount挂接MFS）

◆ 其他部分看�