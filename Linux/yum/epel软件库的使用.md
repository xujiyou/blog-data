# epel 软件库的使用

参考：https://www.jianshu.com/p/1882cd3b2295

epel是社区强烈打造的免费开源发行软件包版本库。

EPEL，即Extra Packages for Enterprise Linux的简称，是为企业级Linux提供的一组高质量的额外软件包，包括但不限于Red Hat Enterprise Linux (RHEL), CentOS and Scientific Linux (SL), Oracle Enterprise Linux (OEL)。(关于 : EPEL)

方法一：yum命令安装

```bash
$ sudo yum install epel-release -y
```



## 手动安装

查看 CentOS 版本：

```bash
$ cat /etc/redhat-release
```

测试前可以删除：

```bash
$ sudo yum remove epel-release
```

查看包：

```bash
$ yum repolist
```

下载：

````bash
$ wget https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-12.noarch.rpm
````

安装：

```bash
$ sudo rpm -ivh epel-release-7-12.noarch.rpm
```

再次查看包，发现多了 epel 中的一万多个包：

```bash
$ yum repolist
epel/x86_64 Extra Packages for Enterprise Linux 7 - x86_64                                                               13,229
```

更新数据：

```bash
$ sudo yum clean all && yum makecache
```



 