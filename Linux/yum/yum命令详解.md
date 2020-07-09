# yum 命令详解

先来看子命令，再看 options。`yum --help` 查看帮助

子命令还是挺多的，有 29 个。

## check

检查rpmdb中的问题



## check-update

检查可用的软件包更新



## clean

清除缓存数据

```bash
$ sudo yum clean
Error: clean requires an option: headers, packages, metadata, dbcache, plugins, expire-cache, rpmdb, all
$ sudo yum clean all
```



## deplist

列出某软件的依赖：

```bash
$ yum deplist docker
```



## distribution-synchronization

将安装的软件包同步到最新的可用版本

```bash
$ sudo yum distribution-synchronization
```





## downgrade

降级一个包



## erase

从系统中删除一个或多个软件包，remove 是 erase 的一个别名。使用以下命令查看别名：

```bash
$ yum help erase
```





##fs

作用于主机的文件系统数据，主要用于删除主机中的文档/语言。 



## fssnapshot

创建文件系统快照，或列出/删除当前快照。



## groups

显示使用组

```bash
$ sudo yum groups
$ yum grouplist
$ yum group info Haskell
$ yum group list Haskell
```



## help

显示帮助信息，要显示子命令的帮助信息，可以这样：

```bash
$ yum help groups
```





## history

查看使用 yum 的历史，非常好用：

```bash
$ sudo yum history
```



## info

查看软件包信息：

```bash
$ yum info docker
```



## install

安装一个包，很常用



## list

列出一个包，可以使用通配符：

```bash
$ yum list dock*
```



## load-transaction

从文件名加载已保存的软件包



## makecache

生成元数据缓存

```bash
$ sudo yum makecache
```

数据缓存到了 `/var/lib/yum` 



## provides

```bash
$ yum provides docker
```

查看包的提供者



## reinstall

重新安装



## repolist

列出 repo：

```bash
$ sudo yum repolist
```



## repo-pkgs

根据上一步列出的 repo id 进行操作：

```bash
$ sudo yum repo-pkgs epel list
```



## search

搜索包，跟 `yum list` 差不多的功能，不过这个 search 不用手动添加通配符，并且有高亮提示。



## shell

运行一个交互式 shell，来执行 yum 的各种子命令：

```bash
$ sudo yum shell
> list docker
> list docker*
```



## update

更新包，升级所有包同时也升级软件和系统内核



## update-minimal

更新包，但是会匹配当前系统



## updateinfo

查看更新信息



## upgrade

只升级所有包，不升级软件和系统内核



## version

显示物理机及各种repo的版本



## 安装软件指定 repo

```bash
$ sudo yum install nginx --enablerepo=epel
```



## 只下载，不安装

```bash
$ yum -y install google-chrome-stable --downloadonly --downloaddir=./
```







