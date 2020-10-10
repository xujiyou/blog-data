# HDFS 命令参考

官方文档：https://hadoop.apache.org/docs/r3.3.0/hadoop-project-dist/hadoop-hdfs/HDFSCommands.html

所有HDFS命令均由 `bin/hdfs` 脚本调用。运行不带任何参数的hdfs脚本将打印所有命令的描述。

使用方法：hdfs [SHELL_OPTIONS] COMMAND [GENERIC_OPTIONS] [COMMAND_OPTIONS]

Hadoop有一个选项解析框架，该框架使用解析通用选项以及运行类：

| COMMAND_OPTIONS         | Description                                                  |
| :---------------------- | :----------------------------------------------------------- |
| SHELL_OPTIONS           | The common set of shell options. These are documented on the [Commands Manual](https://hadoop.apache.org/docs/r3.3.0/hadoop-project-dist/hadoop-common/CommandsManual.html#Shell_Options) page. |
| GENERIC_OPTIONS         | The common set of options supported by multiple commands. See the Hadoop [Commands Manual](https://hadoop.apache.org/docs/r3.3.0/hadoop-project-dist/hadoop-common/CommandsManual.html#Generic_Options) for more information. |
| COMMAND COMMAND_OPTIONS | Various commands with their options are described in the following sections. The commands have been grouped into [User Commands](https://hadoop.apache.org/docs/r3.3.0/hadoop-project-dist/hadoop-hdfs/HDFSCommands.html#User_Commands) and [Administration Commands](https://hadoop.apache.org/docs/r3.3.0/hadoop-project-dist/hadoop-hdfs/HDFSCommands.html#Administration_Commands). |



## User Commands

对hadoop集群的用户有用的命令。

#### `classpath`

用法： `hdfs classpath [--glob |--jar <path> |-h |--help]`

| COMMAND_OPTION | Description                                     |
| :------------- | :---------------------------------------------- |
| `--glob`       | expand wildcards                                |
| `--jar` *path* | write classpath as manifest in jar named *path* |
| `-h`, `--help` | print help                                      |

打印获取Hadoop jar和所需库所需的类路径。如果不带参数调用，则打印命令脚本设置的类路径，该类路径可能在类路径条目中包含通配符。其他选项在通配符扩展后打印类路径，或将类路径写入jar文件的清单中。后者在无法使用通配符且扩展的类路径超过支持的最大命令行长度的环境中很有用。



#### `dfs`

用法：`hdfs dfs [COMMAND [COMMAND_OPTIONS]]`

在Hadoop支持的文件系统上运行文件系统命令。COMMAND_OPTIONS 参见 [File System Shell Guide](https://hadoop.apache.org/docs/r3.3.0/hadoop-project-dist/hadoop-common/FileSystemShell.html).



#### `envvars`

用法：`hdfs envvars`

显示Hadoop环境变量。



#### `fetchdt`

用法：`hdfs fetchdt <opts> <token_file_path>`

| COMMAND_OPTION          | Description                                                  |
| :---------------------- | :----------------------------------------------------------- |
| `--webservice` *NN_Url* | Url to contact NN on (starts with http or https)             |
| `--renewer` *name*      | Name of the delegation token renewer                         |
| `--cancel`              | Cancel the delegation token                                  |
| `--renew`               | Renew the delegation token. Delegation token must have been fetched using the –renewer *name* option. |
| `--print`               | Print the delegation token                                   |
| *token_file_path*       | File path to store the token into.                           |

从NameNode获取委托令牌



#### `fsck`

用法：

```
   hdfs fsck <path>
          [-list-corruptfileblocks |
          [-move | -delete | -openforwrite]
          [-files [-blocks [-locations | -racks | -replicaDetails | -upgradedomains]]]
          [-includeSnapshots] [-showprogress]
          [-storagepolicies] [-maintenance]
          [-blockId <blk_Id>] [-replicate]
```

| COMMAND_OPTION                       | Description                                                  |
| :----------------------------------- | :----------------------------------------------------------- |
| *path*                               | Start checking from this path.                               |
| `-delete`                            | Delete corrupted files.                                      |
| `-files`                             | Print out files being checked.                               |
| `-files` `-blocks`                   | Print out the block report                                   |
| `-files` `-blocks` `-locations`      | Print out locations for every block.                         |
| `-files` `-blocks` `-racks`          | Print out network topology for data-node locations.          |
| `-files` `-blocks` `-replicaDetails` | Print out each replica details.                              |
| `-files` `-blocks` `-upgradedomains` | Print out upgrade domains for every block.                   |
| `-includeSnapshots`                  | Include snapshot data if the given path indicates a snapshottable directory or there are snapshottable directories under it. |
| `-list-corruptfileblocks`            | Print out list of missing blocks and files they belong to.   |
| `-move`                              | Move corrupted files to /lost+found.                         |
| `-openforwrite`                      | Print out files opened for write.                            |
| `-showprogress`                      | Deprecated. A dot is print every 100 files processed with or without this switch. |
| `-storagepolicies`                   | Print out storage policy summary for the blocks.             |
| `-maintenance`                       | Print out maintenance state node details.                    |
| `-blockId`                           | Print out information about the block.                       |
| `-replicate`                         | Initiate replication work to make mis-replicated blocks satisfy block placement policy. |

运行HDFS文件系统检查实用程序



#### `getconf`

用法：

```
   hdfs getconf -namenodes
   hdfs getconf -secondaryNameNodes
   hdfs getconf -backupNodes
   hdfs getconf -journalNodes
   hdfs getconf -includeFile
   hdfs getconf -excludeFile
   hdfs getconf -nnRpcAddresses
   hdfs getconf -confKey [key]
```

| COMMAND_OPTION        | Description                                                  |
| :-------------------- | :----------------------------------------------------------- |
| `-namenodes`          | gets list of namenodes in the cluster.                       |
| `-secondaryNameNodes` | gets list of secondary namenodes in the cluster.             |
| `-backupNodes`        | gets list of backup nodes in the cluster.                    |
| `-journalNodes`       | gets list of journal nodes in the cluster.                   |
| `-includeFile`        | gets the include file path that defines the datanodes that can join the cluster. |
| `-excludeFile`        | gets the exclude file path that defines the datanodes that need to decommissioned. |
| `-nnRpcAddresses`     | gets the namenode rpc addresses                              |
| `-confKey` [key]      | gets a specific key from the configuration                   |

从配置目录获取配置信息，进行后处理。



#### `groups`

用法：`hdfs groups [username ...]`

返回给定一个或多个用户名的组信息。









