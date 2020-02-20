# Kibana 错误记录

### 1，界面内索引错误

报错：

```
Error: [illegal_argument_exception] Fielddata is disabled on text fields by default. Set fielddata=true on [monitor.type] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead.
```

需要执行：

```
PUT heartbeat-7.5.1-*/_mapping
{
  "properties": {
    "monitor.type": { 
      "type":     "text",
      "fielddata": true
    }
  }
}
```

