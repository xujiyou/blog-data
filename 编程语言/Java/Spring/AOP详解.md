# AOP 详解



## AOP

AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑
的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。



- `Aspect`（切面）： Aspect 声明类似于 Java 中的类声明，在 Aspect 中会包含着一些 Pointcut 以及相应的 Advice。

- `Joint point`（连接点）：表示在程序中明确定义的点，典型的包括方法调用，对类成员的访问以及异常处理程序块的执行等等，它自身还可以嵌套其它 joint point。

- `Pointcut`（切点）：表示一组 joint point，这些 joint point 或是通过逻辑关系组合起来，或是通过通配、正则表达式等方式集中起来，它定义了相应的 Advice 将要发生的地方。

- `Advice`（增强）：Advice 定义了在 `Pointcut` 里面定义的程序点具体要做的操作，它通过 before、after 和 around 来区别是在每个 joint point 之前、之后还是代替执行的代码。

- `Target`（目标对象）：织入 `Advice` 的目标对象.。

- `Weaving`（织入）：将 `Aspect` 和其他对象连接起来, 并创建 Adviced object 的过程



## 实现原理

我们现在了解了代码中如何实现，那么AOP实现的原理是什么呢？之前看了一个博客说到，提到AOP大家都知道他的实现原理是动态代理，显然我之前就是不知道的，哈哈，但是相信阅读文章的你们一定是知道的。

讲到动态代理就不得不说代理模式了， 代理模式的定义：给某一个对象提供一个代理，并由代理对象控制对原对象的引用。代理模式包含如下角色：subject：抽象主题角色，是一个接口。该接口是对象和它的代理共用的接口; RealSubject：真实主题角色，是实现抽象主题接口的类; Proxy:代理角色，内部含有对真实对象RealSubject的引用，从而可以操作真实对象。代理对象提供与真实对象相同的接口，以便代替真实对象。同时，代理对象可以在执行真实对象操作时，附加其他的操作，相当于对真实对象进行封装。如下图所示：



## 动态代理

参考：https://www.liaoxuefeng.com/wiki/1252599548343744/1264804593397984

Java标准库提供了一种动态代理（Dynamic Proxy）的机制：可以在运行期动态创建某个`interface`的实例。

























