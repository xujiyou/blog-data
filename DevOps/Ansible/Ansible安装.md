# Ansible 安装

CentOS 安装：

```bash
$ sudo yum install ansible
```

MacOS 安装：

```bash
$ pip install --user ansible
```



## argcomplete 安装

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

修改 /etc/ansible/hosts ，将需要远程操作的主机写入其中：

```
fueltank-1
fueltank-2
fueltank-3
```

测试：

```bash
$ ansible all -m ping
$ ansible all -a "/bin/echo hello"
```

