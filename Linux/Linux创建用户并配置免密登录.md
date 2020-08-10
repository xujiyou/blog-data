# Linux 创建用户并配置免密登录

创建用户，并且指定家目录。

```bash
$ useradd -d /home/xujiyou -m xujiyou
```

创建用户的时候会默认创建一个和用户名相同的用户组，但是有时有需求需要指定用户组，可以使用 -g 命令来完成用户创建，前提条件是指定的用户组已存在。

```bash
$ groupadd xujiyou
$ useradd -d /home/xujiyou -m xujiyou -g xujiyou
```



## 设置用户密码

```bash
$ passwd xujiyou
```



## 创建系统用户

```bash
$ useradd -r username
```

这条命令也会自动添加用户组。

或者：

```bash
$ sudo groupadd --system prometheus
$ sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```





## 修改用户信息

  有时我们需要修改用户的用户组，家目录等信息，这时候使用useradd命令显然就不合适了，linux系统为我们提供了usermod命令，常用的参数与useradd一样大家可自行尝试。



## 删除用户

 删除用户使用命令userdel，示例：userdel myuser，使用这个命令的话只会删除用户，用户的主目录不会被删除，如果需要删除用户的时候也将用户主目录删除则可以使用-r，示例：userdel -r myuser



## 删除用户组

 删除用户组使用命令groupdel，示例：groupdel mygroup，注意，被删除的用户组不可以是任何用户的主用户组，否则删除失败。用户组删除完成后可以到/etc/group文件中去查看被删除则用户组名称已经不存在了。



## Shell 脚本创建用户

```bash

#!/bin/bash
# 需要创建的用户名，示例：USER_NAME=myuser
USER_NAME=
# 创建用户所属的用户组，示例：USER_GROUP=mygroup
USER_GROUP=
# 用户密码，示例：USER_PASSWD=Cloud12#$
USER_PASSWD=
 
# 校验参数
function check_param()
{
    if [[ ! -n ${USER_NAME} ]] || [[ ! -n ${USER_GROUP} ]] || [[ ! -n ${USER_PASSWD} ]]; then
        echo "ERROR: Please check the param USER_NAME,USER_GROUP,USER_PASSWD can not be null"
        exit 1;
    fi
}
 
# 创建用户
function creat_user()
{
    check_param
	
    #create group
    grep "^${USER_GROUP}" /etc/group &> /dev/null
    if [ $? -ne 0 ]; then
        groupadd ${USER_GROUP}
    fi
    #create user
    id ${USER_NAME} &> /dev/null
    if [ $? -ne 0 ]; then
        useradd -g ${USER_GROUP} ${USER_NAME} -d /home/${USER_NAME}
        echo ${USER_PASSWD}| passwd ${USER_NAME} --stdin
        chage -M 99999 ${USER_NAME}
    fi
}
 
creat_user $*
```



## 免密登录

参考：https://blog.csdn.net/aliaichidantong/article/details/86242540

```bash
$ ssh-keygen -t rsa
```

输入密钥，可以为空，这会在 `~/.ssh` 目录下生成一个 私钥和证书。

#### 创建信任

把生成的公钥文件 `id_rsa.pub` 下载、上传到目标服务器上，也可以直接通过命令`ssh-copy-id -i ~/.ssh/id_rsa.pub admin@fueltank-1` 传过去，不过命令默认端口是22。

录目标服务器，相同的目录/root/.ssh/下查看有没有 `authorized_keys` 文件，没有的话需要创建一个，命令是               `touch authorized_keys` ，创建后授权600 ，把公钥文件 `id_rsa.pub` 追加到 `authorized_keys` 文件中，命令是                        `cat 192.168.1.1.pub >> authorized_keys` ，注意是双箭头>>单箭头会覆盖文件中的内容。

#### 免密登录

将公钥复制完成后，就可以免密登录了。

```bash
$ ssh admin@fueltank-1  # 这种是使用自己的公钥登录目标服务器
```



## 给用户添加sudo权限

使用命令 `visudo` 来编辑 `/etc/sudoers` 文件，添加以下行：

```
xujiyou ALL=(ALL)     NOPASSWD:ALL
```

这时候有可能在使用 sudo 时还需要密码。

查看用户的组：

```
$ id xujiyou
uid=1000(xujiyou) gid=1000(xujiyou) groups=1000(xujiyou),10(wheel)
```

发现后面有个 wheel 。

将这句话：

```
xujiyou ALL=(ALL)     NOPASSWD:ALL
```

放到 `/etc/sudoers`  的最后就好了。



## 限制用户只执行一个命令

