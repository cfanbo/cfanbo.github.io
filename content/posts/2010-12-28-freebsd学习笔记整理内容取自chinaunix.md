---
title: FreeBSD学习笔记整理(内容取自chinaunix)
author: admin
type: post
date: 2010-12-28T03:50:33+00:00
url: /archives/7327
IM_contentdowned:
 - 1
categories:
 - 服务器

---
**1、查看 CPU：**
sysctlhw.modelhw.ncpu
dmesg|grep”CPU:”

**2、查看内存：**
dmesg|grep “real memory”|awk -F ‘[()]’ ‘{print$2,$4,$7,$8}’
 查看 swap：
top|grep”Swap:”|awk'{print$1,$2}’
 **3、查看硬盘：**
diskinfo‐vt/dev/ad0
disklable/dev/ad0s2#查看分区信息
 看硬盘大小：
dmesg|grep”sector”|awk'{print$1,$2}’
diskinfo‐v/dev/da0|grep”inbytes”|awk‐F'[()]”{print$2}’


 **4、查看服务器品牌：**
dmesg|grep”ACPIAPIC”
 **5、挂载文件系统：**
fat32：mount\_msdosfs‐Lzh\_CN.eucCN/dev/ad0s1/mnt
ntfs：mount_ntfs‐CeucCn/dev/ad0s1/mnt
cdrom：mount_cd9660/dev/acd0/mnt
 注：ntfs 在 FreeBSD 中只能读无法写入
 **6、给文件添加或禁用系统禁删标志（目录不适用）：**
chflagssunlinkfile1
chflagsnosunlinkfile1
 **7、初始化磁盘:
** fdisk‐BIad1

 **8、建立 FreeBSD 分区：**
disklabel‐B‐w‐rad1s1auto
 **9、建立逻辑分区：**
disklabel‐ead1s1
 **10、格式化分区，创建文件系统：**
newfs/dev/ad1s1e
 **11、显示 PCI 硬件信息：**
pciconf‐lv
 **12、开启 Linux二进制兼容支持**（启用这一功能最简单的方法是载入linuxKLD模块）：
kldloadlinux
让 Linux 兼容在系统初始化时自动启用，在/etc/rc.conf 中中入：
 linux_enable=”YES”
 **13、检查 KLD 模块是否加载：**
kldstat
 **14、在内核中静态链接进 Linux 二进制兼容模式，在内核配置文件里面加入：**
optionsCOMPAT_LINUX
 **15、设置网卡 em0 的 IP 地址：**
ifconfigem0inet192.0.2.10netmask255.255.255.0
 **16、给网卡 em0 设置添加一个别名 IP 地址：**
ifconfigem0inet192.168.51.45/24add
 **17、删除网卡的别名 IP 地址：**
ifconfigem0inet192.168.51.45‐alias
 **18、设置网卡 em0 的工作模式为 100baseTX 全双式：**
ifconfigem0media100baseTXmediaoptfull‐duplex 19、当/usr/local/etc/rc.d 下的脚本无法自动启动时，可尝试在/etc/rc.conf 中加入一行：
local_startup=”/usr/local/etc/rc.d”
 **20、在 ports 中寻找需要的软件，进入/usr/ports 目录执行：**
makesearchname=lsof或echo/usr/ports/\*/\*lsof*或whereislsof
makesearchkey=关键字#在名字、注释、描述中搜索关键字
 **21、使用 package 方式安装管理软件**，使用以下命令：
pkg_addlsof‐4.56.4.tgz#安装软件包
pkg_info#列出已安装所有软件包
pkg_version#统计所有安装的软件版本，比较本地 package 的版本与 ports 目录中的当前 版本是否一致
pkg_deletelsof‐4.56.4#删除软件包，需提供完整包名
 **22、**使用 CVSup 协议更新本地 ports：（将 cvsup.FreeBSD.org 改为离得较近的 CVSup 服务 器）
csup‐L2‐hcvsup.FreeBSD.org/usr/share/examples/cvsup/ports‐supfile
 **23、**一些 shell 会缓存环境变量 PATH 中指定的目录里的可执行文件，以加快查找速度，这 会造成一些新安装的命令无法运行，执行以下命令，然后才能运行新安装的那些命令：
rehash或hash‐r
 **24、**当不是所有时间都能上网时，可在/usr/ports 下执行以下命令，所有需要的文件都将 被下载：（此命令可以在下级目录中执行，如/usr/ports/comms/nmp）
makefetch#只下载所需要文件，不下载依赖包
makefetch‐recursive#连同依赖包一起下载
 **25、改变默认的 Ports 目录：**
makeWRKDIRPREFIX=/usr/home/example/portsinstall#在/usr/home/example/ports 中编译 port，安装到/usr/local
makePREFIX=/usr/home/example/localinstall#在/usr/ports 中编译 port，安装到 /usr/home/example/local
makeWRKDIRPREFIX=../portsPREFIX=../localinstall#在../ports 中编译 port，安装到../local 26、使用 portsclean 工具清除临时目录和 distfiles 目录：
portsclean‐C#清除安装时的临时目录
portsclean‐D#清除 distfiles 目录下所有 port 都不引用的文件
portsclean‐DD#删除目前安装的 port 没有使用的源码包文件
 **27、强制手动检测 SCSI 设备，SCSI 总线扫描：**
camcontrolrescanall
 **28、显示 SCSI 设备列表：**
camcontroldevlist
 **29、利用管道修改用户密码：**
echo”password”|pwusermodroot‐h0
 **30、sed 插入行：**
sed‐i‐E’/serviceport/a\\
apexport:18306\\
‘/home/xiyou/config
 **31、用 freebsd 的 MBR 覆盖现有的 MBR：**
fdisk‐B‐b/boot/boot0device
 **32、根据一个新的文件重新构建用户列表：**
pwd_mkdb‐p/etc/master.passwd.new#‐p 即为生成新的/etc/passwd
 **33、取时间：**
date‐v‐1d+%Y%m%d#Freebsd 取昨天日期方法
date‐v‐1w+%Y%m%d#Freebsd 取上周今日方法
date‐v‐1m+%Y%m%d#Freebsd 取上个月今日方法
date‐v‐1y+%Y%m%d#Freebsd 取去年今日的方法
34、以 xiyou 用户身份执行命令或脚本：
su‐xiyou‐c”cd/home/xiyou/script;./start_apex.sh&” 35、tar 打包时排除某个子目录：
tarzcvfApex09010702.tgz‐‐exclude=ApexItemServer/hook_logApexItemServer
注：上例是使用 GUN 版本的 tar 程序格式，否则‐‐exclude 参数应放在最后

**36、锁住终端：**
lock‐np#‐n永不超时,‐p使用系统密码作为开启终端的密匙
 **37、显示 ATA 设备列表：**
atacontrollist
 **38、查看网络流量：**
systat‐if1#1 表示 1 秒刷新屏幕一次，Traffic流量peak峰值average平均值
netstat1
 **39、查看硬盘详细分区实时读写状况：**
gstat
 **40、进单用户模式也需要密码：**
a.vi/etc/ttys找到 whengoingtosingle‐usermode
b.修改 consolenoneunknownoff 后面的 secure，改为 insecure
c.存盘退出
 **41、在 FreeBSD5.X 以上加载,卸载 ISO 文件：**
mount:
mdconfig‐a‐tvnode‐fmyisofile.iso#屏幕输出 md0 或者类似的设备名
mount‐tcd9660/dev/md0/mnt
umount:
umount/mnt
mdconfig‐d‐u0#‐u 后面的数字和前面的 md?中的数字一致
mdconfig‐l#可以列出关于配置 md?设备的信息 42、更新配置文件，比如编辑了.cshrc 等文件，就需要用 source 命令：
source.cshrc
 **43、修复 UFS 文件系统分区：**
fsck_ufs/dev/ad1
 **44、pf 防火墙**
pfctl‐e#启动 pf 防火墙
pfctl‐d#停止 pf 防火墙
pfctl‐sa|grepStatus#查看状态
pfctl‐f/etc/pf.conf#载入pf.conf文件
pfctl‐nf/etc/pf.conf#检查配置文件错误，但不载入
pfctl‐Nf/etc/pf.conf#只载入文件中的 NAT 规则
pfctl‐Rf/etc/pf.conf#只载入文件中的过滤规则
pfctl‐sn#显示当前的 NAT 规则
pfctl‐sr#显示当前的过滤规则
pfctl‐ss#显示当前的状态表
pfctl‐si#显示过滤状态和计数
pfctl‐sa#显示任何可显示的
pfctl‐thttp_table‐Tshow#查看动态表
pfctl‐thttp_table‐Tadd192.168.1.X#添加一个 IP 到表
pfctl‐thttp_table‐Tdel192.168.1.X#从表中删除 IP
 **45、系统优化+防止 ddos**
加载文件修改
#vi/boot/loader.conf#加入如下文本
kern.dfldsiz=”2147483648″#Settheinitialdatasizelimit
kern.maxdsiz=”2147483648″#Setthemaxdatasize kern.ipc.nmbclusters=”0″#Setthenumberofmbufclusters
kern.ipc.nsfbufs=”66560″#Setthenumberofsendfile(2)bufs
##解释：
a．第一，第二行主要是为了突破 1G 内存设置的
b．第三行其实是 bsd 的一个 bug，当系统并发达到一个数量级的时候，系统会 crash， 这个是非常糟糕的事情，所幸更改了这个参数后，在高并发的时候，基本可以没有类似情 况，当然非常 bt 的情况，还得进一步想办法
c．第四行是读取的文件数，如果你下载的文件比较大，且比较多，加大这个参数，是非 常爽的
Sysctl 修改

> #vi/etc/rc.local
> sysctlkern.ipc.maxsockets=100000##增加并发的 socket，对于 ddos 很有用
> sysctlkern.ipc.somaxconn=65535##打开文件数
> sysctlnet.inet.tcp.msl=2500##timeout 时间

加速 ports 安装

> #vi /etc/make.conf##加入如下
> MASTER\_SITE\_OVERRIDE?=http://ports.hshh.org/${DIST_SUBDIR}/
> MASTER\_SITE\_OVERRIDE?=http://ports.cn.freebsd.org/${DIST_SUBDIR}/

Freebsd 颜色显示
secureCRT 设置:仿真:终端‐>linux>勾选 ANSI 颜色‐‐>确定

> #vi/etc/csh.cshrc##加入如下
> setenvLSCOLORSExGxFxdxCxegedabagExEx
> setenvCLICOLORyes
> #cd/usr/ports/edit/vim;makeinstall
> #echo”syntaxon”>/root/.vimrc
> #echo”aliasvivim”>>/root/.cshrc ##颜色主要是靠 vim 来显示的，因此需要安装 vim，然后把 vialias 成 vim 就可以了

**46、查看系统状态**
fstat#报告系统中打开文件的信息
pstat‐T#显示这几个系统表的状态，包括当前使用的和可以利用的系统表空间，因此可以 用来检查系统在当前负载下是使用多大的系统表，帮助进行优化系统性能
systat#缺省情况下 systat 是报告处理器的使用率，包括总利用状态、空闲使用率和各个 进程的使用率
通过指定参数，systat 也能进行 I/O 的统计、虚存的统计、网络的统计等，这些参数 包括‐iostat,‐vmstat,‐mbufs,‐netstat,‐ip,‐icmp,‐tcp,‐swap 等

> kldstat‐v#显示内核加载的模块
> klsdstat‐mipfilter#显示指定模块
> pnpinfo#即插即用设备
> devinfo‐u#显示设备占用的 IRQ 和内存地址

使用 portsnap 生级 port 的目录树，建议使用；我没做这步，使用 port 安装 vim 系统报错。
 ①配置 portsnap：
我们使用 portsnap，首先要设置一下它的配置文件，位于/etc/portsnap.conf:
[root@bsd01/usr/ports]#vi/etc/portsnap.conf
把 SERVERNAME=portsnap.freebsd.org
修改成：
SERVERNAME=portsnap.hshh.org
在你的 freebsd 首次使用 portsnap 必须执行下面 2 步：

> [root@bsd01~]#portsnap fetch
> [root@bsd01~]#portsnap extract

#这 2 步可以合成使用：

> [root@bsd01~]#portsnap fetch extract #portsnapfecth 是从网上获取 portsnap 快照的最新压缩包，听闻这个压缩包官方没小时更 新一次。

#portsnapextract则是把这个压缩包创立到/usr/ports。哪怕你以前已经手工安装了 ports， 他也会重新创立一次。
 ③以后使用 portsnap 更新 ports：
以后更新，只需要执行下面 2 步：

> [root@bsd01~]#portsnap fetch
> [root@bsd01~]#portsnap update

这 2 步可以合成使用：

> [root@bsd01~]#portsnap fetch update

portsnap 第一次运行 extract 命令时，可能需要一段时间，以后更新使用 update 的时候， 速度就块很多了。

[FreeBSD学习笔记整理(内容取自chinaunix)PDF版下载](/wp-content/uploads/2010/12/FreeBSD_style_from_chinaunix.rar)