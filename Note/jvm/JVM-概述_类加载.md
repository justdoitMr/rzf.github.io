# JAVA虚拟机

* [JAVA虚拟机](#java虚拟机)
  * [概述](#概述)
    * [什么是jvm](#什么是jvm)
    * [java跨平台及原理](#java跨平台及原理)
    * [JVM的内部结构](#jvm的内部结构)
    * [三大商业虚拟机](#三大商业虚拟机)
    * [JVM所处的位置](#jvm所处的位置)
    * [JVM的体系结构](#jvm的体系结构)
    * [Java代码的执行流程](#java代码的执行流程)
    * [JVM的架构模型](#jvm的架构模型)
    * [JVM的生命周期](#jvm的生命周期)
  * [类加载子系统](#类加载子系统)
    * [内存的结构布局。](#内存的结构布局)
    * [类加载子系统的作用](#类加载子系统的作用)
    * [类加载器（class loader）](#类加载器class-loader)
    * [类的加载过程（狭义上的加载）](#类的加载过程狭义上的加载)
    * [类的链接](#类的链接)
      * [验证阶段（Verify）](#验证阶段verify)
      * [准备阶段(prepare)](#准备阶段prepare)
      * [解析阶段（Resolve)](#解析阶段resolve)
      * [初始化阶段](#初始化阶段)
    * [类加载器的分类](#类加载器的分类)
    * [加载器的介绍](#加载器的介绍)
    * [获取ClassLoader的途径](#获取classloader的途径)
    * [双亲委派机制](#双亲委派机制)
    * [沙箱安全机制](#沙箱安全机制)
    * [其他](#其他)

**`java`虚拟机总览图**

![1608200402791](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/182004-220651.png)

## 概述

### 什么是`jvm`

1. `JVM` 是 `java`虚拟机，是用来执行`java`字节码(二进制的形式)的虚拟计算机

2. `jvm`是运行在操作系统之上的，与硬件没有任何关系

### `java`跨平台及原理

![1608200537722](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/182219-317996.png)

1. 跨平台：由`Java`编写的程序可以在不同的操作系统上运行：一次编写，多处运行
2. 原理：编译之后的字节码文件和平台无关，需要在不同的操作系统上安装一个对应版本的虚拟机(`JVM`)

### JVM的内部结构

1. 类加载子系统。
2. 运行时数据区 [ 我们核心关注这里 的**栈、堆、方法区** ]
3. 执行引擎(一般都是**JIT编译器和解释器**共存)
   1. `JIT`编译器(主要影响性能)：编译执行； 一般热点数据会进行二次编译，将字节码指令变成机器指令。将机器指令放在方法区缓存
   2.  解释器(负责响应时间，他的响应时间很快)：逐行解释字节码。

**图解`jvm`内部结构**

![1608200722200](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/182523-923687.png)

### 三大商业虚拟机

**Sun HotSpot**

1. 提起HotSpot VM，相信所有Java程序员都知道，它是Sun JDK和OpenJDK中所带的虚拟机，也是目前使用范围最广的Java虚拟机
2. 在2006年的JavaOne大会上，Sun公司宣布最终会把Java开源，并在随后的一年，陆续将JDK的各个部分（其中当然也包括了HotSpot VM）在GPL协议下公开了源码， 并在此基础上建立了OpenJDK。这样，HotSpot VM便成为了Sun JDK和OpenJDK两个实现极度接近的JDK项目的共同虚拟机。
3. 在2008年和2009年，Oracle公司分别收购了BEA公司和Sun公司，这样Oracle就同时拥有了两款优秀的Java虚拟机：JRockit VM和HotSpot VM。 Oracle公司宣布在不久的将来（大约应在发布JDK 8的时候）会完成这两款虚拟机的整合工作，使之优势互补。 整合的方式大致上是在HotSpot的基础上，移植JRockit的优秀特性，譬如使用JRockit的垃圾回收器与MissionControl服务， 使用HotSpot的JIT编译器与混合的运行时系统
4. 从服务器，桌面，到移动端，嵌入式都有应用。

**BEA JRocket**

1. 专注于服务端应用(JRockit内部**不包含解析器**实现，**全部代码都靠即时编译器编译后执行**)。
2. Jrockit JVM 是世界上最快的jvm3. 2008年被oracle收购。
3. 专注于服务器端的应用。它不太关注程序的启动速度，因此JRockit内部不包含解析器的实现，全部代码都靠及时编译器编译之后执行。

**lBM J9**

1. 市场定位与hotspot接近，服务器端，桌面应用，嵌入式等。
2. 目前，是影响力最大的三大商业虚拟机之一。
3. 应用于IBM的各种Java产品，

 ### JVM所处的位置

![1608201032676](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/183034-174913.png)

### JVM的体系结构

> 类的装载：加载—>链接—>初始化

**JVM体系结构**

![1608201184468](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/183306-184250.png)

1. 入口是编译好的字节码文件（编译器前端）-->经过类加载子系统（将我们的字节码加载到内存当中，生成一个`class`对象，加载--->链接--->初始化）
2. 在内存中，多个对象共享内存的是**方法区和堆区**（多个线程共享区）。
3. `Java`虚拟机栈，本地方法栈，程序计数器每一个线程独有一份。
4. 执行引擎：解释器（解释运行），`jit`及时编译器（编译器后端），垃圾回收器三部分。

### Java代码的执行流程

![1608201526485](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/183848-125323.png)

**说明**

1. 高级语言翻译为机器指令，主要是由执行引擎完成的。
   1. 解释器（解释运行，把字节码翻译为机器指令，主要负责翻编译器性能）
   2. `jit`及时编译器（编译器后端，主要是把热点代码缓存起来，主要负责编译器性能）组成执行引擎。

### JVM的架构模型

> `Java`编译器输入的指令流基本上是基于栈的指令集架构，另一种指令集架构是基于寄存器的指令集架构。

**两种架构之间的区别**：

- 基于栈指令集的架构特点
  - 设计和实现更加简单，适用于资源受限的系统。
  - 避开了寄存器额分配难题，使用零地址指令分配方式。
  - 指令流中的指令大部分是零地址指令，其执行过程依赖于操作栈，指令集更小，编译器更加容易实现。
  - 不需要硬件支持，可移植性好，更好实现跨平台。

- 基于寄存器指令集的特点
  - 典型的应用是x86二进制指令集，比如传统的`pc`以及`android`的`davlik`虚拟机。
  -   **指令集架构完全依赖于硬件，可移植性差。**
  - 性能优秀和执行更加高效。
  - 花费更少的指令去完成一项任务。
  - 在大部分情况下，基于寄存器的指令集往往都以一地址指令，二地址指令，三地址指令为主，而基于栈结构的指令集则以零地址为主。

**小结**

基于栈的指令集：跨平台性，指令集小，指令多，执行性能比寄存器差。所以`java`的指令都是基于栈来设计的，主要是为了跨平台性。

**反编译class文件**

对`class`文件进行反编译，进入到存放`class`的文件，使用`javap -v class`文件名

~~~ java
public class TestClassCode {
    public static void main(String[] args) {
        int i=2;
        int j=3;
        int k=i+j;
    }
}//反编译后的代码
 Last modified 2020-10-24; size 478 bytes
  MD5 checksum 1dc7bb5adfb83d6c9183d59b11a03108
  Compiled from "TestClassCode.java"
public class rzf.qq.com.jvm.TestClassCode
  minor version: 0
  major version: 49
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool://常量池,需要加载到方法区的运行时常量池中
   #1 = Methodref          #3.#21         // java/lang/Object."<init>":()V
   #2 = Class              #22            // rzf/qq/com/jvm/TestClassCode
   #3 = Class              #23            // java/lang/Object
   #4 = Utf8               <init>
   #5 = Utf8               ()V
   #6 = Utf8               Code
   #7 = Utf8               LineNumberTable
   #8 = Utf8               LocalVariableTable
   #9 = Utf8               this
  #10 = Utf8               Lrzf/qq/com/jvm/TestClassCode;
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               args
  #14 = Utf8               [Ljava/lang/String;
  #15 = Utf8               i
  #16 = Utf8               I
  #17 = Utf8               j
  #18 = Utf8               k
  #19 = Utf8               SourceFile
  #20 = Utf8               TestClassCode.java
  #21 = NameAndType        #4:#5          // "<init>":()V
  #22 = Utf8               rzf/qq/com/jvm/TestClassCode
  #23 = Utf8               java/lang/Object
{
  public rzf.qq.com.jvm.TestClassCode();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lrzf/qq/com/jvm/TestClassCode;

  public static void main(java.lang.String[]);//main函数
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=4, args_size=1
         0: iconst_2//前面的数字代表pc寄存器的指令,相当于声明一个常量
         1: istore_1//把声明的常量进行保存到1的操作数栈
         2: iconst_3//声明常量3
         3: istore_2//保存到索引为2的操作数栈中
         4: iload_1//根据所以加载操作数栈中的数值
         5: iload_2
         6: iadd//加法指令
         7: istore_3//结果保存在索引是3的操作数栈中
         8: return
      LineNumberTable:
        line 5: 0
        line 6: 2
        line 7: 4
        line 8: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  args   [Ljava/lang/String;
            2       7     1     i   I
            4       5     2     j   I
            8       1     3     k   I
}
SourceFile: "TestClassCode.java"
~~~

### JVM的生命周期

1. 虚拟机的启动

   `Java`虚拟机的启动是通过引导类加载器`bootstrapclass loader`创建一个初始类`initail calss`来完成的，这个类是由虚拟机的具体实现指定的。

2. 虚拟机的执行

   - 一个运行着的`java`虚拟机有着清晰的任务，执行`java`程序。
   - 程序开始执行虚拟机就运行，程序结束虚拟机就结束运行。

   - 执行一个`java`程序的时候，真真正正的在执行的是一个*`java`*虚拟机进程。

3. 虚拟机的停止，有以下几种情况虚拟机会退出

   - 程序正常执行结束
   - 程序在运行过程中遇到错误或者异常而终止执行。
   - 由于操作系统发故障而导致虚拟机进程终止运行。
   - 某一个线程调用了`Runtime`类或者`system`类的`exit`方法，或者调用`runtime`类的`halt`方法，并且`java`安全管理器允许执行安全退出的方法。

## 类加载子系统

### 内存的结构布局。

![1608202776102](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/185937-45142.png)

**细分内存结构图**

![1608203177685](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/190619-310954.png)

  >加载字节码文件经过三个步骤：加载--->链接--->初始化
  >
  >方法区只有`hotspot`虚拟机有，另外两大虚拟机没有。

### 类加载子系统的作用

**类加载的过程**

![1608204032046](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/192057-422849.png)

**作用**

1. 类加载子系统负责从本地文件或者网络文件中加载`class`文件，`class`文件开头有特定的标识符
2. `classloader`负责`class`文件的加载，至于他是否可以运行，由执行引擎决定`execution engine`。
3. 加载的类信息存放在一块称为**方法区的内存空间**，除了类的信息外，方法区还会存放**运行时常量池信息，可能还包括字符串常量和数字常量（这一部分常量信息是class文件中常量池部分的内存映射）。**

4. 加载完成后，`java`虚拟机外部的二进制字节流就会按照虚拟机所设定的格式存储在方法区之中，方法区的数据格式存储完全是由具体的虚拟机实现而确定的。然后会在`java`的堆内存中生成一个`Class`对象，这个对下行作为程序访问方法区的类型数据的外部入口。

**类加载的过程**

![1608204252917](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/192413-726560.png)

![1608204203227](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1608204203227.png)

> 加载成功后在内存中就会有一个大的`class`类型对象

### 类加载器（class loader）

![1608204329239](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1608204329239.png)

1. `class file`存储于本地磁盘上面，可以理解为设计师画在纸上的模板，而最终这个模板在执行的时候要加载到`jvm`当中，根据这个模板实例化`n`多个一模一样的实例。
2. `class file`加载到`jvm`当中，`b`被称为`DNA`元素的模板，存储在**方法区**。
3. 在`.class--->jvm---->`最终成为元数据模板，此过程只要一个运输工具，**类装载器（`class loader`)**,扮演者一个快递员的角色。
4. 其中`class`文件加载到内存中是以二进制流的方式进行加载，一个对象通过`getclass()`方法还可以获取是哪一个类的对象。

### 类的加载过程（狭义上的加载）

**加载阶段（Loading）**

1. 通过一个类的权限定名，获取定义此类的二进制字节流。
2. 将这个字节流所代表的静态存储结构转化为方法区(`jdk7`前叫做永久代，之后叫做元数据空间，都是方法区的落地实现)的运行时数据结构(也就是运行时数据区）。
3. 在内存中生成一个代表这个类的`java.lang.class`对象，作为方法区这个类的各种数据结构的访问入口。

### 类的链接

#### 验证阶段（Verify）

1. 为什么要进行验证，因为从`java`语言的角度来看，写好的程序经过编译之后的字节码文件应该是没有什么问题的，但是从`jvm`角度来看，读取到的`class`文件可以从任意地方，这就导致可能读入危害`jvm`的字节码文件，所以需要进行验证阶段，检查字节码文件是否是安全的。

2. 目的在于确保`.class`文件的字节流中包含的信息符合当前的虚拟机的要求，保证被加载的正确性，不会危害虚拟机自身的安全。

3. 主要包括四种验证方式：
   - 文件格式验证,不同文件文件头不一样。
   - 元数据验证
   - 字节码验证
   - 符号引用验证

#### 准备阶段(prepare)

1. 为类变量（`static`修饰的变量）设置内存和并且设置该类变量的初始值，即0。

2. 这里不包含用`final`修饰的`static`（也就是常量）,因为`final`修饰的变量在**编译阶段**就已经分配空间了，准备阶段会显示初始化。

3. 这里不会为实例变量分配初始化(此时还没有创建对象），类变量会分配在方法区中，而实例变量会随着对象一起被分配到`java`的堆中。

#### 解析阶段（Resolve)

1. **将常量池中的符号引用转换为直接引用的过程。**

2. 解析操作往往会伴随着`jvm`在执行完初始化之后再执行。

3. 符号引用就是一组符号来描述所引用的目标，符号引用的字面量形式明确定义`java`的`class`文件中，直接引用就是直接指向目标的指针，相对偏移量或者一个间接定位到目标的句柄。
4. 解析动作主要针对类或者接口，字段，类方法，接口方法，方法类型等，对应常量池中的`constant_class_info,constant_fieldref_info,constant_Methodref_info`等。

#### 初始化阶段

1. 初始化方法就是执行类构造器方法`clinit()`过程

2. 此方法不需要自己定义，是`javac`编译器中自动收集类中所有的**类变量的赋值动作和静态代码块中的语句**合并而来的。如果没有这种操作，也就没有`clinit()`方法。
3. 构造方法中的指令按照源文件中出现的顺序执行。（也就是所有变量的赋值操作会按照文件中赋值顺序依次赋值，后面的赋值会覆盖前面的赋值）。

![1608205050218](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/193731-393645.png)

4. `clinit()`方法不同于类的构造器，构造器是虚拟机视角下的`init()`方法。
5. 如果该类具有父类，`jvm`会保证子类的`clinit()`方法执行前，父类的`clinit()`方法已经执行完毕。

![1608205194149](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/193955-435664.png)

6. 虚拟机必须保证一个类的`clinit()`方法在多线程下被同步加锁。

![1608205268662](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/194109-198914.png)

7. **如果类或者方法中没有给变量赋值或者静态代码块，那么就没有调用此方法，**对每一个类进行反编译都会产生一个`init()`方法，`init()`方法对应的就是类的构造器的方法。
8. `clinit()`方法对于一个类来说并不是必须的，如果一个类中没有**静态代码块或者类变量**，也就没有此方法，接口中不能有静态代码块，但是仍然有变量初始化的赋值操作，因此接口和类一样会生成`clinit()`方法，但是和类不同的是，执行接口的`clinit()`方法前不需要先执行其父类的`clinit()`方法，因为只有父类接口中的变量被使用的时候，才会被初始化，另外，接口的实现类在初始化的时候也不一定要执行接口的`clinit()`方法。
9. 可以从另一个角度去理解初始化阶段：初始化过程就是执行类构造器`clinit()`方法的过程，此方法并不是程序员写的，而是编译器自动生成的，此方法与类的构造函数不同，也就是`init()`方法，`clinit()`方法不需要显示的调用父类的构造器，但是构造方法必须调用父类的构造器。
10. 我们注意到如果没有静态变量`c`，那么字节码文件中就不会有`clinit`方法

![1608205522820](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/194523-213387.png)

### 类加载器的分类

**`java`支持两种类型的类加载器：**

1. 引导型类加载器（`bootstrap classloader`)
2. 自定义类加载器（`user-define-classloader`)
   1. 自定义类加载器是所有派生于抽象类`classloader`的类加载器。

**系统中类加载器的组织架构：**

![1608205694674](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/194815-660625.png)

> `extension class loader`和`system class loader`都属于用户自定义加载器，应为他们都是继承自`class loader`,也就是只要是继承`class loader`的加载器，都是用户自定义加载器。
>
> 对于用户自定义类来说：使用系统类加载器`AppClassLoader`进行加载
>
> `java`核心类库都是使用引导类加载器`BootStrapClassLoader`加载的

**类加载器的结构**

![1608205932652](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/195214-156935.png)

**代码说明**

~~~ java
public class ClassLoaderTest {
    public static void main(String[] args) {
        //获取系统类加载器
        ClassLoader classLoader=ClassLoader.getSystemClassLoader();
        //打印系统类加载器对象的引用地址：sun.misc.Launcher$AppClassLoader@18b4aac2
        System.out.println(classLoader);
        
        //获取其父类加载器，即扩展类加载器
        ClassLoader parentClassLoader=classLoader.getParent();
        System.out.println(parentClassLoader);

        //获取bootstraploader加载器
        ClassLoader parent = parentClassLoader.getParent();
        System.out.println(parent);
        
        //对用于自定义类来说，是有哪一个加载器加载的呢？
        ClassLoader classLoader1 = ClassLoaderTest.class.getClassLoader();
        System.out.println(classLoader1);
        
         //查看String有那个加载器加载
        ClassLoader classLoader2 = String.class.getClassLoader();
        System.out.println(classLoader2);
    }
}
sun.misc.Launcher$AppClassLoader@18b4aac2//系统类加载器，也叫作应用类加载器
sun.misc.Launcher$ExtClassLoader@4554617c//扩展类加载器
null
sun.misc.Launcher$AppClassLoader@18b4aac2//用户自定义类有系统加载器加载
null//String类目前的加载器为null,所以string也是有获取bootstraploader加载器类加载器进行加载，系统核心的类库全部是由获取bootstraploader核心加载器进行加载
//获取加载器是null的话，都是由引导类加载器加载的
~~~

### 加载器的介绍

**启动类加载器（引导类加载器 `Bootstrap ClassLoader`）**

- 这个类加载器是由`c&c++`语言实现的，嵌套在`java`虚拟机内部。
- 此加载器用来加载`java`的核心库（`JAVA_HOME/jre/lib/rt.jar`）下面的内容，用于提供`jvm`自身需要的类，
- 此加载器并不继承自`java.lang.classloader`,没有父类加载器。
- **加载扩展类和应用程序类加载器，并指定为他们的父类加载器。**
- 处于安全考虑，`bootstrap`加载器仅仅加载包名为`java,javax,sun`等开头的类。

**扩展类加载器(`extention classloader`)**

- `java`语言编写的加载器，由`sun.misc.launcher$EXTclassloader`实现。

- 派生于`classloader`抽象类。

- ==父类加载器是启动类加载器。==

- 从`java.ext.dirs`系统目录下加载类库，或者从`jdk`的安装目录`jre/lib/ext/`子目录下加载类库，如果用户创建的`jar`包放在此目录下面，也会自动有扩展类加载器进行加载。

**应用程序类加载器（系统类加载器`appclassloader`,`system class loader`)**

- `java`语言编写的加载器，由`sun.misc.launcher$Appclassloader`实现。

- 派生于`classloader`抽象类。

- ==父类加载器为扩展类加载器。==

- 他负责加载环境变量为`classpath`或者系统属性`java.class.path`指定路径下的类库。

- ==该类加载器是系统中默认的加载器，一般来说`java`应用程序的类都是由系统类加载器完成加载的.==

- 通过`classloader#getsystemclassloader`方法可以获取到该类加载器。

**用户自定义类加载器**

> 为什么要自定义类加载器？

- 隔离加载类

- 修改类加载方式。

- 扩展加载源。

- 防止源码泄露。

**自定义类加载器的步骤**

1. 开发人员可以通过继承`java.lang.classloader`的方式，实现自己的类加载器。

2. 在`jdk1.2`之前，在自定义类加载器时候，总会去继承`classloader`类并且重写`loaderClass()`方法，从而实现自定义类加载器，在`jdk1.2`之后，已不再建议用户去覆盖`loaderClass()`方法，而是把自定义类的加载逻辑写在`findclass()`方法中。

3. 在编写自定义类加载器时候，如果没有太过复杂的要求，可以直接继承`URLclassloader`类，这样可以避免自己去写`findclass()`方法，及其获取字节码流的方式，使自定义类加载器更加方便。

**虚拟机自带类加载器**

~~~ java
/**
 * 虚拟机自带加载器
 */
public class ClassLoaderTest1 {
    public static void main(String[] args) {
        System.out.println("********启动类加载器*********");
        URL[] urls = sun.misc.Launcher.getBootstrapClassPath().getURLs();
        //获取BootStrapClassLoader能够加载的api路径
        for (URL e:urls){
            System.out.println(e.toExternalForm());
        }

        //从上面的路径中随意选择一个类 看看他的类加载器是什么
        //Provider位于 /jdk1.8.0_171.jdk/Contents/Home/jre/lib/jsse.jar 下，引导类加载器加载它
        ClassLoader classLoader = Provider.class.getClassLoader();
        System.out.println(classLoader);//null

        System.out.println("********拓展类加载器********");
        String extDirs = System.getProperty("java.ext.dirs");
        for (String path : extDirs.split(";")){
            System.out.println(path);
        }
        //从上面的路径中随意选择一个类 看看他的类加载器是什么:拓展类加载器
        ClassLoader classLoader1 = CurveDB.class.getClassLoader();
        System.out.println(classLoader1);//sun.misc.Launcher$ExtClassLoader@4dc63996
    }
}
~~~

### 获取ClassLoader的途径

![1608209047137](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/17/204408-599338.png)

**代码说明**

~~~ java
	public class TestClassLoader {
    public static void main(String[] args) throws ClassNotFoundException {
//        获取类加载器的第一种方式
//        加载string到内存只是获取内存中一个大的Class对象，也就是String类结构信息
//        系统类使用启动类加载器，所以返回结果是null
        ClassLoader classLoader=Class.forName("java.lang.String").getClassLoader();
        System.out.println(classLoader);
//        第二种方式，获取当前线程上下文的加载器
        ClassLoader classLoader1=Thread.currentThread().getContextClassLoader();
        System.out.println(classLoader1);
//        第三种方式
        ClassLoader classLoader2=ClassLoader.getSystemClassLoader().getParent();
        System.out.println(classLoader2);
    }
}
~~~

### 双亲委派机制

> java`虚拟机对`class`字节码文件采用的是按需加载的方式，也就是说当需要该类的时候才会把该类的字节码文件加载到内存生成`class`对象，而且加载某一个类的`class`对象的时候，`java`虚拟机采用的是双亲委派机制，即把请求交给父类处理，它是一种任务委派模式。（如果父类处理不了请求，那么就在逐层向下，直到类加载为止）

1. 什么是双亲委派模型？
   - 如果一个类加载器收到类的加载请求，他并不会自己先去加载，而是把这个请求先委托给自己的父类去执行。
   - 如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终会到达最顶层的启动类加载器。
   - 如果父类加载器可以完成加载任务，就成功返回，倘若父类加载器无法完成此类的加载任务，子加载器才会尝试自己去加载，这就是双亲委派模型。
   - 比如说自己定义的类，本身由系统类加载器（`app`类加载器）进行加载，但是系统类加载器不会直接加载，先向上传递给扩展类加载器，扩展类加载器在向上传递给启动类加载器，因为启动类加载器没有父类加载器，所以他就尝试自己加载，但是发现自己不能加载，然后他就向下传递给扩展类加载器，但是扩展类加载器发现自己也不能加载，就在向下传递，最终由系统类加载器进行加载。（扩展类加载器和启动类加载器有自己的类的加载目录，不在此目录中，这两个加载器就不会加载）

**图解双亲委派机制**

![1608373878955](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/19/183120-128994.png)

**JDBC类加载举例**

![1608373926931](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/19/183208-440285.png)

某一个程序要用到`spi`接口，那么`spi`中一些核心的`jar`包由引导类加载器进行加载，而核心的`jar`包是一些接口，要具体加载一些实现类，并且是第三方的实现类，不属于核心的`api`可以利用反向委派机制，由系统类加载器进行加载第三方的一些类，这里的系统加载器实际上是线程的上下文类加载器。线程上下文加载器实际上是一些系统加载器。

**双亲委派机制的优势**

- 避免类的重复加载。（保证每一个类只有一个类加载器进行加载）
- 保护程序安全，防止核心`api`被篡改。（也就是说使用和系统一样的包名的话，如果在这个包下定义一个类，会报错，系统包名下的类是启动类加载器加载，但是系统类加载器在加载时会去找系统包名下的这个类，发现没有就会报错，也就是防止核心`api`被任意修改）
- 如图，虽然我们自定义了一个`java.lang`包下的`String`尝试覆盖核心类库中的`String`，但是由于双亲委派机制，启动加载器会加载`java`核心类库的`String`类（`BootStrap`启动类加载器只加载包名为`java、javax、sun`等开头的类），而核心类库中的`String`并没有`main`方法

![1608374129019](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/19/183529-364233.png)

**小结**

- 避免类的重复加载
- 保护程序安全，防止核心`API`被随意篡改 
  - 自定义类：`java.lang.String`
  - 自定义类：`java.lang.MeDsh`（`java.lang`包需要访问权限，阻止我们用包名自定义类）

![1608374251599](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/19/183733-421856.png)

### 沙箱安全机制

自定义`string`类，但是在加载自定义`string`类的时候会率先使用引导类型加载器进行加载，而引导类型加载器在加载的过程中会率先加载jdk自带的文件，（rt.jar包中的java/lang/String.jar），报错提示没有main方法，这就是应为加载的是（rt.jar包中的java/lang/String.jar）下的string类，这样可以保证对java核心源代码的保护，这就是沙箱安全机制。

### 其他

- 在`jvm`中标示两个`class`对象（说的是内存中大的`class`对象，也就是类结构）是否为同一个类存在两个必要的条件：
  - 类的完整类名必须完全一致，包括包名。
  - 加载这个类的`classloader`也必须相同。（指的是`classloader`的实例对象）。
  - 换句话说，在`jvm`中，即使两个类对象（`class`对象），来源于同一个`class`文件，被同一个虚拟机加载，但是只要加载他们的`classloader`实例对象不同，那么这两个对象也不相等。
- `jvm`必须知道一个类型是由启动加载器加载的还是由用户类加载器加载的，如果一个类型是由用户类加载器进行加载，那么`jvm`会将这个类的加载器的一个引用作为类型信息的一部分保存在方法区中，当解析一个类型到另一个类型的引用的时候，`jvm`需要保证这两个类型的类加载器是相同的。
- `java`程序对类的使用方式分为:主动使用和被动使用
  - 主动使用：七中情况
    - 创建类的实例。
    - 访问某一个类或接口的静态变量，或者对该静态变量赋值。
    - 调用类的静态方法。
    - 反射机制（Class.forname(com.rzf.Test)）
    - 初始化一个类的子类。
    - java虚拟机启动时被标明为启动类的类。
    - jdk7开始提供的动态语言支持：
    - java.lang.invoke.MethodHandle实例的解析结果。
  - 除了以上七中情况，其他使用java类的方式都被看作是类的被动使用，都不会导致类的初始化。