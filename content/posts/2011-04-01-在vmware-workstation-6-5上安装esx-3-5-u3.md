---
title: 在VmWare Workstation 6.5上安装Esx 3.5 U3
author: admin
type: post
date: 2011-04-01T12:16:30+00:00
url: /archives/8866
IM_data:
 - 'a:23:{s:73:"http://image4.it168.com/2009/2/3/67d10d00-c240-47d8-8106-4aa6eade0c77.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/ab88_67d10d00-c240-47d8-8106-4aa6eade0c77.jpg";s:73:"http://image4.it168.com/2009/2/3/6ae8ddd4-7f09-4127-91fa-cf8027485df6.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/af9f_6ae8ddd4-7f09-4127-91fa-cf8027485df6.jpg";s:73:"http://image4.it168.com/2009/2/3/7727e6bf-3f97-428b-b954-2428ebcf3c6d.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/a0c8_7727e6bf-3f97-428b-b954-2428ebcf3c6d.jpg";s:73:"http://image4.it168.com/2009/2/3/57362939-6997-43e2-8e82-fabd944f79dd.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/db3c_57362939-6997-43e2-8e82-fabd944f79dd.jpg";s:73:"http://image4.it168.com/2009/2/3/303d8236-5589-45ba-adc5-deda5a5eb1cf.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/0907_303d8236-5589-45ba-adc5-deda5a5eb1cf.jpg";s:73:"http://image4.it168.com/2009/2/3/9c2253c4-ef18-4daf-a77e-c043372c957a.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/471a_9c2253c4-ef18-4daf-a77e-c043372c957a.jpg";s:73:"http://image4.it168.com/2009/2/3/c198b285-22f5-4b97-9b60-48123daa56d2.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/5eaf_c198b285-22f5-4b97-9b60-48123daa56d2.jpg";s:73:"http://image4.it168.com/2009/2/3/ab9ee417-5131-41ef-abb9-473053d6db43.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/e497_ab9ee417-5131-41ef-abb9-473053d6db43.jpg";s:73:"http://image4.it168.com/2009/2/3/691e0662-47ec-4dba-bbdf-c56271bc0606.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/ac99_691e0662-47ec-4dba-bbdf-c56271bc0606.jpg";s:73:"http://image4.it168.com/2009/2/3/54094b7f-6bcf-42c6-bac7-a374aa5aac9c.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/625c_54094b7f-6bcf-42c6-bac7-a374aa5aac9c.jpg";s:73:"http://image4.it168.com/2009/2/3/92d255e9-6774-4b04-a826-d0bc8eb04c6d.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/d010_92d255e9-6774-4b04-a826-d0bc8eb04c6d.jpg";s:73:"http://image4.it168.com/2009/2/3/503094ea-7534-406a-9924-66f79051435e.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/b008_503094ea-7534-406a-9924-66f79051435e.jpg";s:73:"http://image4.it168.com/2009/2/3/15395d6f-0ff6-438f-95d1-b6be85c22fc8.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/ec3d_15395d6f-0ff6-438f-95d1-b6be85c22fc8.jpg";s:73:"http://image4.it168.com/2009/2/3/30744eab-cba8-4692-b552-8b7b24c864ea.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/29fd_30744eab-cba8-4692-b552-8b7b24c864ea.jpg";s:73:"http://image4.it168.com/2009/2/3/08d22650-7727-4d71-bfa3-48920e057b25.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/26c0_08d22650-7727-4d71-bfa3-48920e057b25.jpg";s:73:"http://image4.it168.com/2009/2/3/d3a3ec06-813a-4480-a6b8-eadbfaaa3b0b.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/3325_d3a3ec06-813a-4480-a6b8-eadbfaaa3b0b.jpg";s:73:"http://image4.it168.com/2009/2/3/16d5d515-1ea3-43a6-8a6a-76fbed2eefcd.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/b8ca_16d5d515-1ea3-43a6-8a6a-76fbed2eefcd.jpg";s:73:"http://image4.it168.com/2009/2/3/06f76376-30a9-4519-9e01-ef38e3c5eb96.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/290f_06f76376-30a9-4519-9e01-ef38e3c5eb96.jpg";s:73:"http://image4.it168.com/2009/2/3/69fdbba9-0409-451f-93b1-8996ab7a817d.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/7e90_69fdbba9-0409-451f-93b1-8996ab7a817d.jpg";s:73:"http://image4.it168.com/2009/2/3/15cc92b3-08d6-4592-a45c-c1c6fb283092.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/9a24_15cc92b3-08d6-4592-a45c-c1c6fb283092.jpg";s:73:"http://image4.it168.com/2009/2/3/5f561123-e764-4222-88a8-2b605f1a52fa.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/fd5c_5f561123-e764-4222-88a8-2b605f1a52fa.jpg";s:73:"http://image4.it168.com/2009/2/3/81bab121-f711-4fd3-9862-559119a8caab.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/6b05_81bab121-f711-4fd3-9862-559119a8caab.jpg";s:73:"http://image4.it168.com/2009/2/3/11647973-b5bc-4b76-b94d-7b21ba3f12ac.jpg";s:96:"http://blog.haohtml.com/wp-content/uploads/2011/04/6bee_11647973-b5bc-4b76-b94d-7b21ba3f12ac.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 其它
tags:
 - ESX
 - vmware

---
虚拟化给企业及社区带来了很多益处，同时，拥有虚拟化技术使用经验也让很多IT PRO人员有了工作稳定及高额工资的保证。但近期，与虚拟化形式的喜人发展所不协调的是整体金融形式的始终萎靡，这就限制了很多IT 人员投资于学习使用虚拟化的成本。签于此，参考国外同行所写的英文文档，略加改动以中文的形式来介绍一种如何利用现有硬件资源和软件资源来学习使用VM ESX 的方法。

本文亦是使用SCVMM2008管理虚拟化解决方案系列之一部分。

本文将通过图文的方式介绍如何在VMware Workstation 6.5.0 build 118166安装VM Esx 3.5 Update 3。（在未来的几周内会陆续写出使用VIC 或 VC来管理ESX，以及最终通过VMM来管理VC等系列文章。）以及通过VIC2.5来连接ESX 3.5并在其上创建虚拟机windows xp sp3。

 **一、软硬件需求**

1、 硬件环境：

ThinkPad X61 Intel Core2 Duo CPU T7500 2.19Ghz ，4G内存，160Gsata硬盘

注意：这是偶的个人电脑，偶的大部分虚拟化实验之旅是在此机器上完成的，而要完成本次文章所求之目的，请保证你的硬盘可用空间到少30G左右。

2、 软件需求：

A、 windows server 2003 Ent with sp2为本本的操作系统。

B、 VMware Workstation 6.5.0 build 118166 在本本上已安装。

C、 VM Esx 3.5 Update 3 将安装于VMware Workstation 6.5中。

注意：B、VMware Workstation 6.5.0 build 118166已完成安装，且使用一直良好，相信大部分朋友都有安装的经验，在此文中将不会出现安装方法。



**二、在VMware 6.5创建ESX虚拟机器**

1、在VM6.5控制台，开始新建虚拟机向导，并在虚机新建欢迎页选择”Custom(advanced)”选项。下一步：

注意：相对于VMware 6.5来说，将要安装于此上的ESX 3.5只是一个虚拟机器的角色。

和其它的虚拟机器的待遇是平等的。只是安装的步骤和设置的细节不同罢了。

2、在”Choose the Virtual Machine Hardware Compatibility”界面的”Hardware Compatibility”后框中，确保选中Workstation 6.5。下一步：（图1）

![](http://image4.it168.com/2009/2/3/67d10d00-c240-47d8-8106-4aa6eade0c77.jpg)

3、 在”Guest Operating System Installation”界面，选择’I will install the operating system later’。下一步：（图2）

![](http://image4.it168.com/2009/2/3/6ae8ddd4-7f09-4127-91fa-cf8027485df6.jpg)

4、 在”Select a Guest Operating System”界面选择”Red Hat Enterprise Linux 4 64-bit”。下一步：（图3）

![](http://image4.it168.com/2009/2/3/7727e6bf-3f97-428b-b954-2428ebcf3c6d.jpg)

5、 在”Name the Virtual Machine”界面，在”Virtual Machine name”下框中输入虚机的名机:ESX3.5U3，在”Location”下框中输入本地存储ESX虚拟机的位置F:\vmwaresx35。当然亦可浏览至任何文件夹下，但要保证磁盘可用空间足够大（最好在30G左右，以便未来可以在ESX中安装多个虚拟机器），这和你规划中的实验环境有关。下一步：（图4）

![](http://image4.it168.com/2009/2/3/57362939-6997-43e2-8e82-fabd944f79dd.jpg)

6、 在”Processor Configuration”界面，选择CPU的数量为one。下一步：

7、 在”Memory for the Virtual Machine”界面，选择memory的大小为1024Mb。下一步：

8、 在”Network Type”界面，选择网络连接类型为：Use bridged networking，下一步：

9、 在”Select I/O Adapter Types”界面，在”I/O Adapter Types”后，选择”LSI Logic”，下一步：

10、 在”Select a Disk”界面，选择”Creat a new virtual disk”。下一步：

11、 在”Select a Disk Type”界面，一定要选择磁盘类型为SCSI。下一步：

12、 在”Specify Disk Capacity”界面，磁盘大小可以根据自己今后的实验规划以及硬盘空间大小来定，此处填入30GB。但下述两项一定要选择上：Allocate all disk space now和Store virtual disk as a single file。切记。下一步：（图5）

![](http://image4.it168.com/2009/2/3/303d8236-5589-45ba-adc5-deda5a5eb1cf.jpg)

13、 在”Specify Disk File”界面，你可以选择定义磁盘文件，此处保留默认。下一步：

14、 在”Ready to Create Virtual Machine”，确保”Power on this virtual machine after creation”之前的对勾不被打上。并点击”Customize Hardware”。（图6）

![](http://image4.it168.com/2009/2/3/9c2253c4-ef18-4daf-a77e-c043372c957a.jpg)

15、 在”HardWare”界面，remove Floppy, USB Controller and Sound。

16、 在”HardWare”界面，选择”Network Adapter”，并在右侧的框中确保如图中绿色部分的选择：（图7）

![](http://image4.it168.com/2009/2/3/c198b285-22f5-4b97-9b60-48123daa56d2.jpg)

17、 在”HardWare”界面，选择”Display”选项，确保”Accelerate 3D graphics”前面的对勾不被选择：（图8）

![](http://image4.it168.com/2009/2/3/ab9ee417-5131-41ef-abb9-473053d6db43.jpg)

18、 在”HardWare”界面，选择”Processors”选项，并确保在”Preferred Mode”后面”Intel-VTx or AMD-V”被选择。并单击”OK”。（图9）

![](http://image4.it168.com/2009/2/3/691e0662-47ec-4dba-bbdf-c56271bc0606.jpg)

19、 在返回的”Ready to Create Virtual Machine”界面，单击”Finish”。便会出现磁盘文件创建的进度。这个过程将要花费几分钟到十几分钟的时间。（图10）

![](http://image4.it168.com/2009/2/3/54094b7f-6bcf-42c6-bac7-a374aa5aac9c.jpg)

20、 上述过程完成后，双击”CD/DVE (IDE)”选项，并在接下来的界面中选择”Use ISO image file”。浏览至存放有镜像的文件夹（当然你使用物理光驱也可以，这方面不做太多要求）。并OK。

21、 此时，你仍不能启动虚机，进行安装ESX的进程。还需要添加一行文字到.VMX文件里才可以。

在存放ESX 3.5安装文件的路径F:\vmwaresx35，找到ESX3.5U3.vmx（注意安装路径不同，此文件的位置也不同。请根据自己的实际安装路径及ESX磁盘文件名称来定）。使用记事本进行编辑，并添加”monitor\_control.restrict\_backdoor = “true””（没有外引号）在适当的如图中所示的位置：（图11）

![](http://image4.it168.com/2009/2/3/92d255e9-6774-4b04-a826-d0bc8eb04c6d.jpg)



**三、安装VMware ESX 3.5 Update3**

在二、中，设置好了ESX 3.5在VMware 6.5中得以顺利安装的环境，接下来就要进行ESX 3.5 U3的安装了。

安装ESX 3.5 U3的过程和在VMware 6.5里安装Red Hat Linux的方法基本上一样，所以有LINUX安装基础的朋友，可以很顺利的完成。

1、 启动VMware 6.5里的ESX 3.5 U3虚机（文章前面部已说过对VMWARE WORKSTATION来说，ESX3.6就是一台虚机而已。），然后会弹出一个类似于Red Hat Linux的安装向导。接下来就按Linux来安装吧。此处偶选择的是图形安装方式。（图12）

![](http://image4.it168.com/2009/2/3/503094ea-7534-406a-9924-66f79051435e.jpg)

2、 在安装向导的整个过程中，需要说明的就是针对ESX 3.5的分区，如果你对LINUX的安装很熟悉，那么你可能会很快完成此项任务。如果你是位新生，我建议你保留默认。也就是说你选择由ESX来自行决定分区类型及大小便可。在实际应用中，建议你和VMWARE的工程师做好详细的安装及扩展计划。如下图所示的内容，你需要好好的研究下了：（图13）

![](http://image4.it168.com/2009/2/3/15395d6f-0ff6-438f-95d1-b6be85c22fc8.jpg)

3、 在安装过程中也会提示你设置ESX的网络，一般要设置三种网络接口：服务控制台、与物理网络相连的网络、最好再设置个VMOTION接口网络（用于快速迁移）。由于此处咱们只是测试使用，只使用一种网络接口用于管理以及和物理网络相连接。

4、 在安装过程中，应注意磁盘的类型，在之前的ESX版本中要使用SCSI硬盘。而在新的版本中可以使用SATA硬盘了（存储虚机等）。不过据我测试使用来看，亦可以安装在IDE硬盘中，只是不能在此上安装虚机罢了，因为找不到合适的存放虚机等资源的存储。呵呵。

5、 安装完成的界面如下图，可以通过提示选择ALT F1进入和LINUX一样的登陆界面，但是VMWARE还是在ESX上开发出了自己的一些命令。一般以ESXCFG开头。你可以通过输入ESX并按TAB键来查看一些。（图14）

![](http://image4.it168.com/2009/2/3/30744eab-cba8-4692-b552-8b7b24c864ea.jpg)



**四、管理ESX 3.5 Update3，并在此后安装虚拟机**

通过上述一、二、三、已完成了在VMware Workstation 6.5中安装ESX 3.5 U3，并且验证是可以使用。并且在此三个步骤中ESX被看作是Workstation 6.5的一个虚拟机。

现在，是你转变思维的时侯了。也就是说你要把Workstation 6.5看作是不存在，并且把ESX 3.5看作是安装在物理机器上，当做一个主机使用，并且在此ESX 3.5主机上安装的操作系统被称为虚拟机器。

OK，就如安装完HYPER-V SERVER 2008一样，都是字符界面。该如何管理以及安装虚拟机器呢？

在VMware里提供完整的解决方案，一是通过Vmware Infrastructure Client (VIC)来连接ESX3.5主机进行管理；一是通过Virtual Center（简称VC）连接ESX主机，而再由Vmware Infrastructure Client连接VC来进行管理。前者提供的管理功能较为简单，而后者结合VMOTION、UPDATE MANAGER等组成了一套完整的成熟的管理解决方案。当然通过Web Access也能做一些事情。

OK，VIC从何处下载？如何使用此连接ESX？又如何使用此在ESX上安装虚拟机器呢？接下来将逐个解决此问题：

1、 通过WEB下载VIC，并在物理主机上的操作系统上安装。

VIC是可以安装WINDOWS平台的机器上的。基本上除了ESX是LINUX平台。其他的应用都可以在WINDOWS平台上使用。打开IE浏览器，并在地址栏输入http://192.168.1.251(在三中最后个图示中可以看到提示啦)。（图15）

![](http://image4.it168.com/2009/2/3/08d22650-7727-4d71-bfa3-48920e057b25.jpg)

呵呵，不错吧，由于系统是简体中文版，所以此处提示的亦为简体中文界面，这种本地化让我们管理起VMWARE的虚拟化解决方案来更加得心应用。

然后选择图中的”下载VMware Infrastructure Client”,这时就会从ESX上下载VIC，直接安装便可以。当然你也可以选择下载VC，但会转到VMWARE的官网上。而且这个是要付费的。

注意，如果你想使用Web Access来访问管理ESX，请选择右侧的”登陆Web Access”选项。

2、 安装完VIC后，便可以使用此连接ESX并进行管理的操作了。如图，输入ESX3.5的IP地址、用户名（root）和密码，点登陆。（图16）

![](http://image4.it168.com/2009/2/3/d3a3ec06-813a-4480-a6b8-eadbfaaa3b0b.jpg)

在登录过程中会出现”安全警告”的提示，由于此环境中并没有证书服务器。此处可以选择”忽略”，并不影响正常连接。

3、 连接登录上后，就会显示如下图所示的界面：（图17）

![](http://image4.it168.com/2009/2/3/16d5d515-1ea3-43a6-8a6a-76fbed2eefcd.jpg)

简体中文的界面，很清楚明了的显示着各种可以操作的功能及按钮。建议好好研究下，最好是能把帮助文件找出来仔细阅读，是不错的学习和参考资料。

4、 连接之后，就可以在ESX主机上安装虚拟机了。在VIC控制台右侧的基本任务栏，点”创建新的虚拟机”，此时，会弹出一个新建虚拟机向导。

有两种创建虚机的方式，一种是典型，一种是自定义。选择典型后，可以在向导面板的右侧看到一些常见的设置项，只需按自己的要求设置好后，下一步就行了。（图18）

![](http://image4.it168.com/2009/2/3/06f76376-30a9-4519-9e01-ef38e3c5eb96.jpg)

5、 经过几次的下一步，如下图显示了我对XP的设置。（图19）

![](http://image4.it168.com/2009/2/3/69fdbba9-0409-451f-93b1-8996ab7a817d.jpg)

6、 对新建虚机WINXP按向导配置好后，还可以在VIC的管理控制台对此进行一些简要的设置如下图：（图20）

![](http://image4.it168.com/2009/2/3/15cc92b3-08d6-4592-a45c-c1c6fb283092.jpg)

从图中可知，可以对新建虚拟机WINXP进行如”硬件”、”选项”、”资源”等的再设置。”硬件”中要注意的是选择安装XP时需要的安装镜像或文件的位置，其中的”数据存储ISO文件”是指的在ESX中已放置的ISO文件。但此场景并没有预选存放ISO于ESX上。所以，选择”客户端设备”（看清楚此下面的提示）。

较为重要的是”资源”选项，可对关键的CPU、内存、磁盘等的使用进行资源分配。（图21）

![](http://image4.it168.com/2009/2/3/5f561123-e764-4222-88a8-2b605f1a52fa.jpg)

7、 完成上述工作后，就可以在VIC管理控制台”启动虚拟机”了。并点”启运虚拟机控制台”按钮。弹出如下窗口。（图22）

![](http://image4.it168.com/2009/2/3/81bab121-f711-4fd3-9862-559119a8caab.jpg)

选择图中标为绿色的”连接CD/DVD”，并在弹出的窗口中选定自己的安装镜像文件。接下来的操作就是物理机上安装XP是一样的流程。此处略。

8、 安装完成后，最好安装上VMWARE TOOLS。（图23）

![](http://image4.it168.com/2009/2/3/11647973-b5bc-4b76-b94d-7b21ba3f12ac.jpg)

终于完成了。这些操作步骤都较为简单。并不说明学习和使用VMWARE的虚拟化也是如此的简单。但愿通过这些操作能给你带来学习的一些方法或思路。同进重点建议你加深理解ESX的虚拟网络定义及设置。

此篇文章的另一大目的就是为使用VMM 2008来管理VC及ESX做准备的。所以接下来将把重点转移到VMM2008上。至于VMWARE VC及ESX的学习及使用在有精力时再向大家学习和探讨。

转自: