---
title: FreeBSD下如何开启SSH
author: admin
type: post
date: 2009-01-01T11:02:05+00:00
excerpt: |
 首先vi编辑/etc/inetd.conf,去掉ssh前的#，保存退出
 编辑/etc/rc.conf
 最后加入:sshd_enable="yes"即可
 激活sshd服务：
 techo#/etc/rc.d/sshd start
 用下面命令检查服务是否启动，在22端口应该有监听。
 #netstat -an ## check port number 22
 最后
 vi /etc/ssh/sshd_config
url: /archives/776
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ssh

---
首先vi编辑/etc/inetd.conf,去掉ssh前的#，保存退出

 编辑/etc/rc.conf

 最后加入:sshd_enable=”yes”即可

 激活sshd服务：

 techo#/etc/rc.d/sshd start

 用下面命令检查服务是否启动，在22端口应该有监听。

 #netstat -an ## check port number 22

 最后

 vi /etc/ssh/sshd_config,

下面是我的配置文件：(/etc/ssh/sshd_config)
####################################################

\# $OpenBSD: sshd_config,v 1.72 2005/07/25 11:59:40 markus Exp $
\# $FreeBSD: src/crypto/openssh/sshd_config,v 1.42.2.1 2005/09/11 16:50:35 des Exp $

\# This is the sshd server system-wide configuration file. See
\# sshd_config(5) for more information.

\# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

\# The strategy used for options in the default sshd_config shipped with
\# OpenSSH is to specify options with their default value where
\# possible, but leave them commented. Uncommented options change a
\# default value.

\# Note that some of FreeBSD’s defaults differ from OpenBSD’s, and
\# FreeBSD has a few additional options.

#VersionAddendum FreeBSD-20050903

#Port 22
#Protocol 2
#AddressFamily any
#ListenAddress 10.1.10.196
#ListenAddress ::

\# HostKey for protocol version 1
#HostKey /etc/ssh/ssh\_host\_key
\# HostKeys for protocol version 2
#HostKey /etc/ssh/ssh\_host\_dsa_key

\# Lifetime and size of ephemeral version 1 server key
#KeyRegenerationInterval 1h
#ServerKeyBits 768

\# Logging
\# obsoletes QuietMode and FascistLogging
#SyslogFacility AUTH
#LogLevel INFO

\# Authentication:

#LoginGraceTime 2m
#PermitRootLogin no
#StrictModes yes
#MaxAuthTries 6

#RSAAuthentication yes
#PubkeyAuthentication yes
#AuthorizedKey .ssh/authorized_keys
\# For this to work you will also need host keys in /etc/ssh/ssh\_known\_hosts
#RhostsRSAAuthentication no
\# similar for protocol version 2
#HostbasedAuthentication no
\# Change to yes if you don’t trust ~/.ssh/known_hosts for
\# RhostsRSAAuthentication and HostbasedAuthentication
#IgnoreUserKnownHosts no
\# Don’t read the user’s ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

\# Change to yes to enable built-in password authentication.
PasswordAuthentication yes
#PermitEmptyPasswords no

\# Change to no to disable PAM authentication
#ChallengeResponseAuthentication yes

\# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

\# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes

\# Set this to ‘no’ to disable PAM authentication, account processing,
\# and session processing. If this is enabled, PAM authentication will
\# be allowed through the ChallengeResponseAuthentication mechanism.
\# Depending on your PAM configuration, this may bypass the setting of
\# PasswordAuthentication, PermitEmptyPasswords, and
\# “PermitRootLogin without-password”. If you just want the PAM account and
\# session checks to run without PAM authentication, then enable this but set
\# ChallengeResponseAuthentication=no
#UsePAM yes

#AllowTcpForwarding yes
#GatewayPorts no
#X11Forwarding yes
#X11DisplayOffset 10
#X11UseLocalhost yes
#PrintMotd yes
#PrintLastLog yes
#TCPKeepAlive yes
#UseLogin no
#UsePrivilegeSeparation yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /var/run/sshd.pid
#MaxStartups 10

\# no default banner path
#Banner /some/path

\# override default of no subsystems
Subsystem sftp /usr/libexec/sftp-server

IgnoreRhosts yes
IgnoreUserKnownHosts yes
PrintMotd yes
StrictModes no
RSAAuthentication yes
PermitRootLogin yes #允许root登录
PermitEmptyPasswords no #不允许空密码登录
PasswordAuthentication yes # 设置是否使用口令验证。

##############################################

记得修改完配置文件后，重新启动sshd服务器(/etc/rc.d/sshd restart)即可。