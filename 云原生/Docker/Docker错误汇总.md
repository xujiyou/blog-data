# Docker 错误汇总

MacOS 执行 `docker login` 是报错（采用 ToolBox 方式安装）：

```
Error saving credentials: error storing credentials - err: exec: "docker-credential-desktop": executable file not found in $PATH, out: ``
```

Fix:

```
$ vim ~/.docker/config.json
```

修改为：

```
"credStore": ""
```

