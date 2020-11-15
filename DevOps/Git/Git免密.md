# Git 免密

Git 每次 pull 和 push 时都需要输入用户名密码太繁琐，可以通过设置避免这个不必要的操作。



```bash
$ vim ~/.git-credentials
```

写入以下内容：

```
https://xujiyou%40testing.com:123456@git.testing.com
```

然后执行下面的命令：

```bash
 $ git config --global credential.helper store
```

打开~/.gitconfig文件，会发现多了一项:

```
[credential]
			helper = store
```

然后第一次执行 git 的时候需要密码，之后就不需要了！