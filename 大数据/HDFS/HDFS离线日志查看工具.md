# HDFS 离线日志查看工具

官方文档：https://hadoop.apache.org/docs/r3.3.0/hadoop-project-dist/hadoop-hdfs/HdfsEditsViewer.html

离线Edits Viewer是用于解析Edits日志文件的工具。当前的处理器对于在不同格式之间进行转换最为有用，包括比原始二进制格式更易于阅读和编辑的XML。

该工具可以解析-18（大致为Hadoop 0.19）及更高版本的编辑格式。该工具仅对文件运行，不需要运行Hadoop集群。

支持的输入格式：

1. **binary**：Hadoop内部使用的本机二进制格式
2. **xml**：由XML处理器产生的XML格式，如果文件名具有`.xml`（不区分大小写）扩展名，则使用

注意：不允许XML /二进制格式输入文件由同一类型的处理器处理。

脱机编辑查看器提供了几个输出处理器（除非另有说明，否则可以将处理器的输出转换回原始编辑文件）：

1. **binary**：Hadoop内部使用的本机二进制格式
2. **xml**：XML格式
3. **stats**：打印出统计信息，无法将其转换回Edits文件



## 用法



#### XML 解析

XML处理器可以创建一个包含编辑日志信息的XML文件。用户可以通过-i和-o命令行指定输入和输出文件。

```bash
$ hdfs oev -p xml -i /var/lib/hadoop/hdfs/namenode/current/edits_0000000000004775213-0000000000004775283 -o edits.xml
```

XML处理器是“离线编辑查看器”中的默认处理器，用户也可以使用以下命令：

```bash
$ hdfs oev -i /var/lib/hadoop/hdfs/namenode/current/edits_0000000000004775213-0000000000004775283 -o edits.xml
```



## 二进制解析

二进制处理器与XML处理器相反。用户可以通过-i和-o命令行指定输入XML文件和输出文件。

```bash
$ hdfs oev -p binary -i edits.xml -o edits
```



## 统计处理器

统计处理器用于汇总编辑日志文件中包含的操作码计数。用户可以通过-p选项指定此处理器。

```bash
$ hdfs oev -p stats -i edits -o edits.stats
```



## 案例研究：Hadoop集群恢复

如果hadoop群集出现问题，并且编辑文件已损坏，则可以保存至少一部分正确的编辑文件。可以通过将二进制编辑转换为XML，手动进行编辑，然后再将其转换回二进制来完成。最常见的问题是edits文件缺少结束记录（具有opCode -1的记录）。该工具应识别出该错误，并且应正确关闭XML格式。

如果XML文件中没有结束记录，则可以在最后一个正确的记录之后添加一个。 opCode -1记录之后的所有内容都将被忽略。

关闭记录的示例（使用opCode -1）：

```xml
  <RECORD>
    <OPCODE>-1</OPCODE>
    <DATA>
    </DATA>
  </RECORD>
```





















