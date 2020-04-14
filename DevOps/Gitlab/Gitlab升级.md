# Gitlab 升级

需要处理 Gitlab 升级的工作。

前同事的部署文档：http://asset.bbdops.com/software/info/e1d4c131-1a30-442c-a15f-c6c84100d79a （仅内网）

目前的版本是 `10.1.2` ，最新版本是 `12.8.5`

官方升级文档：https://docs.gitlab.com/ee/policy/maintenance.html#upgrade-recommendations



## 查看版本号

```bash
$ cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
10.1.2
```



## 升级版本策略

根据官方升级文档：

![image-20200414141409671](../../resource/image-20200414141409671.png)

决定升级路线是 ： `10.1.2`  ->  `10.8.7`  -> `11.11.8` ->  `12.0.12`  -> `12.8.5` 。共四次升级。



## 升级思路及准备

升级过程中，需要关闭服务，需要提前约定好。

需要准备好安装包，可以在 https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/ 查找特定版本号的 rpm 包。

也可以配置好源，然后用 yum 指定版本号 。

关于数据备份，升级一般不会对数据造成影响，纵然万死依旧不能升级成功的话，数据也不会丢失。



## 升级到 10.8.7



