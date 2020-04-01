# Ansible 安装

CentOS 安装：

```bash
$ sudo yum install ansible
```

MacOS 安装：

```bash
$ brew install ansible 
```



## argcomplete 安装

命令提示。

CentOS：

```bash
$ sudo yum install python-argcomplete
$ sudo activate-global-python-argcomplete
```

MacOS:

```bash
$ pip install argcomplete
$ sudo activate-global-python-argcomplete
```



## 配置

MacOS 和 CentOS的配置是一样的。

修改 /etc/ansible/hosts ，将需要远程操作的主机写入其中：

```
[fueltank] #可以随意命名分组
fueltank-1
fueltank-2
fueltank-3
```

各个主机之间要配置免密登录。

测试：

```bash
$ ansible all -m ping
$ ansible all -a "/bin/echo hello"
$ # 指定用户
$ ansible all -m ping -u admin
```

