# CentOS7 重置密码

参考：https://blog.51cto.com/jschu/1706020

在 Linux 启动界面按 e 键，下滑，把 ro 改成 rw init=/sysroot/bin/sh  完成之后按 “Ctrl+x”**

现在已经进入单用户模式了。

依次执行：

```bash
chroot /sysroot/
passwd root
touch /.autorelabel
exit
reboot
```





