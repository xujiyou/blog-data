# xargs 命令

大多数命令都不接受标准输入作为参数，只能直接在命令行输入参数，这导致无法用管道命令传递参数。举例来说，`echo`命令就不接受管道传参。

```bash
$ echo "hello world" | echo
```

`xargs`命令的作用，是将标准输入转为命令行参数。

```bash
$ echo "hello world" | xargs echo
hello world
```

