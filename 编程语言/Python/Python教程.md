# Python 教程

Python官网：https://www.python.org/

官方中文文档：https://docs.python.org/zh-cn/3/



快速启动一个 HTTP 服务，代理当前目录下的文件：

```bash
$ nohup python -m SimpleHTTPServer 8001 &
```

这个 http 服务是单线程的！

如果是 python3：

```bash
$  nohup python3 -m http.server 8001 &
```

