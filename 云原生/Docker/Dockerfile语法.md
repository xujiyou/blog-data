# Dockerfile 语法

`Dockerfile` 语法格式如下:

```Dockerfile
# Comment
INSTRUCTION arguments
```

\# 开头的是注释，命令使用大写（不强制），参数使用小写。

`Dockerfile`按顺序运行指令。一个`Dockerfile` **必须用`FROM`指令开始**

## 解析器指令

解析器指令是可选的，并且会影响`Dockerfile`处理a 中后续行的方式。解析器指令不会在构建中添加层，也不会显示为构建步骤。解析器指令以形式写为特殊类型的注释`# directive=value`。单个指令只能使用一次。

处理完注释，空行或生成器指令后，Docker不再寻找解析器指令。而是将格式化为解析器指令的任何内容都视为注释，并且不会尝试验证它是否可能是解析器指令。因此，所有解析器指令必须位于 Dockerfile 最顶部。

解析器指令不区分大小写。但是，约定是小写的。约定还应在任何解析器指令之后包含一个空白行。解析器指令不支持行继续符。

由于这些规则，以下示例均无效：

由于是行而无效：

```Dockerfile
# direc \
tive=value
```

由于出现两次而无效：

```Dockerfile
# directive=value1
# directive=value2

FROM ImageName
```

由于出现在构建器指令之后而被视为注释：

```Dockerfile
FROM ImageName
# directive=value
```

由于出现在不是解析器指令的注释之后，因此被视为注释：

```Dockerfile
# About my dockerfile
# directive=value
FROM ImageName
```

由于未被识别，未知指令被视为注释。另外，由于在非语法分析器指令的注释之后出现，因此已知指令被视为注释。

```Dockerfile
# unknowndirective=value
# knowndirective=value
```

解析器指令中允许非换行空格。因此，以下各行都被相同地对待：

```Dockerfile
#directive=value
# directive =value
#	directive= value
# directive = value
#	  dIrEcTiVe=value
```

支持以下解析器指令：

- `syntax`
- `escape`



## 环境变量

用 ENV 声明。使用时，用 `$variable_name`或`${variable_name}` 。

该`${variable_name}`语法还支持一些标准`bash` 修饰符，如下所示：

- `${variable:-word}`表示如果`variable`设置，则结果将是该值。如果`variable`未设置，则结果为`word`。
- `${variable:+word}`指示如果`variable`设置了，则结果将是`word`，否则结果为空字符串。

另外转义字符为 \ ，可以转义上面这些特殊字符。

以下指令的参数中支持环境变量：

- `ADD`
- `COPY`
- `ENV`
- `EXPOSE`
- `FROM`
- `LABEL`
- `STOPSIGNAL`
- `USER`
- `VOLUME`
- `WORKDIR`

## .dockerignore文件

在 Docker Cli 将数据发送给 Docker 守护程序之前，它会在上下文中查找名为 `.dockerignore` 的文件，如果此文件存在，则 Cli 会修改 上下文排除其中与 .dockerignore 中匹配的文件。这有助于避免不必要地将大型文件或敏感文件和目录发送到守护程序，并避免使用`ADD`或 COPY 将它们添加到映像中。

## FROM

```
FROM [--platform=<platform>] <image> [AS <name>]
```

或

```
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
```

或

```
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
```

`FROM`指令初始化一个新的构建阶段，并为后续指令设置 [*基本映像*](https://docs.docker.com/glossary/#base-image)。因此，有效的`Dockerfile`必须从`FROM`指令开始

一个 Dockerfile 可以包含多个 FROM，需要在提交了一次 image 之后才能出现下一个 FROM。as name 就是给下一个 FROM 使用的。

另外 FROM 之前还可以出现 ARG。

该`tag`或`digest`值是可选的。如果您忽略其中任何一个`latest`

`--platform`在`FROM`引用多平台图像的情况下，可选标志可用于指定图像的平台。例如，`linux/amd64`， `linux/arm64`，或`windows/amd64`。默认情况下，使用构建请求的目标平台。可以在此标志的值中使用全局构建参数，例如，[自动平台ARG](https://docs.docker.com/engine/reference/builder/#automatic-platform-args-in-the-global-scope) 允许您将阶段强制到本机构建平台（`--platform=$BUILDPLATFORM`），并使用它来交叉编译到阶段内部的目标平台。

### 了解ARG和FROM之间的交互方式

`FROM`指令支持由`ARG` 第一指令之前的任何指令声明的变量`FROM`。

```Dockerfile
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
CMD  /code/run-app

FROM extras:${CODE_VERSION}
CMD  /code/run-extras
```



## RUN

RUN 有两种形式

- `RUN `（*shell*形式，命令在shell中运行，默认情况下`/bin/sh -c`或`cmd /S /C`Windows 上运行）
- `RUN ["executable", "param1", "param2"]`

`RUN`指令将在当前 Image 顶部的新层中执行所有命令，并提交结果。

```
RUN /bin/bash -c 'source $HOME/.bashrc; \
echo $HOME'
```

它们在一起等效于以下这一行：

```
RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'
```



## CMD

`CMD`指令具有三种形式：

- `CMD ["executable","param1","param2"]`（*exec*形式，这是首选形式）
- `CMD ["param1","param2"]`（作为*ENTRYPOINT的默认参数*）
- `CMD command param1 param2`（ shell 形式）

一个 Dockerfile 中只能有一个 CMD，如果列出多个，则只会有最后一个生效。

CMD 的目的是为容器提供默认值，这些默认值可以包含可执行文件，也可以省略可执行文件，这种情况下，必须指定一条 ENTRYPOINT 指令。

**注意**：请勿`RUN`与混淆`CMD`。`RUN`实际上运行命令并提交结果；`CMD`在生成时不执行任何操作，但指定映像的预期命令。

RUN 与 CMD 的区别如下：

RUN
RUN命令是创建Docker镜像（image）的步骤，RUN命令对Docker容器（ container）造成的改变是会被反映到创建的Docker镜像上的。一个Dockerfile中可以有许多个RUN命令。

CMD
CMD命令是当Docker镜像被启动后Docker容器将会默认执行的命令。一个Dockerfile中只能有一个CMD命令。通过执行docker run \$image $other_command启动镜像可以重载CMD命令。



## LABEL

`LABEL`指令将元数据添加到镜像，LABEL 是一个键值对，

如：

```dockerfile
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```



## EXPOSE

EXPOSE 用于导出端口

默认情况下，`EXPOSE`假定为TCP。您还可以指定UDP：

```Dockerfile
EXPOSE 80/udp
```



## ADD

`ADD`指令从中复制新文件，将它们添加到镜像的文件系统中。

```dockerfile
ADD --chown=55:mygroup files* /somedir/
ADD --chown=bin files* /somedir/
ADD --chown=1 files* /somedir/
ADD --chown=10:11 files* /somedir/
```

除了不能用在 multistage 的场景下，ADD 命令可以完成 COPY 命令的所有功能，并且还可以完成两类超酷的功能：

- 解压压缩文件并把它们添加到镜像中
- 从 url 拷贝文件到镜像中



## COPY

同 ADD



## ENTRYPOINT

最后一条是 `ENTRYPOINT` 指令`Dockerfile`才会生效。

您可以使用*exec*形式的`ENTRYPOINT`设置相当稳定的默认命令和参数，然后使用这两种形式的`CMD`设置其他默认值。

```dockerfile
FROM ubuntu
ENTRYPOINT ["top", "-b"]
CMD ["-c"]
```



## VOLUME



## USER



## WORKDIR



## ARG



## ONBUILD



## STOPSIGNAL



## HEALTHCHECK



## SHELL