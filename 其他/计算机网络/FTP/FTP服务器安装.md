---
title: FTP服务器安装
date: 2020-07-20 11:21:52
tags:
---

安装：

```bash
$ yum -y install vsftpd
```

安装完之后在/etc/vsftpd/路径下会存在三个配置文件。

vsftpd.conf: 主配置文件

ftpusers: 指定哪些用户不能访问FTP服务器,这里的用户包括root在内的一些重要用户。

user_list: 指定的用户是否可以访问ftp服务器，通过vsftpd.conf文件中的userlist_deny的配置来决定配置中的用户是否可以访问，userlist_enable=YES ，userlist_deny=YES ，userlist_file=/etc/vsftpd/user_list 这三个配置允许文件中的用户访问FTP。

在 `/etc/vsftpd/user_list` 中添加可以访问的用户名。 

启动：

```bash
$ systemctl enable vsftpd.service
$ systemctl start vsftpd.service
```

SELinux 要关闭，防火墙要打开 21 端口。

FileZilla 要用主动模式。

