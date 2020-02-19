# ClassNotFoundException与NoClassDefFoundError

当 JVM 在 classpath 中找不到类时会报这两个错误，尽管看起来有些相似，但是他们还是有一些不同点的。

## ClassNotFoundException

ClassNotFoundException 是编译期异常，当编译器尝试通过完全限定名加载类，但在 classpath 中找不到其定义时发生。

通常是在使用反射时爆出异常，比如*Class.forName()*, *ClassLoader.loadClass()* or *ClassLoader.findSystemClass()*.

比如：

```java
@Test(expected = ClassNotFoundException.class)
public void givenNoDrivers_whenLoadDriverClass_thenClassNotFoundException() 
  throws ClassNotFoundException {
      Class.forName("oracle.jdbc.driver.OracleDriver");
}
```

## NoClassDefFoundError

NoClassDefFoundError 是一个致命错误，发生在运行期，一旦发生，程序就会退出。

发生场景：

- new 一个实例变量时
- 方法调用时

在编译期是可以正常编译的，但是在运行期，找不到类文件就会发生这个错误。也可能是在初始化静态块或者类的静态字段时发生异常，因此抛出错误，如：

```java
public class ClassWithInitErrors {
    static int data = 1 / 0;
}

public class NoClassDefFoundErrorExample {
    public ClassWithInitErrors getClassWithInitErrors() {
        ClassWithInitErrors test;
        try {
            test = new ClassWithInitErrors();
        } catch (Throwable t) {
            System.out.println(t);
        }
        test = new ClassWithInitErrors();
        return test;
    }
}

@Test(expected = NoClassDefFoundError.class)
public void givenInitErrorInClass_whenloadClass_thenNoClassDefFoundError() {
  
    NoClassDefFoundErrorExample sample
     = new NoClassDefFoundErrorExample();
    sample.getClassWithInitErrors();
}
```



## 几种解决方法

1. 确定 classpath 中添加了包含该类的 jar
2. 类的全限定名不要出现两个，比如引用了两个版本的 jar 包
3. 如果 JVM 使用多个类加载器，则一个类加载器加载的类可能无法被其他类加载器使用