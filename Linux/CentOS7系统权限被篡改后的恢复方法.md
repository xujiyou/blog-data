# CentOS 7 系统权限被篡改后的恢复方法

## 1. 背景描述

研发人员反馈，由于系统误操作，执行了`chmod -R 777 /usr`，导致普通用户无法执行`sudo`，`root`用户又无法正常登陆成功。

## 2. 排查发现

- 普通用户可以正常`SSH`登陆
- 普通用户无法执行`sudo` ,报错：`sudo: /bin/sudo must be owned by uid 0 and have the setuid bit set`
- console和ssh下，root都无法登录成功。

## 3. 解决方法

适用于Linux系统中默认权限被篡改，如/etc/、/bin、/usr等目录被篡改。

### 3.1 登录单用户模式

- 重启服务器，在选择内核界面使用上下箭头移动
- 选择内核并按`e`进入编辑界面
- 找到 `linux16 vmlinuz…` 那行, 按 `end` 移动到最后, 空一格加入 `init=/bin/bash`，删除`rhgb quiet`

注: OpenStack上的云主机可能添加了`console=tty0 console=ttyS0,115200n8`，要删除它。

- 按 `Ctrl + x` 开机，即可进入单用户模式。
- 将根目录挂载为读写模式：`mount -o remount,rw /`。

### 3.2 恢复默认权限设置

方式一： 利用rpm包的默认权限设置来恢复

***推荐\***

```
for i in `rpm -qa`; do rpm --setperms $i; done
for i in `rpm -qa`; do rpm --setugids $i; done
chmod u+s /usr/bin/sudo
```

方式二：利用acl权限恢复

通过系统自带的getfacl命令来拷贝和还原系统权限，修复的方法如下：

```
# 通过一台权限正常的Linux（最好内核版本和故障服务器相同）来获取系统权限列表
getfacl -R / > systemp.bak
将systemp.bak拷贝至目标机
# 异常服务器中执行恢复权限命令
setfacl --restore=systemp.bak
# 重启系统
reboot
```