# Golang 类型转换

## int 到 string

```go
str := strconv.Itoa(10)
```



## string 到 int

```
nodePort,_ := strconv.Atoi(nodePortStr)
```





## string 转 byte数组

```go
var data []byte = []byte(str)
```

反转：

```go
str2 = string(data2[:])
```



## 打印结构体

```go
fmt.Printf("%+v\n", zooKeeperInstance)
```

