# FreeIPA 安装

参考：https://www.sysit.cn/blog/post/sysit/FreeIPA-HA%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE

`FreeIPA`建立在著名的开源组件和标准协议之上，是一个集成的安全信息管理解决方案，具有易于管理、安装和配置任务自动化的特点。它整合了`389-ds（LDAP）`、`Kerberos`、`NTP`、`bind`、`apache`、`tomcat`核心软件包，形成一个以`389-ds（LDAP）`为数据存储后端，`Kerberos`为验证前端，`bind`为主机识别，并且具有统一的命令行管理工具及`apache+tomcat`提供的`web`管理界面的集成信息管理系统。



## 安装

在 CentOS8 上安装 FreeIPA

```bash
$ sudo yum -y install @idm:DL1
$ sudo yum -y install freeipa-server
```

配置：

```
sudo ipa-server-install
```

