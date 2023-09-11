---
title: gitlab安装-设置1-修改仓库(repositories)的位置
author: admin
type: post
date: 2016-06-07T01:03:10+00:00
url: /archives/17070
categories:
 - 服务器
tags:
 - gitlab

---
安装好gitlab后，要将仓库(repositories)放在一个大硬盘上,在ubuntu服务器上安装的默认位置为 /var/opt/gitlab/git-data/ 目录，需要修改仓库对应的目录

**操作步骤：**

1：新建新仓库目录

```
mkdir -p /mnt/application/gitlab/git-data
```

2：修改配置文件 sudo vi /etc/gitlab/gitlab.rb
搜索：git_data_dir 修改成：git_data_dir “新目录”
如：

```
git_data_dir "/mnt/application/gitlab/git-data"
```

保存
3：重新生成gitlab

```
sudo gitlab-ctl reconfigure
```

生成不报错，而且在新建仓库目录可以看到从下的目录，即修改成功。