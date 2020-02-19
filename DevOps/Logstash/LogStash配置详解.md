# Logstash 配置详解

Logstash 的配置分为三段：

```
# This is a comment. You should use comments to describe
# parts of your configuration.
input {
  ...
}

filter {
  ...
}

output {
  ...
}
```

每段都可以包含一或多个插件，filter 可以配置多个

使用插件时，需要一个插件名跟一个代码块：

```
input {
  file {
    path => "/var/log/messages"
    type => "syslog"
  }

  file {
    path => "/var/log/apache/access.log"
    type => "apache"
  }
}
```

后面再说 input插件，filter 插件， output 插件和 codec 插件。

## 值类型

### 数组

例子：

```
users => [ {id => 1, name => bob}, {id => 2, name => jane} ]
```

### Lists

```
  path => [ "/var/log/messages", "/var/log/*.log" ]
  uris => [ "http://elastic.co", "http://example.net" ]
```

### Boolean

```
ssl_enable => true
```

### Bytes

```
  my_bytes => "1113"   # 1113 bytes
  my_bytes => "10MiB"  # 10485760 bytes
  my_bytes => "100kib" # 102400 bytes
  my_bytes => "180 mb" # 180000000 bytes
```

### Codec

```
  codec => "json"
```

### Hash

```
match => {
  "field1" => "value1"
  "field2" => "value2"
  ...
}
# or as a single line. No commas between entries:
match => { "field1" => "value1" "field2" => "value2" }
```

### Number

```
 port => 33
```

### Password

密码不会被打印

```
my_password => "password"
```

### URI

```
 my_uri => "http://foo:bar@example.net"
```

### Path

```
my_path => "/tmp/logstash"
```

### String

### Escape Sequences

转译序列：

| Text | Result                     |
| ---- | -------------------------- |
| \r   | carriage return (ASCII 13) |
| \n   | new line (ASCII 10)        |
| \t   | tab (ASCII 9)              |
| \\   | backslash (ASCII 92)       |
| \"   | double quote (ASCII 34)    |
| \'   | single quote (ASCII 39)    |

### 注释

```
# this is a comment

input { # comments can appear at the end of a line, too
  # ...
}
```



## 在配置中访问事件中的字段和数据

在 input 中不能访问，可以在 filter 和 output 中访问。

例子：

```json
{
  "agent": "Mozilla/5.0 (compatible; MSIE 9.0)",
  "ip": "192.168.24.44",
  "request": "/index.html"
  "response": {
    "status": 200,
    "bytes": 52353
  },
  "ua": {
    "os": "Windows 7"
  }
}
```

可以这样获取字段值：

```
output {
  statsd {
    increment => "apache.%{[response][status]}"
  }
}
```

可以看到，以%{} 括起来，每个字段名用 [] 括起来。

```
output {
  file {
    path => "/var/log/%{type}.%{+yyyy.MM.dd.HH}"
  }
}
```

也可以使用 if else:

```
filter {
  if [action] == "login" {
    mutate { remove_field => "secret" }
  }
}
```

```
filter {
  if [foo] in [foobar] {
    mutate { add_tag => "field in field" }
  }
  if [foo] in "foo" {
    mutate { add_tag => "field in string" }
  }
  if "hello" in [greeting] {
    mutate { add_tag => "string in field" }
  }
  if [foo] in ["hello", "world", "foo"] {
    mutate { add_tag => "field in list" }
  }
  if [missing] in [alsomissing] {
    mutate { add_tag => "shouldnotexist" }
  }
  if !("foo" in ["hello", "world"]) {
    mutate { add_tag => "shouldexist" }
  }
}
```

not in：

```
output {
  if "_grokparsefailure" not in [tags] {
    elasticsearch { ... }
  }
}
```

@metadata:

```
input { stdin { } }

filter {
  mutate { add_field => { "show" => "This data will be in the output" } }
  mutate { add_field => { "[@metadata][test]" => "Hello" } }
  mutate { add_field => { "[@metadata][no_show]" => "This data will not be in the output" } }
}

output {
  if [@metadata][test] == "Hello" {
    stdout { codec => rubydebug }
  }
}
```

利用 @metadata 更改时间格式：

```
input { stdin { } }

filter {
  grok { match => [ "message", "%{HTTPDATE:[@metadata][timestamp]}" ] }
  date { match => [ "[@metadata][timestamp]", "dd/MMM/yyyy:HH:mm:ss Z" ] }
}

output {
  stdout { codec => rubydebug }
}
```

## 使用环境变量

可以使用环境变量：${var}

在 Logstash 启动时，会将环境变量值进行替换。

替换是区分大小写的。

对未定义变量的引用会引发 Logstash 配置错误。

如果想使用默认值，可以这样子：${var:default value}

环境变量的值可以是任何值

环境变量是不可变的，必须重启才可以利用新的变量值。

例子：

```
export TCP_PORT=12345
```

```
input {
  tcp {
    port => "${TCP_PORT}"
  }
}
```

在启动后，Logstash 会替换成：

```
input {
  tcp {
    port => 12345
  }
}
```

如果没有配置环境变量，Logstash 会返回配置错误，通过设置默认值来避免错误：

```
input {
  tcp {
    port => "${TCP_PORT:54321}"
  }
}
```

