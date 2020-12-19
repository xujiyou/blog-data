# IoC 详解



## IoC

`IoC`（Inversion of Control，控制倒转）。这是`spring`的核心，贯穿始终。所谓`IoC`，对于`spring`框架来说，就是由`spring`来负责控制对象的生命周期和对象间的关系。

- **正控：**若要使用某个对象，需要**自己去负责对象的创建**
- **反控：**若要使用某个对象，只需要**从 Spring 容器中获取需要使用的对象，不关心对象的创建过程**，也就是把**创建对象的控制权反转给了Spring框架**
- **好莱坞法则：**Don’t call me ,I’ll call you



好处：降低对象之间的耦合，我们不需要理解一个类的具体实现，只需要知道它有什么用就好了（直接向 IoC 容器拿）。





## DI

DI(Dependency Injection)依赖注入

IOC其实有两种方式，一种就是DI，而另一种是DL，即Dependency Lookup（依赖查找），前者是当前软件实体被动接受其依赖对其他组件被IOC容器注入，而后者则是当前软件实体主动去某个服务注册地查找其依赖对那些服务，概念之间对关系图如下图所示可能更贴切些。



## 解决循环依赖

配置信息如下：

```
<bean id="beanA" class="xyz.coolblog.BeanA">
    <property name="beanB" ref="beanB"/>
</bean>
<bean id="beanB" class="xyz.coolblog.BeanB">
    <property name="beanA" ref="beanA"/>
</bean>
```

IOC 容器在读到上面的配置时，会按照顺序，先去实例化 beanA。然后发现 beanA 依赖于 beanB，接在又去实例化 beanB。实例化 beanB 时，发现 beanB 又依赖于 beanA。如果容器不处理循环依赖的话，容器会无限执行上面的流程，直到内存溢出，程序崩溃。当然，Spring 是不会让这种情况发生的。在容器再次发现 beanB 依赖于 beanA 时，容器会获取 beanA 对象的一个早期的引用（early reference），并把这个早期引用注入到 beanB 中，让 beanB 先完成实例化。beanB 完成实例化，beanA 就可以获取到 beanB 的引用，beanA 随之完成实例化。

在才开始创建对象时，他内部的域都是 null。在填充域时将缓存中的对象填上即可。