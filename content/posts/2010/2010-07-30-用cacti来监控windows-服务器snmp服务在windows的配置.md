---
title: 用cacti来监控windows 服务器,snmp服务在windows的配置
author: admin
type: post
date: 2010-07-30T01:27:54+00:00
url: /archives/4880
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - CACTI

---
**监控客户端windows2003服务器的snmp服务配置(202.96.209.2)**

(1)、打开“控制面板”—“添加删除程序”—“添加删除组建”，在“管理和监视工具”中选中“简单网络管理协议（snmp）”，点击“下一步”，开始安装，在安装过程中需要i386文件.

(2)、打开“开始”—“程序”—“管理工具”—“服务”，找到“snmp service”，右键打开“属性”，选择“安全”，在“接受团体名称”处，点“添加”，在“团体名称”处写入你的cacti使用的community，选中“接受来自这些主机的snmp数据包”，默认值为“localhost”，点击“编辑”，将“localhost”改为**cacti监控服务器的实际 ip地址**.（指定要接收哪些主机的snmp数据,这里要填写上运行cacti程序的服务器ip地址）

(3)、还需要安装SNMP Informant-STD 1.6 软件下载地址： [http://www.wtcs.org/informant/download.htm](http://www.wtcs.org/informant/download.htm)
有防火墙的要开通UDP端口161(可以在cmd命令行下输入：netstat -an 来查看udp协议的161端口是否在监听，如果找不到,则表示配置错误)

```

```

```
cacti服务器(202.96.209.5)
```

1、测试监控机的snmp连接

>

```
# snmpwalk -v2c -c private 202.96.209.2 system
SNMPv2-MIB::sysDescr.0 = STRING: Hardware: x86 Family 6 Model 15 Stepping 7 AT/AT COMPATIBLE – Software: Windows Version 5.2 (Build 3790 Multiprocessor Free)
SNMPv2-MIB::sysObjectID.0 = OID: SNMPv2-SMI::enterprises.311.1.1.3.1.2
DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (7862939) 21:50:29.39
SNMPv2-MIB::sysContact.0 = STRING:
SNMPv2-MIB::sysName.0 = STRING: CHINESE-FD21F3C
SNMPv2-MIB::sysLocation.0 = STRING:
SNMPv2-MIB::sysServices.0 = INTEGER: 76
```

显示这个说明连接正常，如果不能正常连接，检查监控机snmp服务器是否正常还有防火墙有没有开放snmp的端口 udp 161

```
2、cacti模板文件 Windows XP/Win2000/Win2003/Vista/Win2008 Templates
下载地址 Cacti_SNMP_INFORMANT_STD_W32_Metrics.zip
解压后10个文件
cacti_data_query_w32_-_cpu_statistics.xml
cacti_data_query_w32_-_network_statistics.xml
cacti_data_query_w32_-_disk_statistics.xml
cacti_data_query_w32_-_object_statistics.xml
cacti_data_query_w32_-_memory_statistics.xml
以上文件在cactit管理界面，Import Templates导入。
snmp_informant_disk.xml
snmp_informant_objects.xml
snmp_informant_memory.xml
snmp_informant_cpu.xml
snmp_informant_network.xml
以上文件copy到服务器cacti安装目录的resource/snmp_queries里.
剩下的事情就是添加device还有graph，简单就不管啦。
```