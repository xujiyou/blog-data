# 添加 ES 节点

在 ES 集群中加入一个新节点后，短时间内会出现问题，这是因为这段时间 ES 集群在平衡数据。

查看 ES 的健康状态：

```bash
curl -XGET 'http://10.28.92.11:9200/_cluster/health?pretty=true'
{
  "cluster_name" : "security",
  "status" : "red",
  "timed_out" : false,
  "number_of_nodes" : 2,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 48,
  "active_shards" : 63,
  "relocating_shards" : 0,
  "initializing_shards" : 6,
  "unassigned_shards" : 80,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 6,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 7064,
  "active_shards_percent_as_number" : 42.281879194630875
}
```

这里因为我刚加入了一个 ES 节点，所以这里是 red 的。

active_shards_percent_as_number 这里显示了分片正常的比例，因为刚刚加入一个 ES 节点，所以这个值是逐渐变大的，因为这时在往新节点传递数据。这里如果是 100 表示一切 OK。

查看分片状态：

```bash
curl http://10.28.93.11:9200/_cat/shards
```

这里显示的分片状态包括 master 分片和副本分片。

分片正常的状态应该是 STARTED。

之前，当我的 ES 集群中只有一个节点时，会发现有一半的索引都处于 UNASSIGNED 状态，这是因为只有一个节点，索引的一个副本无处安放。再加入一个ES节点就可以了。

还有一个需要注意的问题，再添加一个 ES 节点后，Kibana 会暂时不能使用，过一段时间重启 Kibana 就可以了。

查看磁盘占用，即总数据量：

```bash
curl http://10.28.93.11:9200/_cat/allocation?v
shards disk.indices disk.used disk.avail disk.total disk.percent host        ip          node
    75        2.3gb    87.1gb     11.7tb     11.8tb            0 10.28.92.11 10.28.92.11 security-1
    74        2.3gb    60.8gb      8.1tb      8.1tb            0 10.28.93.11 10.28.93.11 IDS-transfer
```

allocation 是分配的意思，不知道字段的意思，可以使用下面的方法查询：

```bash
curl http://10.28.93.11:9200/_cat/allocation?help
shards       | s              | number of shards on node      
disk.indices | di,diskIndices | disk used by ES indices       
disk.used    | du,diskUsed    | disk used (total, not just ES)
disk.avail   | da,diskAvail   | disk available                
disk.total   | dt,diskTotal   | total capacity of all volumes 
disk.percent | dp,diskPercent | percent disk used             
host         | h              | host of node                  
ip           |                | ip of node                    
node         | n              | name of node 
```

disk.indices 是 ES 实际使用的磁盘空间，disk.used 是当前机器使用的总磁盘容量。







