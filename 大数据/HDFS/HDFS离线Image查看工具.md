# HDFS 离线 Image 查看工具

脱机图像查看器是一种工具，可将hdfs fsimage文件的内容转储为人类可读的格式，并提供只读的WebHDFS API，以允许脱机分析和检查Hadoop群集的名称空间。该工具能够相对快速地处理非常大的图像文件。该工具处理Hadoop 2.4及更高版本中随附的布局格式。如果要处理较旧的布局格式，则可以使用Hadoop 2.3的Offline Image Viewer或oiv_legacy Command。如果该工具无法处理图像文件，它将干净地退出。 Offline Image Viewer不需要运行Hadoop集群；它的操作完全脱机。

脱机图像查看器提供了几个输出处理器：

- Web是默认的输出处理器。它启动一个HTTP服务器，该服务器公开只读的WebHDFS API。用户可以使用HTTP REST API以交互方式调查名称空间。它不支持安全模式，也不支持HTTPS。
- XML创建fsimage的XML文档，并包括fsimage中的所有信息。该处理器的输出适合使用XML工具进行自动处理和分析。由于XML语法的冗长性，该处理器还将生成最大量的输出。
- FileDistribution是用于分析名称空间映像中文件大小的工具。为了运行该工具，应通过指定maxSize和一个步骤来定义整数范围[0，maxSize]。整数范围分为大小为段的段：[0，s [1]，…，s [n-1]，maxSize]，处理器计算出系统中每个段中有多少文件[s [i] -1]，s [i]）。请注意，大于maxSize的文件始终属于最后一段。默认情况下，输出文件的格式设置为制表符，分隔两列表：Size和NumFiles。其中Size代表该段的开始，numFiles是该映像中属于该段的图像文件数量。通过指定选项-format，将以易于阅读的方式格式化输出文件，而不是“大小”列中显示的字节数。此外，“大小”列将更改为“大小范围”列。
- Delimited（实验性）：生成一个文本文件，其中包含inode和inode-under-construction共同的所有元素，并由分隔符分隔。缺省分隔符为\ t，尽管可以通过-delimiter参数更改。
- DetectCorruption（实验性）：通过有选择地加载图像的一部分并主动搜索不一致来检测图像的潜在损坏。以定界格式输出找到的损坏的摘要。请注意，该检查不是穷举性的，仅在名称空间重构期间捕获丢失的节点。
- ReverseXML（实验性）：这与XML处理器相反。它从XML文件重建fsimage。使用此处理器，可以轻松创建fsimage进行测试，并在损坏时手动编辑fsimage。



## 使用

#### web 处理器

Web处理器启动HTTP服务器，该服务器公开只读WebHDFS API。用户可以通过-addr选项指定要监听的地址（默认为localhost：5978）。

```bash
$ hdfs oiv -i fsimage
```

用户可以通过以下shell命令访问查看器并获取fsimage的信息：

```bash
$ hdfs dfs -ls webhdfs://127.0.0.1:5978/
```

要获取所有文件和目录的信息，只需使用以下命令：

```bash
$ hdfs dfs -ls -R webhdfs://127.0.0.1:5978/
```



























