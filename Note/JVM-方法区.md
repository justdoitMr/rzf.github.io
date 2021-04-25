# 方法区

[TOC]

## 栈堆和方法区之间交互的理解

**运行时数据区一览图**

![1608119398311](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/16/195000-419015.png)

`Jdk8`以后方法区叫做元空间，`jdk7`以及之前叫做永久代，方法区是运行时数据区的最后一个部分。

**从线程共享的角度理解运行时数据区**

![1608119486582](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/16/195128-540207.png)

`ThreadLocal`：如何保证多个线程在并发环境下的安全性？典型应用就是数据库连接管理，以及会话管理

**对象的访问和定位**

- `Person`：存放在**元空间**，也可以说方法区，也就是这个类的信息存放在方法区（元空间）。
- `person`：存放在`Java`栈的局部变量表中。
- `new Person()`：存放在`Java`堆中。

**图解内存中的对象结构**

![1608119606985](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/16/195328-47204.png)

**补充**

1. `Person`这个类对应的字节码文件放在**方法区**中。(永久代，元空间都是方法区的实现)
2. `New()`出来的对象放在**堆空间**中。
3. 对象实例数据中的指针指向此对象对应在方法区中的类型。也就是被加载到方法区的类字节码信息。
4. 这里涉及对象的访问定位问题。
5. 逻辑上可以把方法区看做堆空间的一部分，但是堆空间会有垃圾回收和压缩，具体的虚拟机的方法区不会实现垃圾回收和压缩算法，也就是说方法区独立于堆空间而存在。
6. 方法区物理上内存不要求连续，但是逻辑上必须连续。
7. 在设置堆内存大小的时候，不会影响方法区的大小。也就是说在设置方法区的大小的时候，本身就没有包含方法区的大小。

## 方法区的理解

《`Java`虚拟机规范》中明确说明：“尽管所有的方法区在逻辑上是属于堆的一部分，但一些简单的实现可能不会选择去进行垃圾收集或者进行压缩。”但对于`HotSpotJVM`而言，方法区还有一个别名叫做`Non-Heap`（非堆），目的就是要和堆分开。

所以，方法区看作是一块独立于`Java`堆的内存空间。

**内存布局**

![1608119848663](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/16/195730-622595.png)

1. 方法区主要存放的是 **`Class`**，而堆中主要存放的是**实例化的对象。**
2. 方法区（`Method Area`）与`Java`堆一样，是各个线程**共享的内存区域**。
3. 方法区在`JVM`启动的时候被创建，并且它的实际的物理内存空间中和`Java`堆区一样都可以是不连续的。
4. 方法区的大小，跟堆空间一样，可以选择固定大小或者可扩展。 
5. 方法区的大小决定了系统可以保存多少个类，如果系统定义了太多的类，导致方法区溢出，虚拟机同样会抛出内存溢出错误：`ava.lang.OutofMemoryError：PermGen space(jdk8之前) 或者java.lang.OutOfMemoryError:Metaspace（jdk8之后）`
   1. 加载大量的第三方的`jar`包。
   2. `Tomcat`部署的工程过多（30~50个）。
   3. 大量动态的生成反射类。
6. 关闭`JVM`就会释放这个区域的内存。

### Hotspot虚拟机中方法区的严谨

1. 在`jdk7`及以前，习惯上把方法区，称为**永久代**。`jdk8`开始，使用**元空间**取代了永久代。可以把方法区看做是一个接口，而元空间和永久代是虚拟机接口的不同实现。
2. `JDK 1.8`后，元空间存放在堆外内存中（也就是实际是**本地内存**）,`jdk7`还是放在堆中实现。
3. 本质上，方法区和永久代并不等价（对于其他的虚拟机）。仅是对`hotspot`而言的（可以认为是等价的）。《`Java`虚拟机规范》对如何实现方法区，不做统一要求。例如：`BEAJRockit / IBM J9 `中不存在永久代的概念。
4. **现在来看，当年使用永久代，不是好的`idea`。导致`Java`程序更容易`oom`（超过`-XX:MaxPermsize`上限），因为永久代使用的是`java`虚拟机的内存，而元空间使用的是本地内存，也就是说元空间的大小理论上可以增至本地内存的大小。**

**jdk8和jdk9的内存布局差异**

![1608120385037](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/16/200626-248046.png)

在`JDK8`，终于完全废弃了永久代的概念，改用与`JRockit、J9`一样在本地内存中实现的元空间（`Metaspace`）来代替，本地内存就是物理内存。

![1608120467041](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/16/200748-720583.png)

> 1. 元空间的本质和永久代类似，都是对`JVM`规范中方法区的实现。不过元空间与永久代最大的区别在于：元空间不在虚拟机设置的内存中，而是使用本地内存，本地内存一般都很大。
> 2. 永久代、元空间二者并不只是名字变了，内部结构也调整了。
> 3. 根据《`Java`虚拟机规范》的规定，如果方法区无法满足新的内存分配需求时，将抛出`OOM`异常，超出本地内存上限的话报出异常。

## 设置方法区的大小和oom

> 方法区的大小不必是固定的，`JVM`可以根据应用的需要动态调整，有固定大小和动态两种。

### JDK7及以前

1. 通过`-xx:Permsize=size`来设置永久代初始分配空间。默认值是`20.75M`
2. `-XX:MaxPermsize=size`来设定永久代最大可分配空间。32位机器默认是`64M`，64位机器模式是`82M`
3. 当`JVM`加载的类信息容量超过了这个值，会报异常`OutofMemoryError:PermGen space`。

**命令行查看内存大小**

![1608120838851](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/16/201400-819419.png)

如果在`jdk8`的环境中使用`jdk7`的内存设置参数的话，会报错，因为`jdk8`之后废除`jdk7`中内存的设置参数。

### JDK8之后

1. 元数据区大小可以使用参数` -XX:MetaspaceSize=size `和 `-XX:MaxMetaspaceSize=size`指定
2. 默认值依赖于平台。`windows`下，`-XX:MetaspaceSize`是`21M`，`-XX:MaxMetaspaceSize`的值是`-1`，即没有限制，物理内存多大，元空间理论上就可以多大。
3. 与永久代不同，如果不指定大小，默认情况下，虚拟机会耗尽所有的可用系统内存。如果元数据区发生溢出，虚拟机一样会抛出异常`OutOfMemoryError:Metaspace`。
4. `-XX:MetaspaceSize=size`：设置初始的元空间大小。对于一个64位的服务器端`JVM`来说，其默认的-`xx:MetaspaceSize`值为`21MB`。这就是初始的高水位线，一旦触及这个水位线，`Ful1GC`将会被触发并卸载没用的类（即这些类对应的类加载器不再存活）然后这个高水位线将会重置。新的高水位线的值取决于`GC`后释放了多少元空间。如果释放的空间不足，那么在不超过`MaxMetaspaceSize`时，适当提高该值。如果释放空间过多，则适当降低该值。
5. 如果初始化的高水位线设置过低，上述高水位线调整情况会发生很多次。通过垃圾回收器的日志可以观察到`Ful1GC`多次调用。为了避免频繁地`GC`，建议将`-XX:MetaspaceSize`设置为一个相对较高的值,上限我们一般不做设置，如果多次进行`GC`操作，开销很大，会降低虚拟机的运行效率。

### 如何解决OOM

**测试代码**

~~~ java
//代码：OOMTest 类继承 ClassLoader 类，获得 defineClass() 方法，可自己进行类的加载
/**
 * jdk6/7中：
 * -XX:PermSize=10m -XX:MaxPermSize=10m
 *
 * jdk8中：
 * -XX:MetaspaceSize=10m -XX:MaxMetaspaceSize=10m
 *
 */
public class OOMTest extends ClassLoader {
    public static void main(String[] args) {
        int j = 0;
        try {
            OOMTest test = new OOMTest();
            for (int i = 0; i < 10000; i++) {
                //创建ClassWriter对象，用于生成类的二进制字节码
                ClassWriter classWriter = new ClassWriter(0);
                //指明版本号，修饰符，类名，包名，父类，接口
                classWriter.visit(Opcodes.V1_8, Opcodes.ACC_PUBLIC, "Class" + i, null, "java/lang/Object", null);
                //返回byte[]
                byte[] code = classWriter.toByteArray();
                //类的加载
                test.defineClass("Class" + i, code, 0, code.length);//Class对象
                j++;
            }
        } finally {
            System.out.println(j);
        }
    }
}
~~~

**测试结果**

~~~ java
不设置元空间的上限
使用默认的 JVM 参数，元空间不设置上限，物理内存的大小。
输出结果：
10000

设置元空间的上限
JVM 参数，设置元空间的大小是10m。
-XX:MetaspaceSize=10m -XX:MaxMetaspaceSize=10m
输出结果：
8531
Exception in thread "main" java.lang.OutOfMemoryError: Metaspace
    at java.lang.ClassLoader.defineClass1(Native Method)
    at java.lang.ClassLoader.defineClass(ClassLoader.java:763)
    at java.lang.ClassLoader.defineClass(ClassLoader.java:642)
    at com.atguigu.java.OOMTest.main(OOMTest.java:29)
~~~

**关于`OOM`问题**

1. 要解决`OOM`异常或`heap space`的异常，一般的手段是首先通过内存映像分析工具（如`Ec1ipse
   Memory Analyzer`）对`dump`出来的堆转储快照进行分析，重点是确认内存中的对象是否是必要的，也就是要先分清楚到底是出现了内存泄漏（`Memory Leak`）还是内存溢出（`Memory Overflow`）。（内存泄漏：是内存中还有很多变量，也有引用指针指向这些对象，但是这些对象已经不使用了，而虚拟机的`GC`也不会去回收这些对象，所以会导致堆内存越来越小，而内存泄漏分析就是分析这类对象，删除其引用，让`GC`回收内存空间）。
2. **内存泄漏**就是有大量的引用指向某些对象，但是这些对象以后不会使用了，但是因为它们还和`GC ROOT`有关联，所以导致以后这些对象也不会被回收，这就是内存泄漏的问题。
3. 如果是内存泄漏，可进一步通过工具查看泄漏对象到`GC Roots`的引用链。于是就能找到泄漏对象是通过怎样的路径与`GC Roots`相关联并导致垃圾收集器无法自动回收它们的。掌握了泄漏对象的类型信息，以及`GC Roots`引用链的信息，就可以比较准确地定位出泄漏代码的位置。
4. 如果不存在内存泄漏，换句话说就是内存中的对象确实都还必须存活着，那就应当检查虚拟机的堆参数（`-Xmx`与`-Xms`），与机器物理内存对比看是否还可以调大，从代码上检查是否存在某些对象生命周期过长、持有状态时间过长的情况，尝试减少程序运行期的内存消耗。

## 方法区的内部结构

我们在把写好的程序经过编译后形成字节码文件，也就是`.class`文件结构，经过加载器加载到方法区之中，此时方法区不仅有`.class`文件中的内容，而且还有此类是哪个加载器加载的信息。（也就是**类加载器信息**）

**类加载过程**

![1608122271134](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/16/203752-526588.png)

《深入理解`Java`虚拟机》书中对方法区（`Method Area`）存储内容描述如下：它用于存储已被虚拟机加载的**类型信息、常量、静态变量、即时编译器编译后的代码缓存等。**

![1608122355375](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/16/203916-830441.png)

### 类型信息

在运行时方法区中，类信息中记录了哪个加载器加载了该类，同时类加载器也记录了它加载了哪些类

对每个加载的类型（类`class`、接口`interface`、枚举`enum`、注解`annotation`），`JVm`必须在方法区中存储以下类型信息：

> 1. 这个类型的完整有效名称（全名=包名.类名）
>
> 2. 这个类型直接父类的完整有效名（对于`interface`或是`java.lang.object`，都没有父类）
>
> 3. 这个类型的修饰符（`public`，`abstract`，`final`的某个子集）
>
> 4. 这个类型直接接口的一个有序列表

**存储的类的全类名**

~~~ java
//类型信息 ，以全类名形式存在。     
public class com.rzf.java.MethodInnerStrucTest extends java.lang.Object implements java.lang.Comparable<java.lang.String>, java.io.Serializable
~~~

### 域信息（成员变量）

> `descriptor`:` I` 表示字段类型为` Integer`
>
> `flags`: `ACC_PUBLIC `表示字段权限修饰符为` public`

**域信息存储的内容**

1. `JVM`必须在方法区中保存类型的所有域的相关信息以及域的声明顺序。
2. 域的相关信息包括：域名称、域类型、域修饰符（`public，private，protected，static，final，volatile，transient`的某个子集）

**域信息**

~~~ java
//域信息
  public int num;
    descriptor: I
    flags: ACC_PUBLIC

  private static java.lang.String str;
    descriptor: Ljava/lang/String;
    flags: ACC_PRIVATE, ACC_STATIC
~~~

### 方法（Method)信息

>1. descriptor: ()V` 表示方法返回值类型为` `void`
>2. `flags: ACC_PUBLIC`表示方法权限修饰符为` public`
>3. `stack=3 `表示操作数栈深度为 3
>4. `locals=2` 表示局部变量个数为 2 个（实例方法包含 `this`）
>5. `test1()` 方法虽然没有参数，但是其` args_size=1` ，这时因为将 `this `作为了参数

**代码说明**

~~~ java
public void test1();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=2, args_size=1
         0: bipush        20
         2: istore_1
         3: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
         6: new           #4                  // class java/lang/StringBuilder
         9: dup
        10: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
        13: ldc           #6                  // String count =
        15: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        18: iload_1
        19: invokevirtual #8                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
        22: invokevirtual #9                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        25: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        28: return
      LineNumberTable:
        line 17: 0
        line 18: 3
        line 19: 28
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      29     0  this   Lcom/atguigu/java/MethodInnerStrucTest;
            3      26     1 count   I
~~~

`JVM`必须保存所有方法的以下信息，同域信息一样包括声明顺序：

1. 方法名称
2. 方法的返回类型（或`void`）
3. 方法参数的数量和类型（按顺序）
4. 方法的修饰符（`public，private，protected，static，final，synchronized，native，abstract`的一个子集）
5. 方法的字节码（`bytecodes`）、操作数栈、局部变量表及大小（`abstract`和`native`方法除外）
6. 异常表（`abstract`和`native`方法除外）
   1. 每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引。

### Non-final的类变量

1. **静态变量和类关联在一起，随着类的加载而加载，他们成为类数据在逻辑上的一部分。**

2. 类变量被类的所有实例共享，即使没有类实例时，你也可以访问它

**代码说明**

~~~ java
/**
 * non-final的类变量
 *如下代码所示，即使我们把order设置为null，也不会出现空指针异常
这更加表明了 static 类型的字段和方法随着类的加载而加载，并不属于特定的类实例

 * @create: 2020-07-08-16:54
 */
public class MethodAreaTest {
    public static void main(String[] args) {
        Order order = new Order();
        order.hello();
        System.out.println(order.count);
    }
}
class Order {
    public static int count = 1;//类变量不会在编译期间直接赋值
    public static final int number = 2;//常量在编译的时候直接就已经进行赋值，常量
    public static void hello() {
        System.out.println("hello!");
    }
}
//如上代码所示，即使我们把order设置为null，也不会出现空指针异常,因为类中所有的方法和属性都是属于这一个类的
~~~

### 全局常量

1. 全局常量就是使用 `static final` 进行修饰

2. 被声明为`final`的类变量的处理方法则不同，每个全局常量在**编译的时候**就会被分配了。

**代码说明**

~~~ java
class Order {
    public static int count = 1;//静态变量
    public static final int number = 2;//常量
    ...
}    
~~~

**反编译结果**

~~~ java
public static int count;
    descriptor: I
    flags: ACC_PUBLIC, ACC_STATIC

public static final int number;
    descriptor: I
    flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL
    ConstantValue: int 2//常量的标识,已经进行赋值
//可以发现 staitc和final同时修饰的number 的值在编译上的时候已经写死在字节码文件中了，以后不可进行修改。
~~~

### 方法区案例测试

**代码说明**

~~~ java
**
 * 测试方法区的内部构成
 */
public class MethodInnerStrucTest extends Object implements Comparable<String>,Serializable {
    //属性
    public int num = 10;
    private static String str = "测试方法的内部结构";
    //构造器
    //方法
    public void test1(){
        int count = 20;
        System.out.println("count = " + count);
    }
    public static int test2(int cal){
        int result = 0;
        try {
            int value = 30;
            result = value / cal;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    @Override
    public int compareTo(String o) {
        return 0;
    }
}
~~~

**反编译结果**

~~~ java
//javap -v -p MethodInnerStrucTest.class > test.txt
//反编译字节码文件，并输出值文本文件中，便于查看。参数 -p 确保能查看 private 权限类型的字段或方法
Classfile /F:/IDEAWorkSpaceSourceCode/JVMDemo/out/production/chapter09/com/atguigu/java/MethodInnerStrucTest.class
  Last modified 2020-11-13; size 1626 bytes
  MD5 checksum 0d0fcb54854d4ce183063df985141ad0
  Compiled from "MethodInnerStrucTest.java"
//类型信息      
public class com.atguigu.java.MethodInnerStrucTest extends java.lang.Object implements java.lang.Comparable<java.lang.String>, java.io.Serializable
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #18.#52        // java/lang/Object."<init>":()V
   #2 = Fieldref           #17.#53        // com/atguigu/java/MethodInnerStrucTest.num:I
   #3 = Fieldref           #54.#55        // java/lang/System.out:Ljava/io/PrintStream;
   #4 = Class              #56            // java/lang/StringBuilder
   #5 = Methodref          #4.#52         // java/lang/StringBuilder."<init>":()V
   #6 = String             #57            // count =
   #7 = Methodref          #4.#58         // java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   #8 = Methodref          #4.#59         // java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
   #9 = Methodref          #4.#60         // java/lang/StringBuilder.toString:()Ljava/lang/String;
  #10 = Methodref          #61.#62        // java/io/PrintStream.println:(Ljava/lang/String;)V
  #11 = Class              #63            // java/lang/Exception
  #12 = Methodref          #11.#64        // java/lang/Exception.printStackTrace:()V
  #13 = Class              #65            // java/lang/String
  #14 = Methodref          #17.#66        // com/atguigu/java/MethodInnerStrucTest.compareTo:(Ljava/lang/String;)I
  #15 = String             #67            // 测试方法的内部结构
  #16 = Fieldref           #17.#68        // com/atguigu/java/MethodInnerStrucTest.str:Ljava/lang/String;
  #17 = Class              #69            // com/atguigu/java/MethodInnerStrucTest
  #18 = Class              #70            // java/lang/Object
  #19 = Class              #71            // java/lang/Comparable
  #20 = Class              #72            // java/io/Serializable
  #21 = Utf8               num
  #22 = Utf8               I
  #23 = Utf8               str
  #24 = Utf8               Ljava/lang/String;
  #25 = Utf8               <init>
  #26 = Utf8               ()V
  #27 = Utf8               Code
  #28 = Utf8               LineNumberTable
  #29 = Utf8               LocalVariableTable
  #30 = Utf8               this
  #31 = Utf8               Lcom/atguigu/java/MethodInnerStrucTest;
  #32 = Utf8               test1
  #33 = Utf8               count
  #34 = Utf8               test2
  #35 = Utf8               (I)I
  #36 = Utf8               value
  #37 = Utf8               e
  #38 = Utf8               Ljava/lang/Exception;
  #39 = Utf8               cal
  #40 = Utf8               result
  #41 = Utf8               StackMapTable
  #42 = Class              #63            // java/lang/Exception
  #43 = Utf8               compareTo
  #44 = Utf8               (Ljava/lang/String;)I
  #45 = Utf8               o
  #46 = Utf8               (Ljava/lang/Object;)I
  #47 = Utf8               <clinit>
  #48 = Utf8               Signature
  #49 = Utf8               Ljava/lang/Object;Ljava/lang/Comparable<Ljava/lang/String;>;Ljava/io/Serializable;
  #50 = Utf8               SourceFile
  #51 = Utf8               MethodInnerStrucTest.java
  #52 = NameAndType        #25:#26        // "<init>":()V
  #53 = NameAndType        #21:#22        // num:I
  #54 = Class              #73            // java/lang/System
  #55 = NameAndType        #74:#75        // out:Ljava/io/PrintStream;
  #56 = Utf8               java/lang/StringBuilder
  #57 = Utf8               count =
  #58 = NameAndType        #76:#77        // append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #59 = NameAndType        #76:#78        // append:(I)Ljava/lang/StringBuilder;
  #60 = NameAndType        #79:#80        // toString:()Ljava/lang/String;
  #61 = Class              #81            // java/io/PrintStream
  #62 = NameAndType        #82:#83        // println:(Ljava/lang/String;)V
  #63 = Utf8               java/lang/Exception
  #64 = NameAndType        #84:#26        // printStackTrace:()V
  #65 = Utf8               java/lang/String
  #66 = NameAndType        #43:#44        // compareTo:(Ljava/lang/String;)I
  #67 = Utf8               测试方法的内部结构
  #68 = NameAndType        #23:#24        // str:Ljava/lang/String;
  #69 = Utf8               com/atguigu/java/MethodInnerStrucTest
  #70 = Utf8               java/lang/Object
  #71 = Utf8               java/lang/Comparable
  #72 = Utf8               java/io/Serializable
  #73 = Utf8               java/lang/System
  #74 = Utf8               out
  #75 = Utf8               Ljava/io/PrintStream;
  #76 = Utf8               append
  #77 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #78 = Utf8               (I)Ljava/lang/StringBuilder;
  #79 = Utf8               toString
  #80 = Utf8               ()Ljava/lang/String;
  #81 = Utf8               java/io/PrintStream
  #82 = Utf8               println
  #83 = Utf8               (Ljava/lang/String;)V
  #84 = Utf8               printStackTrace
{
//域信息
  public int num;
    descriptor: I
    flags: ACC_PUBLIC

  private static java.lang.String str;
    descriptor: Ljava/lang/String;
    flags: ACC_PRIVATE, ACC_STATIC

  //方法信息
  public com.atguigu.java.MethodInnerStrucTest();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: bipush        10
         7: putfield      #2                  // Field num:I
        10: return
      LineNumberTable:
        line 10: 0
        line 12: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/atguigu/java/MethodInnerStrucTest;

  public void test1();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=2, args_size=1
         0: bipush        20
         2: istore_1
         3: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
         6: new           #4                  // class java/lang/StringBuilder
         9: dup
        10: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
        13: ldc           #6                  // String count =
        15: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        18: iload_1
        19: invokevirtual #8                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
        22: invokevirtual #9                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        25: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        28: return
      LineNumberTable:
        line 17: 0
        line 18: 3
        line 19: 28
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      29     0  this   Lcom/atguigu/java/MethodInnerStrucTest;
            3      26     1 count   I

  public static int test2(int);
    descriptor: (I)I
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: iconst_0
         1: istore_1
         2: bipush        30
         4: istore_2
         5: iload_2
         6: iload_0
         7: idiv
         8: istore_1
         9: goto          17
        12: astore_2
        13: aload_2
        14: invokevirtual #12                 // Method java/lang/Exception.printStackTrace:()V
        17: iload_1
        18: ireturn
      Exception table:
         from    to  target type
             2     9    12   Class java/lang/Exception
      LineNumberTable:
        line 21: 0
        line 23: 2
        line 24: 5
        line 27: 9
        line 25: 12
        line 26: 13
        line 28: 17
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            5       4     2 value   I
           13       4     2     e   Ljava/lang/Exception;
            0      19     0   cal   I
            2      17     1 result   I
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 12
          locals = [ int, int ]
          stack = [ class java/lang/Exception ]
        frame_type = 4 /* same */

  public int compareTo(java.lang.String);
    descriptor: (Ljava/lang/String;)I
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=2, args_size=2
         0: iconst_0
         1: ireturn
      LineNumberTable:
        line 33: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       2     0  this   Lcom/atguigu/java/MethodInnerStrucTest;
            0       2     1     o   Ljava/lang/String;

  public int compareTo(java.lang.Object);
    descriptor: (Ljava/lang/Object;)I
    flags: ACC_PUBLIC, ACC_BRIDGE, ACC_SYNTHETIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: checkcast     #13                 // class java/lang/String
         5: invokevirtual #14                 // Method compareTo:(Ljava/lang/String;)I
         8: ireturn
      LineNumberTable:
        line 10: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/atguigu/java/MethodInnerStrucTest;

  static {};
    descriptor: ()V
    flags: ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: ldc           #15                 // String 测试方法的内部结构
         2: putstatic     #16                 // Field str:Ljava/lang/String;
         5: return
      LineNumberTable:
        line 13: 0
}
Signature: #49                          // Ljava/lang/Object;Ljava/lang/Comparable<Ljava/lang/String;>;Ljava/io/Serializable;
SourceFile: "MethodInnerStrucTest.java"
~~~

### 运行时常量池 VS 常量池

**JVM运行时的内存结构**

![1608165023355](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/083026-664265.png)

1. 方法区，内部包含了**运行时常量池**，不是固定不变的，可以动态变化。

2. 字节码文件，内部包含了**常量池**，把字节码文件中的常量池加载到方法区中的结构叫做运行时常量池。**常量池对应于静态，而运行时常量池是动态的概念。**

3. **要弄清楚方法区，需要理解清楚`C1assFile`，因为加载类的信息都在方法区。**

4. 要弄清楚方法区的运行时常量池，需要理解清楚`classFile`中的常量池。

#### 常量池

一个有效的字节码文件中除了包含类的版本信息、字段、方法以及接口等描述符信息外，还包含一项信息就是常量池表（`Constant Pool Table`），包括各种字面量和对类型、域和方法的符号引用。

**为什么需要常量池**

一个`java`源文件中的类、接口，编译后产生一个字节码文件。而`Java`中的字节码需要数据支持，通常这种数据会很大以至于不能直接存到字节码里，换另一种方式，可以存到常量池，这个字节码包含了指向常量池的引用。在动态链接的时候会用到运行时常量池，之前有介绍。

比如：如下的代码：

~~~ java
public class SimpleClass {
    public void sayHello() {
        System.out.println("hello");
    }
}
~~~

虽然上述代码只有194字节，但是里面却使用了`String、System、PrintStream`及`Object`等结构。这里的代码量其实很少了，如果代码多的话，引用的结构将会更多，这里就需要用到常量池了。

**常量池中有什么**

1. 数量值

2. 字符串值

3. 类引用

4. 字段引用

5. 方法引用

**代码说明**

~~~ java
public class MethodAreaTest2 {
    public static void main(String args[]) {
        Object obj = new Object();
    }
}
~~~

**字节码**

~~~ java
new #2  
dup
invokespecial
~~~

- 其中以`#`开头的都是引用常量池中的常量。
- 小结：
  - 常量池、可以看做是一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量等类型

#### 运行时常量池

1. 运行时常量池（`Runtime Constant Pool`）是方法区的一部分。
2. 常量池表（`Constant Pool Table`）是`Class`文件的一部分，用于存放编译期生成的各种字面量与符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。
3. 运行时常量池，在加载类和接口到虚拟机后，就会创建对应的运行时常量池。
4. `JVM`为每个已加载的类型（类或接口）都维护一个常量池。池中的数据项像数组项一样，是通过索引访问的。
5. 运行时常量池中包含多种不同的常量，包括编译期就已经明确的数值字面量，也包括到运行期解析后才能够获得的方法或者字段引用。此时不再是常量池中的符号地址了，这里换为真实地址。
6. 运行时常量池，相对于`Class`文件常量池的另一重要特征是：**具备动态性。**
7. 运行时常量池类似于传统编程语言中的符号表（`symboltable`），但是它所包含的数据却比符号表要更加丰富一些。
8. 当创建类或接口的运行时常量池时，如果构造运行时常量池所需的内存空间超过了方法区所能提供的最大值，则`JVM`会抛`outofMemoryError`异常。

### 方法区应用举例

**代码说明**

~~~ java
public class MethodAreaDemo {
    public static void main(String[] args) {
        int x = 500;
        int y = 100;
        int a = x / y;
        int b = 50;
        System.out.println(a + b);
    }
}
~~~

**反编译结果**

~~~ java
public class com.atguigu.java1.MethodAreaDemo
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #5.#24         // java/lang/Object."<init>":()V
   #2 = Fieldref           #25.#26        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = Methodref          #27.#28        // java/io/PrintStream.println:(I)V
   #4 = Class              #29            // com/atguigu/java1/MethodAreaDemo
   #5 = Class              #30            // java/lang/Object
   #6 = Utf8               <init>
   #7 = Utf8               ()V
   #8 = Utf8               Code
   #9 = Utf8               LineNumberTable
  #10 = Utf8               LocalVariableTable
  #11 = Utf8               this
  #12 = Utf8               Lcom/atguigu/java1/MethodAreaDemo;
  #13 = Utf8               main
  #14 = Utf8               ([Ljava/lang/String;)V
  #15 = Utf8               args
  #16 = Utf8               [Ljava/lang/String;
  #17 = Utf8               x
  #18 = Utf8               I
  #19 = Utf8               y
  #20 = Utf8               a
  #21 = Utf8               b
  #22 = Utf8               SourceFile
  #23 = Utf8               MethodAreaDemo.java
  #24 = NameAndType        #6:#7          // "<init>":()V
  #25 = Class              #31            // java/lang/System
  #26 = NameAndType        #32:#33        // out:Ljava/io/PrintStream;
  #27 = Class              #34            // java/io/PrintStream
  #28 = NameAndType        #35:#36        // println:(I)V
  #29 = Utf8               com/atguigu/java1/MethodAreaDemo
  #30 = Utf8               java/lang/Object
  #31 = Utf8               java/lang/System
  #32 = Utf8               out
  #33 = Utf8               Ljava/io/PrintStream;
  #34 = Utf8               java/io/PrintStream
  #35 = Utf8               println
  #36 = Utf8               (I)V
{
  public com.atguigu.java1.MethodAreaDemo();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 7: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/atguigu/java1/MethodAreaDemo;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=5, args_size=1
         0: sipush        500
         3: istore_1
         4: bipush        100
         6: istore_2
         7: iload_1
         8: iload_2
         9: idiv
        10: istore_3
        11: bipush        50
        13: istore        4
        15: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        18: iload_3
        19: iload         4
        21: iadd
        22: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
        25: return
      LineNumberTable:
        line 9: 0
        line 10: 4
        line 11: 7
        line 12: 11
        line 13: 15
        line 14: 25
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      26     0  args   [Ljava/lang/String;
            4      22     1     x   I
            7      19     2     y   I
           11      15     3     a   I
           15      11     4     b   I
}
SourceFile: "MethodAreaDemo.java"
~~~

**图解字节码执行的过程**

1. 运行时数据区初始状态

![1608166299294](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/085141-625038.png)

2. 第一条指令`sipush`表示将操作数500入栈操作，入栈入的是`java`虚拟机栈，然后操作数 500 从操作数栈中取出，存储到局部变量表中索引为 1 的位置，在把数据存储到局部变量表中对应的指令是：`istore_1`，局部变量表是每一个方法存储参数，局部变量的位置。

![1608166344232](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/085230-294199.png)

3. 数据100入栈操作，然后在存储到局部变量表中。

![1608166710516](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/085832-478606.png)

![1608166719809](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/085841-141032.png)

4. 接着读取局部变量表中的数据，做出栈操作。

![1608166823907](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/090025-169656.png)

![1608166832353](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/090033-431011.png)

5. 把操作数栈中的两个数值做除法操作，然后再把结果入栈操作。

![1608166887222](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/090129-199408.png)

6. 将结果存储到局部变量表中，然后在把50做入栈操作，紧接着在存储到局部变量表中

![1608167035371](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/090356-573925.png)

![1608167094189](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/090455-430429.png)

7. 图片写错了是#25和#26（获得`System`类）

![1608167150447](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/090552-469466.png)

8. 从局部变量表中取出元素5和50，做入栈操作。

![1608167197073](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/090640-450856.png)

![1608167207180](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/090701-515974.png)

9. 执行加法运算后，将计算结果放在操作数栈顶

![1608167254306](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/090735-322516.png)

10. 打印结果，结束方法。

![1608167275676](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/090801-423228.png)

![1608167293005](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/090815-440309.png)

**符号引用—>直接引用**

1. 上面代码调用 `System.out.println() `方法时，首先需要看看 `System `类有没有加载，再看看 `PrintStream `类有没有加载

2. 如果没有加载，则执行加载，执行时，将常量池中的符号引用（字面量）转换为运行时常量池的直接引用（真正的地址值）

## 方法区的演进细节

### 永久代的演进细节

首先明确：只有`Hotspot`才有永久代。`BEA JRockit、IBMJ9`等来说，是不存在永久代的概念的。原则上如何实现方法区属于虚拟机实现细节，不受《`Java`虚拟机规范》管束，并不要求统一。

| `DK1.6`及以前 有永久代 | 静态变量存储在永久代上                                       |
| ---------------------- | ------------------------------------------------------------ |
| `JDK1.7`               | 有永久代，但已经逐步 “去永久代”，**字符串常量池，静态变量移除，保存在堆中** |
| `JDK1.8`               | 无永久代，类型信息，字段，方法，常量保存在本地内存的元空间，但字符串常                     量池、静态变量仍然在堆中。 |

**JDK1.6**

**方法区由永久代实现**，使用 `JVM` 虚拟机内存（虚拟的内存），**没有在堆中实现**。

![1608168012224](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/092014-399104.png)

**JDK1.7**

方法区由永久代实现，使用 `JVM `虚拟机内存，**但是实现仍然是在堆中**。

![1608168057868](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/092059-462918.png)

`JDK1.8`

方法区由元空间实现，**使用物理机本地内存**。

![1608168100545](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/092141-847356.png)

>三代`jdk`方法区所处的位置
>
>`jdk1.6`：不在堆内存。
>
>`jdk1.7`：迁移到堆内存中。
>
>`jdk1.8`：迁移到本地内存中。

### 永久代为什么要被元空间所代替

- 随着`Java8`的到来，`HotSpot VM`中再也见不到永久代了。但是这并不意味着类的元数据信息也消失了。这些数据被移到了一个与堆不相连的本地内存区域，这个区域叫做元空间（`Metaspace`）。
- 由于类的元数据分配在本地内存中，元空间的最大可分配空间就是系统可用内存空间。
- 这项改动是很有必要的，原因有：

  - 为永久代设置空间大小是很难确定的。在某些场景下，如果**动态加载类过多**，容易产生`Perm`区的`OOM`。比如某个实际`Web`工程中，因为功能点比较多，在运行过程中，要不断动态加载很多类，经常出现致命错误。`Exception in thread 'dubbo client x.x connector' java.lang.OutOfMemoryError:PermGen space`而元空间和永久代之间最大的区别在于**：元空间并不在虚拟机中，而是使用本地内存。 因此，默认情况下，元空间的大小仅受本地内存限制。**
  - 对永久代进行调优是很困难的。方法区的垃圾收集主要回收两部分内容：**常量池中废弃的常量和不再用的类型(但是对类进行回收条件非常苛刻)**，**但是**对类进行判断回收的条件非常的苛刻，所以虚拟机的开销也就比较大。**方法区的调优主要是为了降低**`Full GC`，不用频繁的进行`Full GC`操作，会降低开销。
  - 有些人认为方法区（如`HotSpot`虚拟机中的元空间或者永久代）是没有垃圾收集行为的，其实不然。《`Java`虚拟机规范》对方法区的约束是非常宽松的，提到过可以不要求虚拟机在方法区中实现垃圾收集。事实上也确实有未实现或未能完整实现方法区类型卸载的收集器存在（如`JDK11`时期的`ZGC`收集器就不支持类卸载）。
  - 一般来说这个区域的回收效果比较难令人满意，尤其是类型的卸载，条件相当苛刻。但是这部分区域的回收有时又确实是必要的。以前`Sun`公司的`Bug`列表中，曾出现过的若干个严重的`Bug`就是由于低版本的`HotSpot`虚拟机对此区域未完全回收而导致内存泄漏。


### 字符串常量池

**字符串常量池 `StringTable` 为什么要调整位置？**

`JDK7`中将`StringTable`放到了堆空间中。因为永久代的回收效率很低，在`Full GC`的时候才会执行永久代的垃圾回收，而`Full GC`是老年代的空间不足、永久代不足时才会触发。

这就导致`StringTable`回收效率不高，而我们开发中会有大量的字符串被创建，回收效率低，导致永久代内存不足。放到堆里，能及时回收内存。

  ### 静态变量存放在哪里（类变量）

#### 对象实体存放在哪里

**代码说明**

~~~ java
/**
 * 结论：
 * 1、静态引用对应的对象实体(也就是这个new byte[1024 * 1024 * 100])始终都存在堆空间，
 * 2、只是那个变量(相当于下面的arr变量名，也就是对象的引用)在JDK6,JDK7,JDK8存放位置中有所变化
 *
 * jdk7：
 * -Xms200m -Xmx200m -XX:PermSize=300m -XX:MaxPermSize=300m -XX:+PrintGCDetails
 * jdk 8：
 * -Xms200m -Xmx200m -XX:MetaspaceSize=300m -XX:MaxMetaspaceSize=300m -XX:+PrintGCDetails
 */
public class StaticFieldTest {
    private static byte[] arr = new byte[1024 * 1024 * 100];//100MB

    public static void main(String[] args) {
        System.out.println(StaticFieldTest.arr);
    }
}
~~~

- JDK6环境下，存放在老年代

![1609068970322](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/27/193611-881154.png)

- JDK7环境下，存放在老年代。大对象直接放在老年代。

![1609069038162](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/27/193719-121825.png)

- JDK8环境，存放在老年代。

![1609069082063](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/27/193802-445296.png)

上面的静态对象，也就是new出来的对象的实体始终存放在堆空间中。只是对象的引用名字存放位置发生变化，在Jdk1.8中存放在堆空间中。

但是对象的引用名字在jdk8中存放在堆空间中。注意，这里是静态变量，非静态的变量存放在java虚拟机栈空间上，而静态对象的引用理论存放在堆空间的元空间上。应为静态变量不属于某一个方法，是属于类的，所以放在堆上。

在jdk8之前，静态变量（类变量）存放在方法区，具体来说就是永久代（永久代是方法区的实现），但JDK8之后就取消了“永久代”，取而代之的是“元空间”，永久代中的数据也进行了迁移，**静态成员变量迁移到了堆中**（方法区是JVM的规范，永久代是方法区的具体实现）。

> **注：**
>
> 静态成员变量、常量池由方法区迁移到堆(java虚拟机中的堆空间)
>
> 元空间在本地内存

**方法区：**JDK8之前，由永久代实现，主要存放类的信息、常量池、常量和静态变量，方法数据、方法代码等；JDK8之后，取消了永久代，提出了元空间，并且常量池、静态成员变量等迁移到了堆中；元空间不在虚拟机内存中，而是放在本地内存中。那么，方法区是不是就不属于虚拟机内存的一部分了？还是元空间只是方法区的一部分，还有一部分东西存放在方法区中？待了解。

#### 变量名存放在哪里

这个问题需要用JHSDB工具来进行分析，这个工具是JDK9开始自带的(JDK9以前没有)，在bin目录下可以找到

**代码说明**

~~~ java
package com.atguigu.java1;

/**
 * 《深入理解Java虚拟机》中的案例：
 * staticObj、instanceObj、localObj存放在哪里？
 */
public class StaticObjTest {
    static class Test {
        static ObjectHolder staticObj = new ObjectHolder();
        ObjectHolder instanceObj = new ObjectHolder();

        void foo() {
            ObjectHolder localObj = new ObjectHolder();
            System.out.println("done");
        }
    }

    private static class ObjectHolder {
    }

    public static void main(String[] args) {
        Test test = new StaticObjTest.Test();
        test.foo();
    }
}
~~~

- 上面new出来的三个对象，不管是静态变量，非静态或者是方法内部的对象，对象的实体都放在堆空间中，但是这三个对象的引用存放位置不同：
  - staticObj随着Test的类型信息存放在**方法区**
  - instanceObj随着Test的对象实例存放在**Java堆**（也就是成员变量的引用放在堆空间中）
  - localObject则是存放在foo()方法栈帧的**局部变量表中**。
- 测试发现：三个对象的数据在内存中的地址都落在Eden区范围内，所以结论：**只要是对象实例必然会在**Java堆中分配。

![1609070436991](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/27/200037-627575.png)

1. 0x00007f32c7800000(Eden区的起始地址) ---- 0x00007f32c7b50000(Eden区的终止地址) 

2. 可以发现三个变量都在这个范围内

3. 所以可以得到上面结论

接着，找到了一个引用该staticObj对象的地方，是在一个java.lang.Class的实例里，并且给出了这个实例的地址，通过Inspector查看该对象实例，可以清楚看到这确实是一个java.lang.Class类型的对象实例，里面有一个名为staticobj的实例字段：

![1609070514291](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/27/200203-71897.png)

从《Java虚拟机规范》所定义的概念模型来看，所有Class相关的信息都应该存放在方法区之中，但方法区该如何实现，《Java虚拟机规范》并未做出规定，这就成了一件允许不同虚拟机自己灵活把握的事情。JDK7及其以后版本的HotSpot虚拟机选择把静态变量与类型在Java语言一端的映射Class对象存放在一起，**存储于**Java堆之中，从我们的实验中也明确验证了这一点，而在hotspot中方法区的实现是在元空间，也就是本地内存当中。

### 方法区的垃圾回收

1. 有些人认为方法区（如Hotspot虚拟机中的元空间或者永久代）是没有垃圾收集行为的，其实不然。《Java虚拟机规范》对方法区的约束是非常宽松的，提到过可以不要求虚拟机在方法区中实现垃圾收集。事实上也确实有未实现或未能完整实现方法区**类型卸载**的收集器存在（如JDK11时期的ZGC收集器就不支持类卸载）。

2. 一般来说这个区域的回收效果比较难令人满意，尤其是类型的卸载，条件相当苛刻。但是这部分区域的回收有时又确实是必要的。以前sun公司的Bug列表中，曾出现过的若干个严重的Bug就是由于低版本的HotSpot虚拟机对此区域未完全回收而导致内存泄漏

3. 方法区的垃圾收集主要回收两部分内容：**常量池中废弃的常量和不再使用的类型**。
   1. 先来说说方法区内常量池之中主要存放的两大类常量：字面量和符号引用。字面量比较接近Java语言层次的常量概念，如文本字符串、被声明为final的常量值等。而符号引用则属于编译原理方面的概念，包括下面三类常量：
      1. 类和接口的全限定名
      2. 字段的名称和描述符
      3. 方法的名称和描述符
   2. HotSpot虚拟机对常量池的回收策略是很明确的，只要常量池中的常量没有被任何地方引用，就可以被回收。
   3. 回收废弃常量与回收Java堆中的对象非常类似。（关于常量的回收比较简单，重点是类的回收）

**下面也叫做类型卸载**

1. 判定一个常量是否“废弃”还是相对简单，而要判定一个类型是否属于“不再被使用的类”的条件就比较苛刻了。需要同时满足下面三个条件：
   1. 该类所有的实例都已经被回收，也就是Java堆中不存在该类及其任何派生子类的实例。
   2. 加载该类的类加载器已经被回收，这个条件除非是经过精心设计的可替换类加载器的场景，如OSGi、JSP的重加载等，否则通常是很难达成的。
   3. 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。
2. Java虚拟机被允许对满足上述三个条件的无用类进行回收，这里说的仅仅是“被允许”，而并不是和对象一样，没有引用了就必然会回收。关于是否要对类型进行回收，HotSpot虚拟机提供了`-Xnoclassgc`参数进行控制，还可以使用`-verbose:class` 以及 `-XX：+TraceClass-Loading`、`-XX：+TraceClassUnLoading`查看类加载和卸载信息
3. 在大量使用反射、动态代理、CGLib等字节码框架，动态生成JSP以及OSGi这类频繁自定义类加载器的场景中，通常都需要Java虚拟机具备类型卸载的能力，以保证不会对方法区造成过大的内存压力。

### 小结

![1609070784271](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/27/200626-167669.png)

线程私有的数据区：程序计数器，本地方法栈，虚拟机栈，其中虚拟机栈中是一个一个的栈帧，栈帧里面有方法返回值，本地变量表，操作数栈，运行时常来那个池，运行时常量池负责运行时的动态链接，线程共享的区域是方法区和堆空间，其中方法区在jdk1.8之后的实现用的是本地内存，主要存放的是类型信息和运行时常量池，而在堆中有划分为eden区域，幸存者0区和幸存者1区，以及老年代，新生代主要是minor gc,而老年代主要是major gc,对于堆的整个内存空间，使用的是full gc（也包括方法区）。

 