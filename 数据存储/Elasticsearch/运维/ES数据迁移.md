# ES 数据迁移

公司切换测试集群，其中的 ES 也要切换，数据也需要迁移，下面使用 Logstash 来迁移数据。

参考：https://www.elastic.co/guide/en/elasticsearch/reference/6.8/modules-snapshots.html



## 迁移数据

Logstash 配置如下：

```
input {
    elasticsearch {
        hosts => ["http://lb.testing.bbdops.com:9200"]
        index => "*"
        docinfo => true
    }
}

output {
    elasticsearch {
        hosts => ["http://lb.testing2.bbdops.com:9200"]
        index => "%{[@metadata][_index]}"
        user  => "elastic"
        password => "Qv62Q1O3zNY8R885V6kp0Spg"
    }
}
```

```
multielasticdump \
  --direction=dump \
  --match='^.*$' \
  --input=http://lb.testing.bbdops.com:9200 \
  --output=/tmp/es_backup
```

