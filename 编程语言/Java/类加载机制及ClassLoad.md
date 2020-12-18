# 类加载机制及ClassLoad

## 一，类生命周期

包括：加载（Loading），连接（Linking），初始化(Initialization)，使用(Using)，卸载(Unloading)

其中连接（Linking）又包括 验证（Verification），准备(Preparation)，解析(Resolution)

加载，连接，初始化这三部分就是类加载机制！

*顺序问题*

> 加载，验证，准备，初始化，卸载，这五部分的必须从前到后进行
> 解析的时间不确定，***因为要支持java语言的运行时绑定***

java使用的是动态加载（运行期类加载），即在程序运行时需要什么类就加载什么类！

那通过什么方式来触发类加载？*有且只有*下面五种方法

1. 使用new实例化对象时，使用类的静态字段时(***被final修饰，已在编译器把结果放入常量池的静态字段除外\***？书上这么写的。不太懂)，使用类的静态方法时
2. 使用java.lang.reflect包中的方法对类进行反射调用时
3. 根据加载顺序，需要先加载父类
4. 虚拟机启动时，加载含有main方法的类
5. ***java.lang.invoke.MethodHandle***?

*------------------------------------*

*下面有几点需要注意的地方*

```java
public class NotInitialization {
	public static void main(String[] args)
    {
        System.out.println(SubClass.value);
    }
}

class SuperClass{
    static
    {
        System.out.println("SuperClass init!");
    }
 
    public static int value = 123;
 
    public SuperClass()
    {
        System.out.println("init SuperClass");
    }
}
class SubClass extends SuperClass{
    static
    {
        System.out.println("SubClass init");
    }
 
    public SubClass()
    {
        System.out.println("init SubClass");
    }
}
```

结果：

```
SuperClass init!
123
```



为什么没有输出SubClass init？

回答：对于静态字段，只有直接定义这个字段的类才会被初始化，因此通过其子类来引用父类中定义的静态字段，只会触发父类的初始化而不会触发子类的初始化。

对于这种现象来说，不同的jvm有不同的实现，当然我们常用的jvm是上面这种情况。

这里如果把value放在SubClass中，就会输出SubClass init了！

*-----------------下面也需要注意-------------------*

```java
public class NotInitialization {
   public static void main(String[] args)
    {
        SuperClass[] su = new SuperClass[10];
    }
}
```

main方法改成这样之后不会有任何输出，可以看出定义数组不会加载类。

不过，这里有个重点,这段代码会触发一个由虚拟机自动生成的，直接继承于Object的子类的加载(通过newarraay指令触发)，这个自动生成的类代表了一维数组，其中有length属性和clone()方法。这就是为什么数组能直接使用length的原因，由此还看出使用clone()方法可以直接复制数组！

*--------------------还有一点需要注意----------------*

```java
public class NotInit {

	public static void main (String[] args) {
		System.out.println(ConstClass.HELLO);
	}
}

class ConstClass{
	static {
		System.out.println("ConstClass static init");
	}
	
	static final String HELLO = "hello world";
}
```

输出：hello world

这里不触发ConstClass的初始化，这是因为这编译阶段通过常量传播优化，把hello world的值存储到了NotInit类的常量池中了，也就是说，实际NotInit中并没有ConstClass的引用！

-------------------接口的加载---------------------

接口中不能使用static{}语句块，但编译器仍为接口生成<clinit>()类构造器，用于初始化接口中所定义的成员变量。

接口的初始化场景与类基本相同，不过接口不同的是当其加载时，其父接口不会加载，除非子接口使用了父接口。

## 二，类加载过程

#### 1，加载

加载阶段需要做的事情：

1. 通过类的全限定名来获取此类的二进制流。
2. 将这个类中的静态储存结构转化为方法区上的运行时数据结构
3. 在内存中生成一个代表这个类的Class对象，作为方法区这个类的各种数据的访问入口

二进制流的来源：

1. 从zip包读取，与之类似，也可以从JAR,EAR,WAR包读取
2. 从网络中获取
3. *运行时计算生成，使用最多的是动态代理技术，java.lang.reflect.Proxy*
4. jsp文件生成的Class类
5. 从数据库读取

这个阶段对于开发人员来说，可控性是最强的！因为除了使用系统提供的类加载器外，开发者可以自定义类加载器，开发人员可以通过继承ClassLoad，重写loadClass()方法来控制字节流的获取方式

为提高速度，在加载阶段未完成之前，就开始了连接了，不过加载阶段完不成，连接阶段的验证阶段也休想完成，所以这些阶段还是保持着一定的顺序的。

#### 2，验证

验证的目的是确保二进制字节流符合当前虚拟机的要求，让他不会危害虚拟机自身的安全。

使用正常的Java代码生成的Class文件是符合虚拟机规范的，但是Class文件不一定就是Java代码转换来的，还有可能是手写的！所以jvm要对二进制字节流进行验证。

*这个阶段是否严谨，直接决定了jvm是否能承受恶意代码的攻击。*

四个阶段的检验动作：文件格式验证，元数据验证，字节码验证，符号引用验证。

①文件格式验证

1. 是否以魔数0xCAFEBABE开头
2. 主次版本号是否在jvm能接受的范围之中
3. 常量池中的常量类型是不是都支持
4. 检查常量的各种索引值是否指向不存在的常量或不符合类型的常量
5. CONSTANT_Utf-8_info型的常量是否有不符合UTF8编码的数据
6. class文件中的各个部分及文件本身是否有被删除或附加的其他信息

还有很多，总之就是根据Class文件的格式和限制来验证的！只有通过这个阶段的验证后，字节流才会进入内存的方法区进行储存，所以后面的三个验证阶段全部基于方法区的储存结构进行的，不会再直接操作字节流。

从这也可以看出加载完成之前是一定要经过一部分验证阶段的！

②元数据验证

这个阶段对字节码描述的信息进行语义分析，验证点：

1. 这个类是否有父类
2. 这个类是否继承了不允许继承的类
3. 如果这个类不是抽象类，是否都实现了其父类或接口之中要求实现的所有方法
4. 类中的字段，方法是否与父类产生矛盾

这个阶段主要目的是对类的元数据(类型，域，方法)进行语义校验，保证不存在不符合语言规范的元数据信息。

③字节码验证

主要目的是通过数据流和控制流分析，确定程序语义是合法的，符合逻辑的。第二阶段对元数据信息中的数据类型做完校验后，这个阶段对类的方法体进行校验分析，保证方法在运行时不会做出危害虚拟机安全的事件！

1. 保证任意时刻操作数栈上的数据类型和指令都正确
2. 保证跳转指令不会跳转到方法体之外
3. 保证方法体中的类型转换都是有效的

这个阶段的jvm实现起来是最难的！

④符号引用验证

最后一个阶段的验证发生在虚拟机将符号引用转化为直接引用的时候，**这个转化的动作将在链接的第三个阶段---解析阶段完成。**

符号引用验证可以看做是对类自身之外的信息进行匹配性校验，验证的内容：

1. 符号引用中通过字符串描述的权限定名是否能找到对应的类
2. 在指定类中是否存在方法的字段描述性以及简单名称所描述的方法和字段
3. 符号引用中的类，字段，方法的访问性是否可以被当前类访问

如果出错会抛出错误！

这个阶段不是必要的，如果能保证字节码全部是安全的可以使用-Xverify:none参数来关闭大部分的验证措施，来缩短类加载的时间

#### 3，准备

准备阶段是正式为类变量分配内存并设置初始值的阶段，这些变量所使用的内存都将在方法区中进行分配。

注意，这时进行内存分配的仅包括类变量，不包括实例变量，实例变量将会在对象实例化时随着对象一起在堆中分配。

还要注意，这里的内存分配完后，类变量的初始值为都为0，这时还未执行任何方法，而赋值操作是在类构造器<clinit>()中进行的。***特殊的是final，final定义的类变量在这个阶段直接赋予指定的值，而不是0*** 。

#### 4，解析

解析阶段是将常量池内的符号引用替换为直接引用的过程。

> 常量池中的符号引用：符号引用以一组符号(字符串字面量，定义在class文件格式中)描述所引用的目标
> 直接引用：是直接指向目标的指针，相对偏移或能间接定位到目标的句柄

解析阶段的时间并不固定，只要在使用这些符号之前就行解析就行，

#### 5，初始化

这是类加载的最后一步，到了这个阶段，才开始执行java代码(字节码)，初始化阶段是执行类构造器的<clinit>()的过程，<clinit>()的执行过程：

1. <clinit>()中的语句是所有类变量和静态语句块按从***前到后的顺序***组合而成的，静态语句块只能使用定义在它前面的类变量，对于它后面的类变量，它只能赋值，不能访问。
2. <clinit>()方法和构造函数(<init>()方法)不同，它不需要显示的调用父类构造器，虚拟机保证在执行<clinit>()之前，其父类的<clinit>()已执行完毕，因此在jvm中第一个执行的<clinit>()方法肯定是Object
3. 由于父类<clinit>()方法先执行，也就意味着父类的静态语句块要先与子类的静态语句块执行，
4. 如果没有静态变量和静态块，则编译器不会为这个类生成<clinit>()方法
5. 接口不能有静态初始化块，但仍然有静态变量赋值的语句，所以接口也有<clinit>()方法，不过接口的<clinit>()执行之前不会执行父接口的<clinit>()方法，接口的实现类在初始化时也一样不会执行接口的<clinit>()方法
6. 虚拟机会保证一个类的<clinit>()方法被正确的加锁同步，若多个线程同时要初始化同一个类，那只有一个线程执行<clinit>()方法，不过这个线程执行完之后，其他线程就不需要执行了，因为在同一个类加载器下，一个类型只会初始化一次。



## 三，类加载器

jvm允许开发人员自定义类加载器，以便让应用程序自己决定如何去获取所需的二进制流！

这是一项创新，是为了Java Applet创造的，不过Java Applet已经死掉，但自定义类加载器却在类层次划分，代码加密，远程调用等方面大放异彩。可以说是失之桑榆，收之东偶。



#### 1，类与类加载器

两个类相同的条件是，同一个类加载器，相同的全限定名。即使两个类来源于同一个class文件，被同一个虚拟机加载，只要加载他们的类加载器不同，那这两个类就不相等。

这两个所指的"相等"包括代表Class对象的equals(),isAssignableFrom(),和isInstance()方法，也包括使用instanceof关键字做对象所属关系的判定。

```java
public class Ser {

	 public static void main(String... strings) throws ClassNotFoundException {
		 ClassLoader my = new ClassLoader() {
			 @Override
			 public Class<?> loadClass(String name) throws ClassNotFoundException {
				 String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
				 InputStream is = getClass().getResourceAsStream(fileName);
				 if (is == null) {
					 return super.loadClass(name);
				 }
				 byte[] b;
				try {
					b = new byte[is.available()];
					is.read(b);
					return defineClass(name, b, 0, b.length);
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				return null;
			 }
		 };
		 Object obj = my.loadClass("ser.ConstClass");
		System.out.println(obj.getClass());
		System.out.println(obj instanceof ser.ConstClass);
	 }
   
}
```

输出：

```
class ser.ConstClass
false
```



#### 2，双亲委派模型

java虚拟机有两种类加载器，一种是启动类加载器(Bootstrap ClassLoad)，这个类加载器在jvm内部，又c++实现。另一种就是其他的类加载器，这些类加载器使用Java实现，独立于虚拟机外部，并且全部继承于抽象类java.lang.ClassLoad

1. 启动类加载器，加载$JAVA_HOME/jre/lib下的目录，rt.jar就在这个目录下
2. 扩展类加载器，由sun.misc.Launcher$ExtClassLoader实现，负责加载$JAVA_HOME/jar/lib/ext目录下的类
3. 应用程序类加载器，这个类加载器负责加载用户类路径上所指定的类库

类加载器的这种层次关系被称为双亲委派模型，这里的父子类加载器不是以继承的关系来实现爱你的，而是以组合的关系实现的。

每个类加载器在加载类时，都先让父加载器加载，如果不在父加载器的范围内，父加载器不加载，这时子加载器才自己加载

*使用双亲委派模型的好处就是java类之间有一种层级关系，rt.jar中的类必须被启动类扩展器加载，而不能被别的类加载器加载！双亲委派模型对于保证Java程序的稳定运作很重要！*

可以查看ClassLoad类的loadClas()方法：

```java
  protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

这个方法先查看类是否已经加载了，若没有加载就先委托父类进行加载，若父类找不到，抛出异常后子类再进行加载！

#### 3，"被破坏"的双亲委派模型

有三种情况：

1. ClassLoad的产生时间早于双亲委派模型的产生时间，所以在早期都不遵守双亲委派模型。考虑到兼容性，至今loadCload()方法至今都是能够重写的，其实我们应该重写findClass()方法，来保证双亲委派模型。
2. 存在与rt.jar中的JNDI，JDBC等服务需要调用用户的业务类，所以不得已才允许父加载器要求子加载器加载类的情况，线程上下文的类加载器就是用于这种情况的！
3. 因为业界强烈要求实现热加载，实现模块化，所以有了OSGI，热加载就是使用了类加载机制，通过在程序运行时，替换类加载器来做到的，不过这样破坏了双亲委派模型

被破坏不一定是坏的，这里的OSGI就很好，不过存在争议，并且Java9也更新了，其主打的就是模块化。



完全参考：《深入理解Java虚拟机》