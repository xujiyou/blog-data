# Hive 命令

Hive 官方文档：https://cwiki.apache.org/confluence/display/Hive/LanguageManual

 

## 命令

官方文档：https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Commands

命令是非SQL语句，例如设置属性或添加资源。它们可以在HiveQL脚本中使用，也可以直接在CLI或Beeline中使用。

| Command                                                      | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
|                                                              |                                                              |
| quit exit                                                    | Use quit or exit to leave the interactive shell.             |
| reset                                                        | 将配置重置为默认值（从Hive 0.10开始：参见HIVE-3202）。在hive命令行中使用set命令或-hiveconf参数设置的任何配置参数都将重置为默认值。请注意，这不适用于在set命令中使用键名的前缀“ hiveconf：”设置的配置参数（出于历史原因）。 |
| set <key>=<value>                                            | 设置特定配置变量（键）的值。 注意：如果您拼写错误的变量名，则CLI不会显示错误。 |
| set                                                          | 打印由用户或Hive覆盖的配置变量列表。                         |
| set -v                                                       | 打印所有Hadoop和Hive配置变量。                               |
| add FILE[S] <filepath> <filepath>* add JAR[S] <filepath> <filepath>* add ARCHIVE[S] <filepath> <filepath>* | 将一个或多个文件，jar或存档添加到分布式缓存中的资源列表。有关更多信息，请参见[Hive Resources](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-HiveResources)。 |
| add FILE[S] <ivyurl> <ivyurl>*  add JAR[S] <ivyurl> <ivyurl>*  add ARCHIVE[S]<ivyurl> <ivyurl>* | 从Hive 1.2.0开始，使用格式为 ivy:// group:module:version?query_string 的Ivy URL将一个或多个文件，jar或存档添加到分布式缓存中的资源列表中。有关更多信息，请参见[Hive Resources](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-HiveResources)。 |
| list FILE[S] list JAR[S] list ARCHIVE[S]                     | 列出已经添加到分布式缓存的资源。                             |
| list FILE[S] <filepath>* list JAR[S] <filepath>* list ARCHIVE[S] <filepath>* | 检查给定资源是否已经添加到分布式缓存中                       |
| delete FILE[S] <filepath>* delete JAR[S] <filepath>* delete ARCHIVE[S] <filepath>* | 从分布式缓存中删除资源。                                     |
| delete FILE[S] <ivyurl> <ivyurl>*  delete JAR[S] <ivyurl> <ivyurl>*  delete ARCHIVE[S] <ivyurl> <ivyurl>* | 从Hive 1.2.0开始，从分布式缓存中删除使用<ivyurl>添加的资源。 |
| ! <command>                                                  | 从Hive Shell执行Shell命令。                                  |
| dfs <dfs command>                                            | 从Hive Shell执行dfs命令。                                    |
| <query string>                                               | 执行Hive查询并将结果打印到标准输出。                         |
| source FILE <filepath>                                       | 在CLI内执行脚本文件。                                        |
| compile `<groovy string>` AS GROOVY NAMED <name>             | 允许内联Groovy代码被编译并用作UDF（从Hive 0.13.0开始）。 For a usage example, see [Nov. 2013 Hive Contributors Meetup Presentations – Using Dynamic Compilation with Hive](https://cwiki.apache.org/confluence/download/attachments/27362054/HiveContrib-Nov13-groovy_plus_hive.pptx?version=1&modificationDate=1385171856000&api=v2). |

例子：

```
  hive> set mapred.reduce.tasks=32;
  hive> set;
  hive> select a.* from tab1;
  hive> !ls;
  hive> dfs -ls;
```

