# Helm 命令

Helm 是 Kubernetes 的包管理工具。本文基于 Helm 3.

官方文档：https://helm.sh/docs/helm/helm/

---

下面是 Helm 用到的环境变量

| Name             | Description                                                                    |
| ---------------- | ------------------------------------------------------------                   |
| $XDG_CACHE_HOME  | set an alternative location for storing cached files.                          |
| $XDG_CONFIG_HOME | set an alternative location for storing Helm configuration.                    |
| $XDG_DATA_HOME   | set an alternative location for storing Helm data.                             |
| $HELM_DRIVER     | set the backend storage driver. Values are: configmap, secret, memory          |
| $HELM_NO_PLUGINS | disable plugins. Set HELM_NO_PLUGINS=1 to disable plugins.                     |
| $KUBECONFIG      | set an alternative Kubernetes configuration file (default “~/.kube/config”)    |

Helm stores configuration based on the XDG base directory specification, so

- cached files are stored in $XDG_CACHE_HOME/helm
- configuration is stored in $XDG_CONFIG_HOME/helm
- data is stored in $XDG_DATA_HOME/helm

各个操作系统的默认位置：

| Operating System | Cache Path                | Configuration Path            | Data Path               |
| ---------------- | ------------------------- | ----------------------------- | ----------------------- |
| Linux            | $HOME/.cache/helm         | HOME/.config/helm             | $HOME/.local/share/helm |
| macOS            | $HOME/Library/Caches/helm | HOME/Library/Preferences/helm | $HOME/Library/helm      |
| Windows          | %TEMP%\helm               | %APPDATA%\helm                | %APPDATA%\helm          |

通用选项：

```
      --add-dir-header                   If true, adds the file directory to the header
      --alsologtostderr                  log to standard error as well as files
      --debug                            enable verbose output
  -h, --help                             help for helm
      --kube-context string              name of the kubeconfig context to use
      --kubeconfig string                path to the kubeconfig file
      --log-backtrace-at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log-dir string                   If non-empty, write log files in this directory
      --log-file string                  If non-empty, use this log file
      --log-file-max-size uint           Defines the maximum size a log file can grow to. Unit is megabytes. If the value is 0, the maximum file size is unlimited. (default 1800)
      --logtostderr                      log to standard error instead of files (default true)
  -n, --namespace string                 namespace scope for this request
      --registry-config string           path to the registry config file (default "/Users/jiyouxu/Library/Preferences/helm/registry.json")
      --repository-cache string          path to the file containing cached repository indexes (default "/Users/jiyouxu/Library/Caches/helm/repository")
      --repository-config string         path to the file containing repository names and URLs (default "/Users/jiyouxu/Library/Preferences/helm/repositories.yaml")
      --skip-headers                     If true, avoid header prefixes in the log messages
      --skip-log-headers                 If true, avoid headers when opening log files
      --stderrthreshold severity         logs at or above this threshold go to stderr (default 2)
  -v, --v Level                          number for the log level verbosity
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging
```

子命令：

```
  completion  Generate autocompletions script for the specified shell (bash or zsh)
  create      create a new chart with the given name
  dependency  manage a chart's dependencies
  env         Helm client environment information
  get         download extended information of a named release
  help        Help about any command
  history     fetch release history
  install     install a chart
  lint        examines a chart for possible issues
  list        list releases
  package     package a chart directory into a chart archive
  plugin      install, list, or uninstall Helm plugins
  pull        download a chart from a repository and (optionally) unpack it in local directory
  repo        add, list, remove, update, and index chart repositories
  rollback    roll back a release to a previous revision
  search      search for a keyword in charts
  show        show information of a chart
  status      displays the status of the named release
  template    locally render templates
  test        run tests for a release
  uninstall   uninstall a release
  upgrade     upgrade a release
  verify      verify that a chart at the given path has been signed and is valid
  version     print the client version information
```

---

## Helm Completion

用于生成 Helm 命令的自动补全脚本（bash 或 zsh）。

```bash
$ helm completion bash
```

也可以使用 source 和 写入配置文件

```bash
$ source <(helm completion bash)
$ echo "source <(helm completion bash)" >> ~/.bashrc
```

---

## Helm Create

创建一个新的 Crate，比如：

```bash
$ helm create xujiyou
```

---

## Helm Dependency

用于管理 Chart 的依赖
