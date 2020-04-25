# Golang 类型转换

## int 到 string

```go
str := strconv.Itoa(10)
```



## string 转 byte数组

```
var data []byte = []byte(str)
```

反转：

```
str2 = string(data2[:])
```

