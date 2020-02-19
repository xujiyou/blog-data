# ES 之 REST APIs

Elasticsearch公开了用户界面组件使用的 REST API，可以直接调用这些 api 来配置和访问 Elasticsearch 的特性。

## API 约定

### 可以在 API 中使用多个索引

可以使用 test1,test2,test3 来查询多个索引，如果查询所有索引，可以使用`_all` 

```json
POST my_index,my_index1/_search
{
  "query": {
    "match_all": {}
  }
}

POST _all/_search
{
  "query": {
    "match_all": {}
  }
}
```

也可以使用通配符，如： `test*` or `*test` or `te*t` or `*test*`

还可以使用排除法，如`test*,-test3` ，这就代表所有 test开头的，但不包括 test3！

包含多个索引的api还可以加以下参数：

- **ignore_unavailable** 默认false，若为 true，则关闭的索引不包含在响应中
- **allow_no_indices** 如果为true，则使用通配符索引时，如果没匹配到，就不返回数据，而不是报错。
- **expand_wildcards** 四个值：**all**  **open** **closed** **none** 

### 可以在索引中执行日期运算

索引格式，如：

```
<static_name{date_math_expr{date_format|time_zone}}>
```

- static_name 索引的静态部分
- date_math_expr 日期的函数表达式
- date_format 日期格式
- time_zone 时区，默认 utc

例子：

```
# GET /<logstash-{now/d}>/_search
GET /%3Clogstash-%7Bnow%2Fd%7D%3E/_search
{
  "query" : {
    "match": {
      "test": "data"
    }
  }
}
```

| `<`  | `%3C` |
| ---- | ----- |
| `>`  | `%3E` |
| `/`  | `%2F` |
| `{`  | `%7B` |
| `}`  | `%7D` |
| `|`  | `%7C` |
| `+`  | `%2B` |
| `:`  | `%3A` |
| `,`  | `%2C` |

例子：

| Expression                       | Resolves to                    |
| -------------------------------- | ------------------------------ |
| `<logstash-{now/d}>`             | `logstash-2024.03.22`          |
| `<logstash-{now/M}>`             | `logstash-2024.03.01`          |
| `<logstash-{now/M{yyyy.MM}}>`    | `logstash-2024.03`             |
| `<logstash-{now/M-1M{yyyy.MM}}>` | `logstash-2024.02`             |
| `<logstash-{now/d{yyyy.MM.dd`    | +12:00}}>`logstash-2024.03.23` |

如果索引中本就有 { 或 }，可以用 \ 来转译：

` <elastic\{ON\}-{now/M}>` resolves to `elastic{ON}-2024.03.01`



## 全局选项

### pretty

当给 API 加上了 ?pretty=true ，则会将结果显示成完整的 JSON 格式。如果设置 ?format=yaml ，则会显示 yaml 格式。

### 人类可读的输出

设置 ?human=false 会返回人类可读的输出，它的默认值就是 false。

### 日期计算

日期计算 包括 gte 、 lte 、from 、to 。now 代表现在，可以对 now 进行计算：

- `+1h`: Add one hour
- `-1d`: Subtract one day
- `/d`: Round down to the nearest day

字母解释如下：

| `y`  | Years   |
| ---- | ------- |
| `M`  | Months  |
| `w`  | Weeks   |
| `d`  | Days    |
| `h`  | Hours   |
| `H`  | Hours   |
| `m`  | Minutes |
| `s`  | Seconds |

输入 `2001-01-01 12:00:00`, 一些例子:

| `now+1h`              | `now` in milliseconds plus one hour. Resolves to: `2001-01-01 13:00:00` |
| --------------------- | ------------------------------------------------------------ |
| `now-1h`              | `now` in milliseconds minus one hour. Resolves to: `2001-01-01 11:00:00` |
| `now-1h/d`            | `now` in milliseconds minus one hour, rounded down to UTC 00:00. Resolves to: `2001-01-01 00:00:00` |
| `2001.02.01\|\|+1M/d` | `2001-02-01` in milliseconds plus one month. Resolves to: `2001-03-01 00:00:00` |

### 响应过滤

filter_path 可以指定不显示哪些字段。

例子：

```
GET /_search?q=elasticsearch&filter_path=took,hits.hits._id,hits.hits._score
```

也可以使用通配符：

```
GET /_cluster/state?filter_path=metadata.indices.*.stat*
```

使用符号 - :

```
GET /_count?filter_path=-_shards
```

### flat_settings

默认为 false。

例子：

```
GET twitter/_settings?flat_settings=true

结果：
{
  "twitter" : {
    "settings": {
      "index.number_of_replicas": "1",
      "index.number_of_shards": "1",
      "index.creation_date": "1474389951325",
      "index.uuid": "n6gzFZTgS664GUfx0Xrpjw",
      "index.version.created": ...,
      "index.provided_name" : "twitter"
    }
  }
}

GET twitter/_settings?flat_settings=false

结果：
{
  "twitter" : {
    "settings" : {
      "index" : {
        "number_of_replicas": "1",
        "number_of_shards": "1",
        "creation_date": "1474389951325",
        "uuid": "n6gzFZTgS664GUfx0Xrpjw",
        "version": {
          "created": ...
        },
        "provided_name" : "twitter"
      }
    }
  }
}
```

### 时间格式

| `d`      | Days         |
| -------- | ------------ |
| `h`      | Hours        |
| `m`      | Minutes      |
| `s`      | Seconds      |
| `ms`     | Milliseconds |
| `micros` | Microseconds |
| `nanos`  | Nanoseconds  |

### 大小格式

| `b`  | Bytes     |
| ---- | --------- |
| `kb` | Kilobytes |
| `mb` | Megabytes |
| `gb` | Gigabytes |
| `tb` | Terabytes |
| `pb` | Petabytes |

### 千，百万

| `k`  | Kilo |
| ---- | ---- |
| `m`  | Mega |
| `g`  | Giga |
| `t`  | Tera |
| `p`  | Peta |

### 距离

| Mile          | `mi` or `miles`                 |
| ------------- | ------------------------------- |
| Yard          | `yd` or `yards`                 |
| Feet          | `ft` or `feet`                  |
| Inch          | `in` or `inch`                  |
| Kilometer     | `km` or `kilometers`            |
| Meter         | `m` or `meters`                 |
| Centimeter    | `cm` or `centimeters`           |
| Millimeter    | `mm` or `millimeters`           |
| Nautical mile | `NM`, `nmi`, or `nauticalmiles` |

### fuzziness

| `0`, `1`, `2` | The maximum allowed Levenshtein Edit Distance (or number of edits) |
| ------------- | ------------------------------------------------------------ |
| `AUTO`        | Generates an edit distance based on the length of the term. Low and high distance arguments may be optionally provided `AUTO:[low],[high]`. If not specified, the default values are 3 and 6, equivalent to `AUTO:3,6` that make for lengths:**`0..2`**Must match exactly**`3..5`**One edit allowed**`>5`**Two edits allowed`AUTO` should generally be the preferred value for `fuzziness`. |

### error_trace

查看错误堆栈：

```
POST /twitter/_search?size=surprise_me&error_trace=true
```

## _cat API

JSON 非常适合计算，但对于人来说，更愿意看到紧凑和对齐的文本，_cat API就是来做这个事情的。

单独请求 _cat 会返回所有命令，对某一个命令加 `help` 参数，会显示命令的列名。

如果加了参数 `v`，则会为数据加上列。

`h` 参数表明要显示的列。

`bytes` 参数表示单位。

`format` 参数指定响应格式，\- text (default) - json - smile - yaml - cbor。

`pretty` 参数指定格式是否完美。

`s` 参数代表排序，例如：s=column1,column2:desc,column3

### _cat 子API

- /_cat/aliases/<alias> 查看别名列表
- /_cat/allocation/<node_id> 查看资源列表
- /_cat/count/<index> 获取文档数量
- /_cat/fielddata/<field> 返回集群中每个数据节点上fielddata(字段元数据)当前使用的堆内存量。
- /_cat/health 获取集群健康信息
- /_cat/indices/<index> 返回索引信息
- /_cat/master 返回 master 节点的信息
- /_cat/nodeattrs 查看 node 的属性
- /_cat/nodes 查看 node 信息列表
- /_cat/pending_tasks 查看未执行的任务
- /_cat/plugins 列出plugin
- /_cat/recovery/<index> 返回有关正在进行和已完成的分片恢复的信息
- /_cat/repositories
- /_cat/tasks 查看任务列表
- /_cat/thread_pool/<thread_pool> 查看线程池
- /_cat/shards/<index> 查看分片信息
- /_cat/segments 查看分片片段信息
- /_cat/snapshots/<repository>
- /_cat/templates/<template_name> 

## _cluster API



