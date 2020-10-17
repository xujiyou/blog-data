# YARN 命令

官方文档：https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YarnCommands.html

YARN 有用户命令和管理命令，命令结构：

```
yarn [SHELL_OPTIONS] COMMAND [GENERIC_OPTIONS] [SUB_COMMAND] [COMMAND_OPTIONS]
```

- SHELL_OPTIONS 通用的选项，对所有子命令都生效
- COMMAND YARN 的各种命令，下面详细介绍。
- COMMAND_OPTIONS 命令选项
- GENERIC_OPTIONS 多个命令都支持的命令选项

敲入 yarn 命令后，会自动给出帮助，如：

```
$ yarn

```



## application 命令

