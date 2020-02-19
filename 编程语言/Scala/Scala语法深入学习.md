# Scala 语法深入学习

经过基础语法学习后，再来学习一下更深入的内容：类型系统，类，trait，元组，with，高阶函数，嵌套函数，case class，模式匹配，单例类，正则表达式，引入包，for 表达式，泛型，多态。

## 类型系统

一张图了解全部类型：

![unified-types-diagram](https://docs.scala-lang.org/resources/images/tour/unified-types-diagram.svg)

如果在定义时，不知道要具体使用哪些类型，可以使用 Any 或 AnyRef 。

如果在使用时，没有具体实例可以传入，可以考虑传入 Nothing 或 Null 实例。

```scala
val list: List[Any] = List(
  "a string",
  732,  // an integer
  'c',  // a character
  true, // a boolean value
  () => "an anonymous function returning a string"
)

list.foreach(element => println(element))
```

隐式转换：

![type-casting-diagram](https://docs.scala-lang.org/resources/images/tour/type-casting-diagram.svg)

另外，Null 只拥有一个实例，就是关键字 `null`

