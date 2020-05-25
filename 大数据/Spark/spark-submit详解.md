# spark-submit 详解

spark-submit 官方教程：https://spark.apache.org/docs/latest/submitting-applications.html

可以使用 `spark-submit --help` 查看完整选项。

下面分别介绍一下这些参数。



### --master MASTER_URL

MASTER_URL 可以是 spark://host:port, mesos://host:port, yarn, k8s://host:port, or local (Default: local[*]).



### --deploy-mode DEPLOY_MODE

可选值为 client 和 cluster，默认是 client，以 client 模式运行，就会把日志打印到当前命令窗口，就像在编辑器中调试一样。如果 cluster，spark 就会将计算调度上某个 worker 之上。

当这个值是 cluster 时，spark 默认会将日志放到 worker 节点下的 spark 安装目录的 work 目录下，这个目录下有这个 worker 运行的所有任务的文件夹，

才开始所有的任务都是以 driver 开头的，只有 FINISHED 状态的任务才会转换为 app 开头的文件夹。



### --class CLASS_NAME

对于 Java 或 Scala 程序来说，通过这个参数来指定主类。。



### --name NAME

通过这个参数来为任务指定一个名字。不过程序中的写好的 name 会覆盖这个参数。



### --jars JARS

以逗号分隔的jar列表，包括在驱动程序和执行程序的类路径中。



### --packages 

以逗号分隔的 jar 包名字列表，格式是 groupId:artifactId:version 



### --exclude-packages

与 --packages  的作用相反。



### --repositories

指定 jar 仓库。



### --py-files PY_FILES

以逗号分隔的放在 PYTHONPATH 中的 .zip, .egg, or .py 文件列表。



### --files FILES

可以通过 SparkFiles.get(fileName) 访问的文件列表，以逗号分隔。



### --conf PROP=VALUE

任意的 Spark 配置属性



### --properties-file FILE

指定配置文件，默认是 `conf/spark-defaults.conf` 。



### --driver-memory MEM

默认是 1024M



### --driver-java-options

额外的工作节点的 java 选项。



### --driver-library-path

额外的库。



### --driver-class-path

额外的类路径



### --executor-memory MEM

执行时使用的内存，默认 1G。



### --proxy-user NAME

使用哪个用户跑，在使用了 --principal / --keytab 的情况下不起作用。



### --version

查看版本



### --help, -h

查看帮助信息



### --driver-cores NUM

可以使用的 CPU 核心数，默认是 1，只有在 cluster 模式下有效



### --total-executor-cores NUM

总共的核心数量，只用于 standalone 模式下。

