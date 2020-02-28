# [Fluentd](https://www.fluentd.org/) 安装

[Fluentd](https://www.fluentd.org/) 是一个日志收集系统，下面首先从 MacOS 上进行测试。

## MacOS 安装

从 https://td-agent-package-browser.herokuapp.com/3/macosx 这个地方下载 dmg 文件，然后双击安装即可。

然后命令行执行：

```bash
$ sudo launchctl load /Library/LaunchDaemons/td-agent.plist
$ less /var/log/td-agent/td-agent.log
2020-02-28 13:33:07 +0800 [info]: parsing config file is succeeded path="/etc/td-agent/td-agent.conf"
2020-02-28 13:33:07 +0800 [warn]: [output_td] secondary type should be same with primary one primary="Fluent::Plugin::TreasureDataLogOutput" secondary="Fluent::Plugin::FileOutput"
2020-02-28 13:33:08 +0800 [info]: using configuration file: <ROOT>
```

从日志可以看到，配置文件在 /etc/td-agent/td-agent.conf

启动之后，会监听 8888 端口，这个端口是写在上面的这个配置文件中的！

传点数据给 Fluentd：

```bash
$ curl -X POST -d 'json={"json":"message"}' http://localhost:8888/debug.test
$ tail -n 1 /var/log/td-agent/td-agent.log
2020-02-28 13:42:34.263609000 +0800 debug.test: {"json":"message"}
```

LInux 的安装基本也是这个套路。