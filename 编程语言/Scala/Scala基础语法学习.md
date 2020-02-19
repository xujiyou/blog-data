# Scala 基础语法学习

Scala 是 Java 的一种方言，我个人比较喜欢，主要是喜欢它对函数式编程范式的支持，然后是喜欢它一些现代化语言的一些特性。

## 环境配置

工欲善其事必先利其器，第一步要搭建好 Scala 的编程环境。

首先需要在 IDEA 中安装 Scala 插件，关于 Scala SDK ，可以在官网下载SDK，然后在 IDEA 中配置一下，也可以直接在 Maven 中添加 Scala SDK 的依赖。

如果想使用本地 Scala SDK，可以在 IDEA 中新建 Maven 项目，然后点 File -> Project Structure -> Global Libraries 添加 Scala SDK 即可。

如果不配置 Scala SDK，IDEA 会给出提示的，按照提示进行下载也行。

也可以直接在 Maven 中添加以下配置，版本可以去 Maven 中央库搜：

```xml
<properties>
    <scala.version>2.13.1</scala.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.scala-lang</groupId>
        <artifactId>scala-library</artifactId>
        <version>${scala.version}</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.scala-tools</groupId>
            <artifactId>maven-scala-plugin</artifactId>
            <executions>
                <execution>
                    <goals>
                        <goal>compile</goal>
                        <goal>testCompile</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

maven-scala-plugin 这个插件是用来让maven能够编译、测试、运行scala项目的。

如果想用 Maven 打包 Scala 项目，可以参考 [插件打包介绍](https://davidb.github.io/scala-maven-plugin/example_java.html)

这样写好 pom.xml 后，就不用自己下载 Scala SDK 了，简单方便，不过要学一下这个 Maven 插件怎么配置。

推荐使用 Maven 配置的方式获取 Scala SDK，因为这样打包后不会跟编辑器互相纠缠。

## Scala 特性

Scala 即有面向对象的特性，也有函数式编程的特性，另外还可以和 Java 互操作。

## Scala 基础语法

Scala 的官方教程都在：https://docs.scala-lang.org/

Scala 中，主方法是这样子写的，main 方法必须定义在 object 中才能生效，因为 Scala 中没有 static 关键字，只能将方法放在 object 中，才算是类方法：

```scala
object Main {
  def main(args: Array[String]): Unit =
    println("Hello, Scala developer!")
}
```

和 Java 一样的基本类型的语法，不过末尾不用加分号：

```scala
1 + 1
println(1) // 1
println(1 + 1) // 2
println("Hello!") // Hello!
println("Hello," + " world!") // Hello, world!定义变量有两种关键字：val 与 var ，val 定义的关键字不可变，相当于其他语言中的 final ，不可变对线程安全有很大帮助。
```

Scala 是一门静态强类型语言，不过 Scala 可以自动推断变量类型，如果想给变量加一个类型，可以在变量名后边加一个冒号，然后跟类型，这跟 Golang、Swift、TypeScript、Rust是相同的，现代化的语言都这么玩。

```scala
val x = 1 + 1
println(x) // 2
//x = 3 // 不能编译，编辑器也会提示错误
var x = 1 + 1
x = 3 // This compiles because "x" is declared with the "var" keyword.
println(x * x) // 9
var x: Int = 1 + 1
```

块语法，以一组大括号组成，大括号内的最后一个语句的结果就是块的返回值：

```scala
println({
  val x = 1 + 1
  x + 1
}) // 3
```

函数，函数可以定义在任何地方，下面定义了一个匿名函数：

```scala
(x: Int) => x + 1
```

Scala 中，函数可以赋值给一个变量，这样的函数参数后必须跟一个 => 才行，其后的大括号是可选的，默认以最后一条语句的结果为返回值：

```scala
val addOne = (x: Int) => x + 1
println(addOne(1)) // 2
val add = (x: Int, y: Int) => {x + y}
println(add(1, 2)) // 3
val getTheAnswer = () => 42
println(getTheAnswer()) // 42
```

上面这种写法不可以标明返回值类型，只能靠 Scala 推断。

也可以跟方法一样定义一个函数，其中，参数后面的返回值类型是可选的，如果函数体只有一样，大括号也是可选的，但是等号必须写：

```scala
def abc (x: Int): Int = {
  x + 1
}
```

方法，方法的写法跟 def 定义的函数的写法是一样的，不过必须写在类下面，作为一个成员变量存在。

特殊的是，Scala 允许存在多个参数列表，如：

```scala
def addThenMultiply(x: Int, y: Int)(multiplier: Int): Int = (x + y) * multiplier
println(addThenMultiply(1, 2)(3)) // 9
```

也可以是没有参数，跟变量的定义差不多：

```scala
def name: String = System.getProperty("user.name")
println("Hello, " + name + "!")
```

类，Scala 中的构造器可以直接写在类后面，如：

```scala
class Greeter(prefix: String, suffix: String) {
  def greet(name: String): Unit =
    println(prefix + name + suffix)
}

val greeter = new Greeter("Hello, ", "!")
greeter.greet("Scala developer") // Hello, Scala developer!
```

Case Class，Case 类通常用于模式匹配，只要内部的实例变量的值相等，对象就相等：

```scala
case class Point(x: Int, y: Int)

val point = Point(1, 2)
val anotherPoint = Point(1, 2)
val yetAnotherPoint = Point(2, 2)

if (point == anotherPoint) {
  println(point + " and " + anotherPoint + " are the same.")
} else {
  println(point + " and " + anotherPoint + " are different.")
} // Point(1,2) and Point(1,2) are the same.

if (point == yetAnotherPoint) {
  println(point + " and " + yetAnotherPoint + " are the same.")
} else {
  println(point + " and " + yetAnotherPoint + " are different.")
} // Point(1,2) and Point(2,2) are different.
```

object，Scala 中，是没有 static 关键字的，想要实现类变量，就需要写 object，这样就把类变量和实例变量都语法层面分开了，class 和 object 的名字可以相同：

```scala
class IdFactory {}

object IdFactory {
  private var counter = 0
  def create(): Int = {
    counter += 1
    counter
  }
}
```

在 Scala 中，是没有 interface 这个关键字的，取而代之的是 `trait` 这个关键字：

```scala
trait Greeter {
  def greet(name: String): Unit
}
```

trait 中的方法体也可以有默认的实现：

```scala
trait Greeter {
  def greet(name: String): Unit =
    println("Hello, " + name + "!")
}
```

可以使用 extends 关键字实现一个 trait ，也可以用 with 关键字实现多个 trait：

```scala
class DefaultGreeter extends Greeter

class CustomizableGreeter(prefix: String, postfix: String) extends Greeter {
  override def greet(name: String): Unit = {
    println(prefix + name + postfix)
  }
}

val greeter = new DefaultGreeter()
greeter.greet("Scala developer") // Hello, Scala developer!

val customGreeter = new CustomizableGreeter("How are you, ", "?")
customGreeter.greet("Scala developer") // How are you, Scala developer?
```

