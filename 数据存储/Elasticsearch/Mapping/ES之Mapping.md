# ES 之 Mapping

在 ES 7.x 版本中，不推荐在 api 中使用类型（type），直接在索引下直接储存文档即可。

## 字段数据类型

### alias

别名，字段的别名，用法：

```json
PUT trips
{
  "mappings": {
    "properties": {
      "distance": {
        "type": "long"
      },
      "route_length_miles": {
        "type": "alias",
        "path": "distance" 
      },
      "transit_mode": {
        "type": "keyword"
      }
    }
  }
}
```

在 PUT 中使用 mappings，就是对文档的字段进行定义。

别名在使用时使用别名代替字段名：

```json
GET _search
{
  "query": {
    "range" : {
      "route_length_miles" : {
        "gte" : 39
      }
    }
  }
}
```

### arrays

数组中的数据类型必须是一样的，例子：

```java
PUT my_index/_doc/1
{
  "message": "some arrays in this document...",
  "tags":  [ "elasticsearch", "wow" ], 
  "lists": [ 
    {
      "name": "prog_list",
      "description": "programming list"
    },
    {
      "name": "cool_list",
      "description": "cool stuff list"
    }
  ]
}

PUT my_index/_doc/2 
{
  "message": "no arrays in this document...",
  "tags":  "elasticsearch",
  "lists": {
    "name": "prog_list",
    "description": "programming list"
  }
}

GET my_index/_search
{
  "query": {
    "match": {
      "tags": "elasticsearch" 
    }
  }
}
```

### binary

二进制类型接收二进制值作为Base64编码字符串。默认情况下，该字段不存储且不可搜索：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "blob": {
        "type": "binary"
      }
    }
  }
}

PUT my_index/_doc/1
{
  "name": "Some binary blob",
  "blob": "U29tZSBiaW5hcnkgYmxvYg==" 
}
```

### bool

`true` 和 `false` 都是 bool，同时 ，字符串 `"true"` `"false"` 也是 bool 类型：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "is_published": {
        "type": "boolean"
      }
    }
  }
}

POST my_index/_doc/1
{
  "is_published": "true" 
}

GET my_index/_search
{
  "query": {
    "term": {
      "is_published": true 
    }
  }
}
```

### date

日期类型，由于 json 没有日期类型，所以 ES 将以下格式的类型看作是日期类型：

- 包含日期格式的字符串，如："2015-01-01"` or `"2015/01/01 12:10:30"
- 一个 long 类型的13位的表示毫秒的时间戳
- 一个 int 类型的 10位的表示秒的时间戳

在 ES 内部，日期转换为 UTC 格式的，并以毫秒级的时间戳储存，当搜索时又转换为相应的格式。

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "date": {
        "type": "date" 
      }
    }
  }
}

PUT my_index/_doc/1
{ "date": "2015-01-01" } 

PUT my_index/_doc/2
{ "date": "2015-01-01T12:10:30Z" } 

PUT my_index/_doc/3
{ "date": 1420070400001 } 

GET my_index/_search
{
  "sort": { "date": "asc"} 
}
```

### dense_vector

储存浮点数的数组，规定了数量，最大为1024。

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "my_vector": {
        "type": "dense_vector",
        "dims": 3  
      },
      "my_text" : {
        "type" : "keyword"
      }
    }
  }
}

PUT my_index/_doc/1
{
  "my_text" : "text1",
  "my_vector" : [0.5, 10, 6]
}

PUT my_index/_doc/2
{
  "my_text" : "text2",
  "my_vector" : [-0.5, 10, 10]
}
```

## geo_point

经纬度类型：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}

PUT my_index/_doc/1
{
  "text": "Geo-point as an object",
  "location": { 
    "lat": 41.12,
    "lon": -71.34
  }
}

PUT my_index/_doc/2
{
  "text": "Geo-point as a string",
  "location": "41.12,-71.34" 
}

PUT my_index/_doc/3
{
  "text": "Geo-point as a geohash",
  "location": "drm3btev3e86" 
}

PUT my_index/_doc/4
{
  "text": "Geo-point as an array",
  "location": [ -71.34, 41.12 ] 
}

PUT my_index/_doc/5
{
  "text": "Geo-point as a WKT POINT primitive",
  "location" : "POINT (-71.34 41.12)" 
}

GET my_index/_search
{
  "query": {
    "geo_bounding_box": { 
      "location": {
        "top_left": {
          "lat": 42,
          "lon": -72
        },
        "bottom_right": {
          "lat": 40,
          "lon": -74
        }
      }
    }
  }
}
```

