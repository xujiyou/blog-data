# HBase 与 MapReduce

官方文档：https://hbase.apache.org/book.html#mapreduce

本章讨论在HBase中对数据使用MapReduce时需要采取的特定配置步骤。此外，它还讨论了HBase和MapReduce作业之间的其他交互和问题。最后，讨论了Cascading，这是MapReduce的替代API。

> `mapred` *and* `mapreduce`
>
> 与MapReduce本身一样，HBase中有两个mapreduce软件包：org.apache.hadoop.hbase.mapred和org.apache.hadoop.hbase.mapreduce。前者执行旧式API，后者执行新模式。后者具有更多功能，尽管您通常可以在较早的软件包中找到等效的软件包。选择与MapReduce部署一起提供的软件包。如有疑问或从头开始，请选择org.apache.hadoop.hbase.mapreduce。在下面的注释中，我们引用o.a.h.h.mapreduce，但如果您使用的是o.a.h.h.mapred，则将其替换。



