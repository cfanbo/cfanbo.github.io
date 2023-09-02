---
title: How to install PHP-fpm + Nginx on CentOS 5.3
author: admin
type: post
date: 2010-09-19T10:49:23+00:00
url: /archives/5774
IM_data:
 - 'a:3:{s:58:"http://adityo.blog.binusian.org/files/2009/07/php-fpm1.png";s:68:"http://blog.haohtml.com/wp-content/uploads/2011/03/e49a_php-fpm1.png";s:58:"http://adityo.blog.binusian.org/files/2009/07/php-fpm2.png";s:68:"http://blog.haohtml.com/wp-content/uploads/2011/03/15ae_php-fpm2.png";s:58:"http://adityo.blog.binusian.org/files/2009/07/php-fpm3.png";s:68:"http://blog.haohtml.com/wp-content/uploads/2011/03/86c0_php-fpm3.png";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - nginx

---
[php-fpm文档中文翻译[转]:http://blog.haohtml.com/archives/9294](http://blog.haohtml.com/archives/9294)

In this tutorial i will try to install php-fpm with nginx on centos 5.3. But first:

## What is PHP-FPM ?

PHP-FPM is a patch for PHP4/5 to greatly improve PHP’s FastCGI SAPI capabilities and administration. This means that you don’t install php through your distribution’s package manager, but rather download the package from the PHP-website, and the patch from another site. You patch up the sourcecode, compile things, and get it started.

The following is a comparison chart of problems and how php-fpm handles them, when enabled with the FastCGI SAPI:

 Description

 php “out of the box”

 spawn-fcgi + spawn-php.sh + daemontools

 php-fpm

 php daemonization: pid file, log file, setsid(), setuid(), setgid(), chroot()

 (-)

 (+)

 (+)

 Process Management. Ability to “graceful” stop and start php workers without losing any queries. The possibility of gradually update the configuration and binary without losing any queries.

 php4 (-), php5 (only graceful completion)

 (-)

 (+)

 Restricting ip addresses from which requests can come from a web server

 php4 (-), php5 (+) (from 5.2.2)

 (-)

 (+)

 Dynamic number of processes, depending on the load

 (-)

 (-)

 TODO

 Starting the workers with different uid/gid/chroot/environment and different php.ini option. You do not need a safe mode!

 (-)

 (-)

 (+)

 Logging stdout & stderr business processes

 (-)

 (-)

 (+)

 Emergency restart all the processes in the event of accidental destruction of shared memory opcode cache, if used accelerator

 (-)

 (-)

 (+)

 Forcing the completion of process if set_time_limit() fails

 (-)

 (-)

 (+)


Compare to spawn- fastcgi installation , php-fpm installation is much slower and you can refer to [http://www.cyberciti.biz/faq/rhel-fedora-install-configure-nginx-php5/](http://www.cyberciti.biz/faq/rhel-fedora-install-configure-nginx-php5/) for spawn-fcgi installation. Okay let’s start the installation.

**Installation**

1. Install some dependecies first
`# yum install gcc libjpeg-devel libpng-devel libmcrypt libmcrypt-devel pcre pcre-devel`

2. We need to get PHP-sourcecode and php-fpm patch, in this tutorial i use php-5.2.10 and php-5.2.10-fpm-0.5.13
`# wget http://id.php.net/get/php-5.2.10.tar.bz2/from/us.php.net/mirror`
`wget http://php-fpm.org/downloads/php-5.2.10-fpm-0.5.13.diff.gz`

3. Extract and run the patch
`# tar xvf php-5.2.10.tar.bz2`
`# gzip -cd php-5.2.10-fpm-0.5.13.diff.gz | patch -d php-5.2.10 -p1`

4. Configure and compile cd php-5.2.10 and hp-5.2.10-fpm-0.5.13 patch
`# ./configure --enable-fastcgi --enable-fpm --with-mcrypt --enable-mbstring --enable-mysql --with-mysql=/usr/include/mysql --with-mysql-sock=/tmp/mysql.sock --with-curl --with-sockets --with-gd --with-zlib --with-iconv --with-dom --with-jpeg-dir=/usr/lib`
`make`
`make install`

You should see this on the end of installation

`

Installing PHP SAPI module: cgi

Installing PHP CGI binary: /usr/local/bin/

Installing FPM config: /usr/local/etc/php-fpm.conf

(installing as php-fpm.conf.default)

Installing init.d script: /usr/local/sbin/php-fpm

Installing PHP CLI binary: /usr/local/bin/

Installing PHP CLI man page: /usr/local/man/man1/

Installing build environment: /usr/local/lib/php/build/

Installing header files: /usr/local/include/php/

^[[BInstalling helper programs: /usr/local/bin/

program: phpize

program: php-config

Installing man pages: /usr/local/man/man1/

page: phpize.1

page: php-config.1

Installing PEAR environment: /usr/local/lib/php/

pear/pear dependency package "pear/Archive_Tar" installed version 1.3.3 is not the recommended version

1.3.2, but may be compatible, use --force to install

pear/Archive_Tar cannot be installed, conflicts with installed packages

[PEAR] Archive_Tar - upgraded: 1.3.3

[PEAR] Console_Getopt - already installed: 1.2.3

[PEAR] Structures_Graph- already installed: 1.0.2

[PEAR] XML_Util - installed: 1.2.1

[PEAR] PEAR - upgraded: 1.8.0

Wrote PEAR system config file at: /usr/local/etc/pear.conf

You may want to add: /usr/local/lib/php to your php.ini include_path

Installing PDO headers: /usr/local/include/php/ext/pdo/`

5. Installing Init Script for PHP-FPM
`# cd /etc/init.d/

# ln -s /usr/local/sbin/php-fpm php-fpm`

add this to /etc/rc.local if you want php-fpm to start from booting
`# vi /etc/rc.local`
`/etc/init.d/php-fpm start`

6. Installing and configure nginx, when i write this tutorial the latest stable versions are nginx-0.7.61 you can go to  for latest nginx sourcode
`wget http://sysoev.ru/nginx/nginx-0.7.61.tar.gz`
`tar xvf http://sysoev.ru/nginx/nginx-0.7.61.tar.gz`
`cd nginx-0.7.61`
`./configure --pid-path=/usr/local/nginx/logs/nginx.pid --sbin-path=/usr/local/sbin/nginx --with-md5=/usr/lib --with-sha1=/usr/lib --with-http_ssl_module --with-http_dav_module --without-mail_pop3_module --without-mail_imap_module --without-mail_smtp_module`
`# make`
`# make install`

7. Installing Nginx Daemon for CentOS, you can copy paste this script to your /etc/init.d/nginx
`

#! /bin/sh

### BEGIN INIT INFO

# Provides: skeleton

# Required-Start: $local_fs $remote_fs

# Required-Stop: $local_fs $remote_fs

# Default-Start: 2 3 4 5

# Default-Stop: S 0 1 6

# Short-Description: Example initscript

# Description: This file should be used to construct scripts to be

# placed in /etc/init.d.

### END INIT INFO

#

# Author: Ryan Norbauer

#

set -e

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

DESC="nginx daemon"

NAME=nginx

DAEMON=/usr/local/sbin/$NAME

PIDFILE=/usr/local/nginx/logs/$NAME.pid

SCRIPTNAME=/etc/init.d/$NAME

# Gracefully exit if the package has been removed.

test -x $DAEMON || exit 0

# Read config file if it is present.

#if [ -r /etc/default/$NAME ]

#then

# . /etc/default/$NAME

#fi

#

# Function that starts the daemon/service.

#

d_start() {

start-stop-daemon --start --quiet --pidfile $PIDFILE \

--exec $DAEMON \

|| echo -n " already running"

}

#

# Function that stops the daemon/service.

#

d_stop() {

start-stop-daemon --stop --quiet --pidfile $PIDFILE \

--name $NAME \

|| echo -n " not running"

}`

`#

# Function that sends a SIGHUP to the daemon/service.

#

d_reload() {

start-stop-daemon --stop --quiet --pidfile $PIDFILE \

--name $NAME --signal 1

}`

`case "$1" in

start)

echo -n "Starting $DESC: $NAME"

d_start

echo "."

;;

stop)

echo -n "Stopping $DESC: $NAME"

d_stop

echo "."

;;

#reload)

#

# If the daemon can reload its configuration without

# restarting (for example, when it is sent a SIGHUP),

# then implement that here.

#

# If the daemon responds to changes in its config file

# directly anyway, make this an "exit 0".

#

# echo -n "Reloading $DESC configuration..."

# d_reload

# echo "done."

#;;

restart|force-reload)

#

# If the "reload" option is implemented, move the "force-reload"

# option to the "reload" entry above. If not, "force-reload" is

# just the same as "restart".

#

echo -n "Restarting $DESC: $NAME"

d_stop

# One second might not be time enough for a daemon to stop,

# if this happens, d_start will fail (and dpkg will break if

# the package is being upgraded). Change the timeout if needed

# be, or change d_stop to have start-stop-daemon use --retry.

# Notice that using --retry slows down the shutdown process somewhat.

sleep 1

d_start

echo "."

;;

*)

echo "Usage: $SCRIPTNAME {start|stop|restart|force-reload}" >&2

exit 3

;;

esac

exit 0

`

Or you can download it from http://www.magnet-id.com/download/nginx/nginx-daemon
`wget http://www.magnet-id.com/download/nginx/nginx-daemon -O /etc/init.d/nginx`
and don’t forget to set the permission
`# chmod 750 /etc/init.d/nginx`

Install start-stop-daemonvand add nginx on the start up
`

wget http://www.magnet-id.com/download/nginx/apps-sys-utils-start-stop-daemon-IR1_9_18-1.tar.gz

tar zxvf apps-sys-utils-start-stop-daemon-IR1_9_18-1.tar.gz

cd apps/sys-utils/start-stop-daemon-IR1_9_18-1/

gcc start-stop-daemon.c -o start-stop-daemon

cp start-stop-daemon /usr/sbin

chkconfig --add nginx

chkconfig --level 345 nginx on

`

Nginx.conf configuration

go to nginx configuration file: “/usr/local/nginx/conf/nginx.conf” , and add the php configuration
`

location ~ \.php$ {

root /usr/share/nginx/html; # itmena the root of the coument are located on /usr/share/nginx/html

fastcgi_pass 127.0.0.1:9000;

fastcgi_index index.php;

fastcgi_param SCRIPT_FILENAME /usr/share/nginx/html$fastcgi_script_name;

include fastcgi_params;

}

`

Go to /usr/share/nginx/html and create php files
`# vi index.php

`

8. Start the nginx and php-fpm
`# service php-fpm start`
`# service nginx start`
9. You should go to your domain or server address and see the phpinfo

[![php-fpm1](http://adityo.blog.binusian.org/files/2009/07/php-fpm1.png)][1]

[![php-fpm2](http://adityo.blog.binusian.org/files/2009/07/php-fpm2.png)][2]

[![php-fpm3](http://adityo.blog.binusian.org/files/2009/07/php-fpm3.png)][3]

 [1]: http://adityo.blog.binusian.org/files/2009/07/php-fpm1.png
 [2]: http://adityo.blog.binusian.org/files/2009/07/php-fpm2.png
 [3]: http://adityo.blog.binusian.org/files/2009/07/php-fpm3.png