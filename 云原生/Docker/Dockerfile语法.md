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

