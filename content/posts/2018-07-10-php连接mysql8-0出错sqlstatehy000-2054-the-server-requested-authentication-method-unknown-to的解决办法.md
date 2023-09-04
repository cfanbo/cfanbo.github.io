---
title: 'PHP连接mysql8.0出错“SQLSTATE[HY000] [2054] The server requested authentication method unknown to”的解决办法'
author: admin
type: post
date: 2018-07-10T10:04:03+00:00
url: /archives/17951
categories:
 - MySQL
tags:
 - mysql

---
错误信息

> SQLSTATE\[HY000\] \[2054\] The server requested authentication method unknown to…

这个错可能是mysql默认使用 `caching_sha2_password` 作为默认的身份验证插件，而不再是 `mysql_native_password`，但是客户端暂时不支持这个插件导致的。 [官方文档说明](https://dev.mysql.com/doc/refman/8.0/en/caching-sha2-pluggable-authentication.html)

> In MySQL 8.0, caching\_sha2\_password is the default authentication plugin rather than mysql\_native\_password. For information about the implications of this change for server operation and compatibility of the server with clients and connectors, see caching\_sha2\_password as the Preferred Authentication Plugin.
>
> 在MySQL 8.0中，caching\_sha2\_password是默认的身份验证插件，而不是mysql\_native\_password。有关此更改对服务器操作的影响以及服务器与客户端和连接器的兼容性的信息，请参阅caching\_sha2\_password作为首选身份验证插件。

**解决方法一：修改MySQL全局配置文件**

编辑 `my.cnf` 文件，更改默认的身份认证插件。

```lang-bash
$ vi /etc/my.cnf

```

在 `[mysqld]` 中添加下边的代码

```lang-bash
default_authentication_plugin=mysql_native_password

```

然后重启mysql

```lang-bash
$ service mysqld restart

```

**解决方法二：修改密码认证方式**

```
ALTER USER 'YOURUSERNAME'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YOURPASSWORD';
```

官方文档： [https://dev.mysql.com/doc/refman/8.0/en/caching-sha2-pluggable-authentication.html](https://dev.mysql.com/doc/refman/8.0/en/caching-sha2-pluggable-authentication.html)