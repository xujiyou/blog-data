# Hive 数据类型

基本的数据类型，比如数字、字符串、布尔，甚至二进制这种，都很简单，不说了，主要看 Hive 支持的三种复合数据类型：Struct、Map、Array。

创建表：

```
create table first (
  arr ARRAY<STRING>,
  m MAP<STRING, STRING>,
  address STRUCT<streat:STRING, city:STRING, state:STRING, zip: INT>
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '|'
COLLECTION ITEMS TERMINATED BY ','
MAP KEYS TERMINATED BY ':'
STORED AS TEXTFILE;
```

> 注意这里的 `STORED AS TEXTFILE`，只有 TEXTFILE 格式的数据才能从 txt 文件中导入！默认格式是 ORC 的，如果想从本地的文本文件传到   HDFS 的 ORC 格式的文件中，可以这样子：
>
> ```
> create table first_orc (
>   arr ARRAY<STRING>,
>   m MAP<STRING, STRING>,
>   address STRUCT<streat:STRING, city:STRING, state:STRING, zip: INT>
> )
> ROW FORMAT DELIMITED
> FIELDS TERMINATED BY '|'
> COLLECTION ITEMS TERMINATED BY ','
> MAP KEYS TERMINATED BY ':';
> 
> INSERT INTO TABLE first_orc SELECT * FROM first;
> ```

插入数据：

```s
insert into table first
  values (
     array('aaa'),
     map('key1', 'value'),
     named_struct('streat', 'aaa', 'city', 'aaa', 'state', 'aaa', 'zip', 1)
  );
```

查找 struct 下的某一项值：

```sql
select address.streat from first;
```

从文件中加载数据：

文件内容如下：

```
aaa,bbb,ccc|aaa:bbb,ccc:ddd|bbb,bbb,bbb,2
```

填入数据：

```
LOAD DATA LOCAL INPATH '/tmp/test.txt' OVERWRITE INTO TABLE first;
```

如果 zip 不是 int ，则会被设置为 null ！























