# SELinux 入门

SELinux 是 Linux 安全相关的内容，我平常在使用时，都会选择将其关掉。

发现这篇文章挺简单易懂的：https://zhuanlan.zhihu.com/p/30483108

查看状态：

```bash
$ sestatus -v
$ getenforce
```



临时关闭SELinux：

```bash
$ setenforce 0
```

永久关闭：修改 `/etc/selinux/config` 文件，将 SELINUX=enforcing 改为 SELINUX=disabled ，然后重启机器即可



