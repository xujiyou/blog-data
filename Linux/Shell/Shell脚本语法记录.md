# Shell 脚本语法记录

使用 IDEA，装上插件来写 Shell 脚本体验更好。写代码的地方尽量都要使用智能 IDE。

为了省去把脚本来拷贝到服务器的步骤，可以考虑使用 ssh 命令来远程执行命令：

```bash
$ ssh -T admin@fueltank-1 < /Users/jiyouxu/IdeaProjects/my-grpc/test.sh
```

或者

```bash
$ ssh admin@fueltank-1 'bash  -s' < /Users/jiyouxu/IdeaProjects/my-grpc/test.sh
```

前提是配置了免密登录，这样就可以实现远程执行命令了，狂拽酷炫吊炸天。

## 解释器

```bash
# 以下两种方式都可以指定 shell 解释器为 bash，第二种方式更好
#!/bin/bash
#!/usr/bin/env bash
```



## 加减法

```bash
$((i-1))
```

