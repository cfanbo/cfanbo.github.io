---
title: Centos下安装Docker
author: admin
type: post
date: 2013-12-11T08:58:14+00:00
url: /archives/14824
categories:
 - 服务器
tags:
 - docker

---
内核要求至少 3.8.0版本或以上. [https://docs.docker.com/linux/](https://docs.docker.com/linux/)

第二步要安装epel库

```
wget http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
sudo rpm -Uvh epel-release-6-8.noarch.rpm
```

此时,会在/etc/yum.repo.d/目录里生成两个文件epel.repo  epel-testing.repo.

默认情况下epel.repo已经启用了enabled=1.

========================================

这里我用的centos7.0 x64的系统。

安装前需要安装相对应的epel库，我用的 Centos 7.0的系统。

```
$ wget http://ftp.sjtu.edu.cn/fedora/epel/beta/7/x86_64/epel-release-7-0.2.noarch.rpm

$ sudo rpm -Uvh epel-release-7-0.2.noarch.rpm

$ ls /etc/yum.repos.d/
CentOS-Base.repo CentOS-Debuginfo.repo CentOS-Sources.repo CentOS-Vault.repo epel.repo epel-testing.repo

```

Next, let’s install the `docker-io` package which will install Docker on our host.

```
<span class="pln" style="color: #48484c;">$ sudo yum install docker</span><span class="pun" style="color: #93a1a1;">-</span><span class="pln" style="color: #48484c;">io</span>
```

Now that it’s installed, let’s start the Docker daemon.

```
<span class="pln" style="color: #48484c;">$ sudo service docker start</span>
```

[sudo] password for sysadmin:

Redirecting to /bin/systemctl start docker.service

If we want Docker to start at boot, we should also:

```
<span class="pln" style="color: #48484c;">$ sudo chkconfig docker on</span>
```

Note: Forwarding request to ‘systemctl enable docker.service’.

ln -s ‘/usr/lib/systemd/system/docker.service’ ‘/etc/systemd/system/multi-user.target.wants/docker.servic

Now let’s verify that Docker is working. First we’ll need to get the latest `centos` image.

```
<span class="pln" style="color: #48484c;">$ sudo docker pull centos</span><span class="pun" style="color: #93a1a1;">:</span><span class="pln" style="color: #48484c;">latest</span>
```

Next we’ll make sure that we can see the image by running:

```
<span class="pln" style="color: #48484c;">$ sudo docker images centos</span>
```

This should generate some output similar to:

```
<span class="pln" style="color: #48484c;">$ sudo docker images centos
REPOSITORY      TAG             IMAGE ID          CREATED             VIRTUAL SIZE
centos          latest          </span><span class="lit" style="color: #195f91;">0b443ba03958</span><span class="lit" style="color: #195f91;">2</span><span class="pln" style="color: #48484c;"> hours ago         </span><span class="lit" style="color: #195f91;">297.6</span><span class="pln" style="color: #48484c;"> MB</span>
```

Run a simple bash shell to test the image:

```
<span class="pln" style="color: #48484c;">$ sudo docker run </span><span class="pun" style="color: #93a1a1;">-</span><span class="pln" style="color: #48484c;">i </span><span class="pun" style="color: #93a1a1;">-</span><span class="pln" style="color: #48484c;">t centos </span><span class="pun" style="color: #93a1a1;">/</span><span class="pln" style="color: #48484c;">bin</span><span class="pun" style="color: #93a1a1;">/</span><span class="pln" style="color: #48484c;">bash</span>
```

bash-4.2#

If everything is working properly, you’ll get a simple bash prompt. Type exit to continue.

Done! You can either continue with the [Docker User Guide](http://docs.docker.com/userguide/) or explore and build on the images yourself.

## Issues?

If you have any issues – please report them directly in the [CentOS bug tracker](http://bugs.centos.org/).

官方安装文档： [http://docs.docker.com/installation/centos/](http://docs.docker.com/installation/centos/)

官方docker使用手册： [http://docs.docker.com/userguide/](http://docs.docker.com/userguide/) (中文手册： [http://www.docker.org.cn/book/docker.html](http://www.docker.org.cn/book/docker.html) )