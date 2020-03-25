# systemctl 命令详解

systemctl 有很多命令，不仅仅是常用的启动停止服务。下面结合 `systemctl -h` 和 实际操作来完整的学习一下 systemctl。

## option

- **-h --help** 展示帮助信息
- **--version** 展示包版本
- **--system** 连接到系统管理
- **-H --host=[USER@]HOST** 操作远程主机
- **-M --machine=CONTAINER** 操作本地容器
- **-t --type=TYPE** 列出特定类型的 Unit
- **--state=STATE** 列出处于 LOAD or SUB or ACTIVE 状态的 unit
- **-p --property=NAME** 仅按此名称显示属性
- **-a --all ** 显示所有加载的单元/属性，包括空单元/属性。要列出系统上安装的所有单元，请改用 `list-unit-files`
- **-l --full** 不要省略输出中的单位名称
- **-r --recursive** 显示主机和本地容器的 unit 列表
- **--reverse** 反转 `list-dependencies`的依赖
- **--job-mode=MODE** 指定在排队新作业时如何处理已排队的作业
- **--show-types** 显示套接字时，显式显示其类型
- **-i --ignore-inhibitors** 关闭时，忽略抑制器
- **--kill-who=WHO** 给谁发送 signal 信号
- **--signal=SIGNAL** 发送哪种信号
- **--now** 启动或停止设备以及启用或禁用它
- **-q --quiet** 抑制输出
- **--no-block** 不用等到操作完成
- **--no-wall** 在 `halt/power-off/reboot` 之前不要发送留言
- **--no-reload** 不要重新加载守护程序，在启用/禁用单元文件后
- **--no-legend** 不要打印多余的信息（列标题和提示）
- **--no-pager** 不要将输出通过管道传送到控制台
- **--no-ask-password** 不要询问系统密码
- **--global** 全局启用/禁用单位文件
- **--runtime** 仅暂时启用单位文件，直到下次重新启动
- **-f --force** 启用单位文件时，覆盖现有的符号链接，关闭时，立即执行操作
- **--preset-mode=** 仅支持 enable, only disable, or all presets
- **--root=PATH** 在指定的根目录中启用单位文件
- **-n --lines=INTEGER** 要显示的 journal 的日志行数
- **-o --output=STRING** 改变 journal 的输出模式(short, short-iso,short-precise, short-monotonic, verbose,export, json, json-pretty, json-sse, cat)
- **--plain** 打印 unit 依赖，以列表的形式代替树的形式



## unit 命令

Systemd 可以管理所有系统资源。不同的资源统称为 Unit（单位）。

Unit 一共分成12种。

> - Service unit：系统服务
> - Target unit：多个 Unit 构成的一个组
> - Device Unit：硬件设备
> - Mount Unit：文件系统的挂载点
> - Automount Unit：自动挂载点
> - Path Unit：文件或路径
> - Scope Unit：不是由 Systemd 启动的外部进程
> - Slice Unit：进程组
> - Snapshot Unit：Systemd 快照，可以切回某个快照
> - Socket Unit：进程间通信的 socket
> - Swap Unit：swap 文件
> - Timer Unit：定时器

- **list-units [PATTERN...]** 列出已加载的 unit 列表
- **list-sockets [PATTERN...]** 列出使用到的 sockets ，按地址排序
- **list-timers [PATTERN...]** 列出加载的计时器（按下一个时间顺序排列）
- **start NAME...** 启动一个或 多个 unit
- **stop NAME...** 停止一个或多个 unit
- **reload NAME...** 重新加载一个或多个 unit
- **restart NAME...** 启动或重启一个或多个 unit
- **try-restart NAME...** 如果在 active 状态，就重启一个或多个 unit
- **reload-or-restart NAME...** 如果可能的话重新加载一个或多个 unit，否则就启动或重启
- **reload-or-try-restart NAME...** 如果可能的话重新加载一个或多个 unit，否则就重启
- **isolate NAME** 启动一个 unit，同时停止所有的其他 unit
- **kill NAME...** 发送 signal 信息给 unit 中的进程
- **is-active PATTERN... ** 检查 unit 是否处于 active 状态
- **is-failed PATTERN... ** 检查 unit 是否处于 failed 状态
- **status [PATTERN...|PID...]** 检查 一个或多个 unit 的状态
- **show [PATTERN...|JOB...]** 展示 unit 的属性
- **cat PATTERN...** 显示一个或多个 unit 的文件和插件
- **set-property NAME ASSIGNMENT...** 为 unit 设置一个或多个属性
- **help PATTERN...|PID...** 展示 一个或多个 unit 的帮助信息
- **reset-failed [PATTERN...]** 全部重置失败状态
- **list-dependencies [NAME]** 列出依赖



## unit 文件命令

每一个 Unit 都有一个配置文件，告诉 Systemd 怎么启动这个 Unit 。

Systemd 默认从目录`/etc/systemd/system/`读取配置文件。但是，里面存放的大部分文件都是符号链接，指向目录`/usr/lib/systemd/system/`，真正的配置文件存放在那个目录。

`systemctl enable`命令用于在上面两个目录之间，建立符号链接关系。

> ```bash
> $ sudo systemctl enable clamd@scan.service
> # 等同于
> $ sudo ln -s '/usr/lib/systemd/system/clamd@scan.service' '/etc/systemd/system/multi-user.target.wants/clamd@scan.service'
> ```

如果配置文件里面设置了开机启动，`systemctl enable`命令相当于激活开机启动。

- **list-unit-files [PATTERN...]** 列出安装的 unit 文件
- **enable NAME... ** 
- **disable NAME...**
- **reenable NAME..**
- **preset NAME...** 预计预设配置，enable 或 disable
- **preset-all** 
- **is-enabled NAME...** 检查 unit 文件是否 enable  了
- **mask NAME...** 注销服务
- **unmask NAME...** 取消注销服务
- **link PATH...** 将不在标准单元目录中的单元文件(通过软链接)连接到标准单元目录中去
- **add-wants TARGET NAME...**  添加 `Wants` 依赖给一组 unit
- **add-requires TARGET NAME...** 添加 `Requires` 依赖给一组 unit
- **edit NAME...** 编辑 unit
- **get-default** 获取默认目标的名称
- **set-default NAME** 设置默认 target

systemctl mask <service> 是注销服务的意思。

注销服务意味着：

1. 该服务在系统重启的时候不会启动
2. 该服务无法进行做systemctl start/stop操作
3. 该服务无法进行systemctl enable/disable操作



##Machine 命令

- **list-machines [PATTERN...]** 列出物理机



## Job 命令

- **list-jobs** 列出所有的 job
- **cancel [JOB...]** 取消所有、一个或多个 job



## Snapshot 命令

- **snapshot [NAME]** 创建一个快照
- **delete NAME...** 删除一个或多个快照



## Environment 命令

- **show-environment** 打印环境变量
- **set-environment NAME=VALUE...** 设置一个或多个环境变量
- **unset-environment NAME...** 取消设置环境变量
- **import-environment [NAME...]** 导入环境变量



## Manager Lifecycle  命令

- **daemon-reload** 重新加载 systemd 的管理配置
- **daemon-reexec** 重新执行 systemd 管理



## System 命令

- **is-system-running** 检查系统是否完全运行
- **default** 进入系统的 default 模式
- **rescue** 进入系统的 rescue(拯救) 模式
- **emergency** 进入系统的 emergency(紧急) 模式
- **halt** 关闭并停止系统
- **poweroff** 关闭并关闭系统电源
- **reboot [ARG]** 重启系统
- **kexec** 通过 kexec 来重启系统
- **exit** 请求用户实例退出
- **switch-root ROOT [INIT]** 更改为其他根文件系统
- **suspend** 挂起系统
- **hibernate** 休眠系统
- **hybrid-sleep** 休眠并挂起系统

