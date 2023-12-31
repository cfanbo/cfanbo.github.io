---
title: 在FreeBSD上建立一个功能完整的邮件服务器(POSTFIX)
author: admin
type: post
date: 2009-11-11T08:52:57+00:00
excerpt: |
 |
 1．0 安装clamav:

 # cd /usr/ports/security/clamav
 # make install
 # make clean

 # vi /usr/local/etc/clamav.conf
 ===============================clamav.conf============================
 # Comment or remove the line below.
 # Example
 LogFile /var/log/clamav/clamd.log
 LogFileMaxSize 1M
 LogTime
 LogVerbose
url: /archives/2589
IM_contentdowned:
 - 1
categories:
 - 服务器

---
第二部分：防病毒、垃圾邮件：clamav+amavisd-new+spam

欢迎大家转贴这个文章，但要保留下面的版权信息：

作者：llzqq
出处：www.chinaunix.net
联系：llzqq@126.com

1．0 安装clamav:

\# cd /usr/ports/security/clamav
\# make install
\# make clean

\# vi /usr/local/etc/clamav.conf
===============================clamav.conf============================
\# Comment or remove the line below.
\# Example
LogFile /var/log/clamav/clamd.log
LogFileMaxSize 1M
LogTime
LogVerbose
PidFile /var/run/clamav/clamd.pid
DataDirectory /usr/local/share/clamav
LocalSocket /tmp/clamd
StreamMaxLength 10M
MaxThreads 10
MaxDirectoryRecursion 15
User clamav
ScanMail
ScanArchive
ScanRAR
ArchiveMaxFileSize 10M
ArchiveMaxRecursion 5
ArchiveMaxFiles 1000
ClamukoScanOnOpen
ClamukoScanOnClose
ClamukoScanOnExec
ClamukoIncludePath /var/spool/virtual
ClamukoMaxFileSize 6M
ClamukoScanArchive
===============================clamav.conf============================

1.1 更新病毒库

\# /usr/local/etc/rc.d/clamav-freshclam.sh start

2.0 安装amavisd-new

\# cd /usr/ports/security/amavisd-new
\# make install
\# make clean

\# cd /usr/local/etc
\# mv amavisd.conf-dist amavisd.conf
\# vi amavisd.conf
============================== amavisd.conf ===============================
$MYHOME = ‘/var/amavis’;           # (default is ‘/var/amavis’)
$mydomain = ‘nero.3322.org’;      # (no useful default)
$daemon_user   = ‘vscan’;          # (no default;   customary: vscan or amavis)
$daemon_group = ‘vscan’;          # (no default;   customary: vscan or amavis)

$log_level = 0;

$sa\_spam\_subject_tag = ‘\*\\*\*SPAM\*\**’

$virus_admin = “root\@$mydomain”;
$spam_admin = “llzqq\@$mydomain”;
$mailfrom\_notify\_admin      = “llzqq\@$mydomain”;
$mailfrom\_notify\_recip      = “llzqq\@$mydomain”;
$mailfrom\_notify\_spamadmin = “llzqq\@$mydomain”;

$inet\_socket\_bind = ‘127.0.0.1’;
$forward_method = ‘smtp:127.0.0.1:10025’;
$notify\_method = $forward\_method;
$inet\_socket\_port = 10024;
$max_servers   =   2;

[‘Clam Antivirus-clamd’,
\&ask_daemon, [“CONTSCAN {}\n”, ‘/tmp/clamd’],
qr/\bOK$/, qr/\bFOUND$/,
qr/^.\*?: (?!Infected Archive)(.\*) FOUND$/ ],
============================== amavisd.conf ===============================

2.1 要启动clamav和amavisd-new需要配置一下/etc/rc.conf

\# vi /etc/rc.conf

spamd_enable=”YES”
amavisd_enable=”YES
clamav\_clamd\_enable=”YES”

3.0 由于在安装amavisd-new时spamassassin被一起安装了下面对其进行配置

3.1 建立过滤规则：

\# cd /usr/local/etc/mail/spamassassin
\# env LANG=C vi local.cf
=============================== local.cf ===============================
\# SpamAssassin config file for version x.xx
\# generated by http://www.yrex.com/spam/spamconfig.php (version 1.01)

\# How many hits before a message is considered spam.
required_hits 4.0

\# Whether to change the subject of suspected spam
rewrite_subject 1

\# Text to prepend to subject if rewrite_subject is used
subject_tag **\*\\*\*SPAM\*\*\***

\# Encapsulate spam in an attachment
report_safe 1

\# Use terse version of the spam report
use\_terse\_report 0

\# Enable the Bayes system
use_bayes 1

\# Enable Bayes auto-learning
auto_learn 1

\# Enable or disable network checks
skip\_rbl\_checks 1
use_razor2 0
use_dcc 0
use_pyzor 0

\# Mail using languages used in these country codes will not be marked
\# as being possibly spam in a foreign language.
\# – chinese english
ok_languages zh en

\# Mail using locales used in these country codes will not be marked
\# as being possibly spam in a foreign language.
ok_locales en zh
score SUBJ\_FULL\_OF_8BITS 2
score NO\_REAL\_NAME 4.0
=============================== local.cf ===============================

3.2 下载新的垃圾邮件地址列表文件

\# cd /usr/local/share/spamassassin
\# fetch http://anti-spam.org.cn/rules/sa/55\_diy\_score.cf

4.0 对POSFIX进行配置，在他的配置文件中添加下面的一些内容

\# vi /usr/local/etc/postfix/master.cf

———————- master.cf ———————
smtp-amavis unix –    –    n      –        2   smtp
-o smtp\_data\_done_timeout=1200
-o disable\_dns\_lookups=yes

127.0.0.1:10025 inet n –        n        –        –   smtpd
-o content_filter=
-o local\_recipient\_maps=
-o relay\_recipient\_maps=
-o smtpd\_restriction\_classes=
-o smtpd\_client\_restrictions=
-o smtpd\_helo\_restrictions=
-o smtpd\_sender\_restrictions=
-o mynetworks=127.0.0.0/8
———————- master.cf ———————

\# vi /usr/local/etc/postfix/main.cf

content_filter = smtp-amavis:[127.0.0.1]:10024

好了，现在一个基于FreeBSD的功能相对完整的邮件服务器就建立起来了，虚拟域的管理员可以登陆OPENWEBMAIL进行用户的添加、删除等操作，虚拟用户可以通过OPENWEBMAIL修改自己的密码。