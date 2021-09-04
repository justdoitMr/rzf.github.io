# 运行时数据区

- [概述](#概述)
- [线程](#线程)
  - [线程概念](#线程概念)
  - [线程的分类](#线程的分类)
- [程序计数器](#程序计数器)
  - [程序寄存器的介绍](#程序寄存器的介绍)
  - [PC寄存器的作用](#PC寄存器的作用)
  - [PC寄存器举例](#PC寄存器的举例)
  - [PC寄存器面试题](#PC寄存器面试题)
- JAVA虚拟机栈
  - [虚拟机栈概述](#虚拟机栈概述)
  - [栈的存储单位](#栈的存储单位)
  - [局部变量表](#局部变量表)
  - [操作数栈](#操作数栈)
  - [代码追踪](#代码追踪)
  - [栈顶缓存技术](#栈顶缓存技术)
  - [动态链接](#动态链接)(指向运行时常量池的方法引用)
  - [方法的调用，解析与分派](#方法的调用，解析与分派)
  - [方法返回地址](#方法返回地址)
  - [一些附加信息](#一些附加信息)
  - [栈相关面试题](#栈相关面试题)
- [本地方法栈](#本地方法栈)
- [本地方法接口](#本地方法接口)

## 概述

![1630668140141](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/192220-292784.png)

我们的字节码文件，经过类加载器的加载（**加载，链接，初始化**）后，在内存的方法区中，就保存我们的运行实例的本身，也就是类的各种信息。然后执行引擎使用我们的运行时数据区去执行程序。

![1630668206136](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/192327-441319.png)

内存是非常重要的系统资源，是硬盘和cpu的中间仓库及桥梁，承载着操作系统和应用程序的实时运行，JVM内存布局规定了java在运行过程中内存的申请，分配，管理的策略，保证了jvm的高效稳定的运行，*不同的*jvm*对内存的划分和管理机制存在着部分的差异*，上面图是hotspot虚拟机内存的划分情况。

下面是更加细致的内存划分情况：

![1630668395914](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/192637-629276.png)

在jdk8以前，方法区叫做元数据区，也可以说是非堆空间。

java虚拟机定义了若干种程序运行期间会使用到的运行时数据取，其中有一些会随着虚拟机的启动而创建，随着虚拟机的推出而销毁，另外一些区域则是与线程一一对应的，这些与线程对应的数据区域会随着线程的开始和结束而创建和销毁。

下年展示内存各个部分的共享情况：

![1630668452079](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/192750-455574.png)

这里面红色区域是每一个进程一份（也就是说多个线程共享的数据区），也就是说一个虚拟机实例一份，而我们的灰色区域，则是一个线程对应于一份。

- 线程独有：程序计数器，虚拟机栈，本地方法栈。

- 线程共享：堆，堆外内存（永久带或者元空间，代码缓存）、

下面这张图展示上面的情况：

![1630668554511](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/192916-748798.png)

对于Runtime类：一个jvm实例就对应着一个Runtime实例，一个Runtime对象就相当于一个运行时数据区。

![1630668585327](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/192946-452772.png)

> 本地方法栈区域没有垃圾回收机制，堆和方法区有垃圾回收机制，pc寄存器也没有垃圾回收机制
>
> Oom:内存溢出异常，pc寄存器没有内存溢出情况，但是虚拟机栈区域和本地方法栈区域可能有内存溢出情况，heap area,method area也可能发生内存的溢出情况。
>
> Pc寄存器既没有GC也没有OOM。
>
> 

## 线程

### 线程概念

- 线程是一个程序里面的运行单元，jvm允许一个应用程序有多个线程并行执行。

- 在hotspot虚拟机里，每一个线程都与操作系统的本地线程直接映射，每当一个java线程准备好执行以后，此时一个操作系统的本地线程也同时创建，java线程执行终止后，操作系统的本地线程也会被回收。

- 操作系统负责所有线程的安排调度到任何一个可用的cpu上，一旦本线程初始化成功，就会调用java线程中的run`()`方法。在这里准备工作是准备一个线程所需要的资源，比如本地方法栈，程序计数器，虚拟机栈。

### 线程的分类

如果使用jconsole或者任何一个调试工具，可以看到后台有许多线程在运行，这些线程不包括调用public
static void main(String []args)的main线程以及所有这个main线程自己创建的线程。（**线程分为普通线程和守护线程**）

- 这些主要的后台系统线程在hotspot虚拟机中是以下几个：
  - 虚拟机线程：这种线程的操作是需要jvm达到安全点才会出现，这些操作必须在不同的线程中发生的原因是他们需要jvm达到安全点，这样堆才不会变化，这种线程的执行类型不包括stop-the-world的垃圾收集，线程栈手机，线程挂起以及偏向锁撤销。
  - 周期任务线程：这种线程是时间周期事件的体现，他们一般用于周期性操作的调度执行。
  - GC线程：这种线程对jvm里面不同种类的垃圾收集行为提供了支持。
  - 编译线程：这种线程在运行时会将字节码编译成本地代码。
  - 信号调度线程：这种线程接收信号并发送给jvm，在他内部通过调用适当的方法进行处理。（守护线程）

## 程序计数器（pc寄存器）

![1630669019368](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/193701-566949.png)

### 程序寄存器的介绍

- 寄存器是程序控制流程的指示器，分支，循环，跳转，异常处理，线程恢复等基本功能都需要依赖这个寄存器来完成。

- 字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令。

- 寄存器是唯一一个在java虚拟机规范中没有规定任何oom情况和垃圾回收的区域。

- 寄存器是一块很小的空间，几乎可以忽略不计，也是运行最快的存储区域。

- 在jvm规范中，每一个线程都有自己的程序计数器，是线程私有的，生命周期与线程的生命周期保持一致。（也就是用来记录当前线程执行到的位置—）

- 任何时间一个线程都只有一个方法在执行，也就是所谓的*当前方法*，程序计数器会存储当前线程正在执行的java方法的jvm指令地址，或者是在执行native方法（本地方法），则是未指定值（undefined）。

- jvm中的程序计数寄存器中，register的名字源于cpu寄存器，寄存器存储指令相关的现场信息，cpu只有把数据装在到寄存器才能够运行。

- 在这里并不是广义上的物理寄存器，或许将其翻译成pc计数器，（或者指令计数器），jvm中的pc寄存器是对物理寄存器的一种抽象模拟。

### PC寄存器的作用

- pc寄存器用来存储指向下一条指令的地址，也即将要执行的指令代码，由执行引擎在寄存器中读取下一条指令地址并且执行指令。

- 在这里，程序计数器每一个线程都有一份，每一个栈帧就对应着一个方法，程序计数器就是记录自己的线程执行到那个位置。

下图说明寄存器在程序运行过程中的

![1630669259970](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/194100-535025.png)

### PC寄存器举例

**代码展示**

~~~ java
public class TestPcRegister {
    public static void main(String[] args) {
        int i=10;
        int j=20;
        int k=i+j;
    }
}
Classfile /D:/intellij/ideaWork/jv/target/classes/rzf/qq/com/jvm/TestPcRegister.class
  Last modified 2020-10-25; size 483 bytes
  MD5 checksum ffad765f0769b0cabdc305cd09685a89
  Compiled from "TestPcRegister.java"
public class rzf.qq.com.jvm.TestPcRegister
  minor version: 0
  major version: 49
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool://常量池
   #1 = Methodref          #3.#21         // java/lang/Object."<init>":()V
   #2 = Class              #22            // rzf/qq/com/jvm/TestPcRegister
   #3 = Class              #23            // java/lang/Object
   #4 = Utf8               <init>
   #5 = Utf8               ()V
   #6 = Utf8               Code
   #7 = Utf8               LineNumberTable
   #8 = Utf8               LocalVariableTable
   #9 = Utf8               this
  #10 = Utf8               Lrzf/qq/com/jvm/TestPcRegister;
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               args
  #14 = Utf8               [Ljava/lang/String;
  #15 = Utf8               i
  #16 = Utf8               I
  #17 = Utf8               j
  #18 = Utf8               k
  #19 = Utf8               SourceFile
  #20 = Utf8               TestPcRegister.java
  #21 = NameAndType        #4:#5          // "<init>":()V
  #22 = Utf8               rzf/qq/com/jvm/TestPcRegister
  #23 = Utf8               java/lang/Object
{
  public rzf.qq.com.jvm.TestPcRegister();
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
            0       5     0  this   Lrzf/qq/com/jvm/TestPcRegister;

  public static void main(java.lang.String[]);//main方法
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=4, args_size=1
//左边的序号，就是偏移地址，也就是我们寄存器中存储的地址
         0: bipush        10 //定义变量10
         2: istore_1 //存储到索引为1的位置
         3: bipush        20
         5: istore_2 // 存储到索引是2的地方
         6: iload_1 //取出索引为1出的数值
         7: iload_2//取出索引为2处的数值
         8: iadd
         9: istore_3 //结果存储到索引为3的位置
        10: return
      LineNumberTable:
        line 5: 0
        line 6: 3
        line 7: 6
        line 8: 10
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  args   [Ljava/lang/String;
            3       8     1     i   I
            6       5     2     j   I
           10       1     3     k   I
}
~~~

**图示**

![1630669369144](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/194250-575546.png)

### PC寄存器面试题

使用pc寄存器存储字节码指令地址有什么用？

- 因为cpu需要不停的在各个线程之间进行切换，这个时候切换回来之后，就不知道从程序的哪一个指令开始，jvm的字节码解释器就需要通过改变pc寄存器的值来明确下一条执行什么样的字节码指令。

pc寄存器为什么会设定为线程私有的？

- 所谓的多线程是在一个特定的时间段内只会执行某一个线程的方法，cpu会不停的做任务的切换，这样必然导致经常的中断和恢复，如何保证在这个过程中不出现差错，为了能够准确的记录各个线程正在执行的当前字节码指令的地址，最好办法是为每一个线程都分配一个pc寄存器，这样一来各个线程之间便可以进行独立计算，从而不会出现干扰的情况。

- 由于cpu时间片轮限制，多个线程在执行的过程中，任何一个确定的时刻，一个处理器或者多核处理器中的一个内核，只会执行某一个线程中的一条指令。这样必然导致经常的中断和恢复，如何保证在这个过程中不出现差错，为了能够准确的记录各个线程正在执行的当前字节码指令的地址，最好办法是为每一个线程都分配一个pc寄存器，这样一来各个线程之间便可以进行独立计算，从而不会出现干扰的情况。

**补充**

并行vs串行

- 并行，若干个线程同时在执行，在一个时间点上同时执行，串行是若干个线程按照顺序执行

- 并发，一个cpu快速在各个线程之间进行切换执行.

## 虚拟机栈（java虚拟机栈）

### 虚拟机栈概述

**栈出现的背景**

- 由于跨平台性设计，java的指令都是根据栈设计的，不同平台的cpu架构不同，所以不能设计为基于寄存器的。
- 优点是跨平台，指令集比较小（8位对齐），编译器容易实现，缺点是性能下降，实现同样的功能需要更多的指令。刚好和寄存器指令相反。

**内存中的栈与堆**

- 栈是运行时的单位，而堆是存储的单位，*即栈解决的是程序的运行问题，程序如何执行，或者说如何处理数据，堆解决的是如何存储数据，即数据怎么存放，放在哪里。*

**java虚拟机栈是什么**

- java虚拟机栈，早期也叫做java栈，**每个线程在创建的时候都会创建一个虚拟机栈，其内部都会保存一个个的栈帧，对应着一次次的方法调用，栈是线程私有的。栈帧是栈的基本单位。**如下图所示：

![1630669926456](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/195207-220342.png)

**生命周期**

- 栈的生命周期和线程的生命周期一致，随着线程的启动而创建，随着线程的停止而销毁。当主线程结束后，整个的虚拟机的栈就全部销毁掉。

**作用**

- 主管java程序的运行，他保存方法的局部变量（8种基本数据类型（局部变量），引用数据类型只能放引用数据对象的地址（引用类型变量，属性），真正new出来的对象存放在堆空间中），部分结果，并且参与方法的调用和结果的返回。

**栈的特点（优点）**

- 栈是一种快速有效的分配存储方式，访问速度仅次于程序计数器，

- jvm直接对java栈的操作有两个，每个方法的执行，伴随着出栈入栈。运行结束后出栈工作。

- 对于栈来说，不存在垃圾回收机制，Pc寄存器：既没有垃圾回收，也没有oom内存溢出。栈：存在oom，会栈溢出，但是不存在垃圾回收机制。

**栈中可能出现的异常**

- java虚拟机规范允许java栈的大小是动态变化的或者是固定不变的，

- 如果是固定大小的虚拟机栈，那么每一个线程的java虚拟机栈的大小可以再创建线程的时候独立确定，如果线程请求分配的栈容量大小超过虚拟机栈允许的最大容量，java虚拟机将会抛出StackOverFlowError异常。

- 如果java虚拟机栈的容量可以进行动态的扩展，并且在扩展的时候无法申请到足够的空间，或者在新创建线程的时候没有足够的内存去创建虚拟机栈，那么java虚拟机将会抛出一个OutOFmemoryError错误。

**设置内存的大小。**

- 我们可以使用参数-Xss来设置线程的最大栈空间容量，栈的大小直接决定了函数调用最大的深度。

### 栈的存储单位

**栈的内部结构**

![1630670295209](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/195816-509250.png)

注：每一个栈帧代表着一个方法的调用。

**栈中存储什么？**

- 每一个线程都有自己的栈，栈中的数据都是以栈帧(satck frame)的格式存在，可以把每一个栈帧看做是一个方法（方法和栈帧是一一对应的关系）。

- 在这个线程上正在执行的每一个方法都对应着一个栈帧。

- 栈帧是一个内存块，是一个数据集，维系着方法执行过程中的各种数据信息。

**栈的运行原理**

- jvm对栈的操作有两个，出栈和入栈，遵循“先进后出，后进先出”的原则。

- 在一条活动的线程中，一个时间点上，只有一个活动的栈帧，即只有当前正在执行的方法的栈帧（栈顶栈帧）是有效的，这个栈帧被称为当前栈帧（current frame）,与当前栈帧对应的方法是当前方法（current method）,当以这个方法的类就是当前类（current class）。

- 执行引擎运行的所有字节码指令只针对当前当前的栈帧进行操作。

- 如果在该方法中调用其他方法，对应的新的栈帧会被创建出来，放在栈的顶端，称为新的当前栈帧。

- 不同的线程所包含的栈帧是不允许相互引用的，即不可能在一个线程的栈帧中引用另外一个线程的栈帧。

- 如果当前方法调用了其他的方法，方法返回之前，当前栈帧会返回此方法的执行结果给前一个栈帧，接着，虚拟机会丢弃当前栈帧，使得前一个栈帧称为当前的栈帧。

- java方法有两种返回的方式，**一种是正常的函数返回，使用**return**命令，另一种是异常的抛出，不管使用哪种方式，都会导致栈帧被弹出。**

**栈帧的基本结构**

- 局部变量表（local variables**）**：变量的声明。

- 操作数栈(operand stack**)**:表达式栈。

- 方法返回地址(return address**)**：方法正常退出或者异常退出的定义。

- 动态链接(dynamic linking**)**：指向运行时常量池的方法引用。

-  一些附加信息。

栈帧的大小决定线程中的栈可以存放几个栈帧，而栈帧的大小又由以上的几部分决定。局部变量表影响最大,如下图所示：

![1630671052834](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/201053-4991.png)

### 局部变量表（👋 重点）

- 局部变量表也叫做局部变量数组或者本地变量表。（是一个一维数组）

- 定义为一个数字数组，主要用于存放**参数和定义在方法体中的局部变量**，这些数据类型包括各种基本数据类型（8中基本数据类型），对象的引用(reference)，以及returna ddress（返回值类型）类型。

- 由于局部变量表是建立在线程的栈上面，是线程的私有数据，**因此不存在数据安全问题**。（当多个线程共同操作共享数据会存在线程安全问题）
- **局部变量表的大小是在编译器编译期间确定下来的**，并且保存在方法的code属性的maximum local variables数据项中，在方法运行期间是不会改变局部变量表的大小的。
- 方法嵌套调用的次数由栈的大小所决定，一般来说，栈越大，方法嵌套调用次数就越多，对于一个函数而言，他的参数或者定义的变量越多，使得局部变量表膨胀，他的栈帧就越大，以满足方法调用所需传递信息增大的需求，进而函数调用会占据更多的栈空间，导致其嵌套调用次数就会减少。
- 局部变量表只在当前的方法调用中有效，在方法执行的时候，虚拟机会通过使用局部变量表完成参数值到参数变量列表的传递过程，当方法调用结束后，就会随着方法栈的销毁而不存在，局部变量表也会销毁。

**代码演示**

~~~ java
public class TestLocal {
    public static void main(String[] args) {
        int i=0;
        int num=10;
        String string=new String("abc");
    }
}
//对于上面代码的局部变量表，有4个元素，args,i,num,string四个变量
LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      16     0  args   [Ljava/lang/String;
            2      14     1     i   I
            5      11     2   num   I
           15       1     3 string   Ljava/lang/String;
Code:
      stack=3, locals=4, args_size=1
//Code的属性locals保存局部变量表最大的长度4。
~~~

**关于slot的理解**

1. 参数值的存放总是在局部变量表数组的index-0开始，到数组长度-1的索引结束。
2. 部变量表，最基本的存储单元是slot,(变量槽)。这也是局部变量表的基本单位。
3. 局部变量表存储编译期间可以确定的各种基本数据类型，引用类型，return address的返回类型。
4. 在局部变量表中，32位以内的变量只占用一个slot,包括return address类型，64位类型（long,double）占用两个slot。
   1. byte,short,char在存储前被转换为int,boolean也被转换为int,0标示false,非零标示true，这几种类型都占1个slot。
   2. long,double占据两个slot。
5. jvm会为局部变量表中的每一个slot分配一个索引，通过这个索引可以成功的访问到局部变量表中这个变量的值。
6. 当一个实例方法被调用的时候，他的方法参数和方法体内部定义的局部变量将会按照顺序（变量的声明顺序）复制到局部变量表的每一个slot上面。
7. 如果要访问局部变量表中一个64位的变量的值时，只需要使用前一个索引即可，即开始索引。
8. 如果当前帧是由**实例方法或者构造方法**创建的，那么该对象引用this将会存放在index为0 的slot处（代表调用该方法的那个对象，而构造器中的this代表当前正在创建的那个对象）。其余的参数按照参数顺序进行排列存放。static修饰的方法没有this。
9. 局部变量表中也存在this变量，在静态的方法中不可以用this（类调用静态方法），因为this不存在与当前方法的局部变量表中，构造方法，普通方法可以使用this是应为在他们的局部变量表中有this的声明，String也是引用类型，占据一个slot。
10. 局部变量表中也存在this变量，在静态的方法中不可以用this（类调用静态方法），因为this不存在与当前方法的局部变量表中，构造方法，普通方法可以使用this是应为在他们的局部变量表中有this的声明，String也是引用类型，占据一个slot。

**局部变量表的理解**

1. 我们创建的一个类经过编译，之后再反编译得到的类信息如下，本类中有6个方法：

![1630671770637](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/202251-434730.png)

2. 首先看main()静态方法：

![1630671825258](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/202346-267719.png)

3. Code信息

![1630671875112](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/202436-619719.png)

4. 局部变量表信息

![1630671920386](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/202521-485107.png)

注：在这里因为main()方法是静态方法，而静态方法是属于类的，所以静态方法的局部变量表中没有this变量。也就是说在静态方法中不可以使用this变量。

5. 构造方法的局部变量表

![1630671982802](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/202623-417654.png)

构造方法的局部变量表就相当于当前需要创建对象的本身。

6. 普通方法的局部变量表

![1630672022331](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/202703-179051.png)

Test01()方法的局部变量表中有this变量，所以在普通方法中可以使用this变量，还有两个局部变量a和b。

7. 带参数的方法

![1630672060247](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/202741-175167.png)

Test02()有两个形参，所以在局部变量表中有num1和num2两个形参的说明所以test02()的局部变量表长度是4。

8. Double和float类型

![1630672154213](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/202915-972664.png)

Test04()方法中有double类型的变量，其占据两个变量槽，所以序号是1和2，但是访问时只需要用到第一个序号即可，long类型也占两个变量槽。

注意：this变量会放在索引为0的变量槽处。静态方法除外。静态方法和非静态方法的在变量方面的区别就是非静态方法中有this变量，而今静态方法中没有，并且this变量在0的位置。

**Slot的重复利用问题**

栈中局部变量表中的槽位是可以重复利用的，如果一个变量过了其作用域，那么在其作用域后声明的新的局部变量就很有可能重复用过期的局部变量表中的槽位，从而达到节省资源的目的。

![1630672216523](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/203017-620267.png)

现在程序中有4个变量（args,a,b,c）,但是局部变量表的长度是3，因为变量b的作用域在大括号里面，所以变量c会占据变量b所开辟的数组空间。变量b,c的索引号都是2，变量b起始地址是4，长度是4（这里的长度指的是变量作用域的长度，在指令中对应的序号），变量c的起始是12，作用域长度是1。

变量b在出了大括号就被销毁，但是在数组中开辟的空间还在，因此变量c使用的是变量b开辟的slot位置。因此c的index和b的index数值上一样。

**补充**：静态变量和局部变量的对比

- 按照数据类型分类：基本数据类型，引用数据类型。
- 按照在类中声明的位置：
  - 成员变量，在使用前，默认都经历过初始化赋值，成员变量又分为类变量和实例变量。
    - 类变量：在linking和prepare阶段，给变量默认赋值，initial阶段：给类变量显示赋值，即静态代码块赋值，如果没有静态代码块显式赋值，直接使用默认值即可。
    - 实例变量：随着对象的创建，会在堆空间分配实例对象的空间，并进行默认赋值。
  - 局部变量：在使用前，必须进行显示赋值，否则编译不会通过。因为局部变量也是存储在局部变量表中，局部变量表是一个数组，数组需要初始化。

在栈帧中，与性能调优关系最为密切的部分就是前面提到的局部变量表，在方法执行时，虚拟机就使用局部变量表完成方法的传递。

局部变量表中也是最重要的垃圾回收根节点，只要被局部变量表中直接引用或者间接引用的对象都不会被回收。

### 操作数栈（👋 重点）

**概述**

1. 每一个独立的栈帧中除了包含局部变量表以外，还包括一个后进先出（last-in-first-out）的操作数栈，也可以称为表达式栈（expression stack）。

2. 操作数栈，在方法执行的过程中，根据字节码指令，往栈中写入数据或者是提取数据，即入栈或者出栈。

3. 某一些字节码指令将值压入操作数栈，其余的字节码指令将操作数取出栈，使用他们后再把结果压入栈。

4. 默认用数组实现，因此在编译阶段大小已经确定。

5. 如果被调用的方法带有返回值的话，其返回值将会被压入当前栈帧的操作数栈中，并且更新pc寄存器中吓一跳需要执行的字节码指令。

6. **操作数栈中元素的数据类型必须和字节码指令的序列严格匹配，这是由编译器的编译期间进行验证的，同时在类加载的过程中类检验阶段的数据流分析阶段要再次验证。**

**另外，我们说**java**虚拟机的解析引擎是基于栈的执行引擎，其中的栈指的是操作数栈**

**作用**

1. 操作数栈，主要用于保存计算过程中的中间结果，同时作为计算过程中变量的临时存储位置。
2. 操作数栈就是jvm执行引擎的一个工作区，当一个方法刚开始执行的时候，一个新的栈帧也会随之被创建出来，这个方法的操作数栈是空的。
3. 每一个操作数栈都会拥有一个明确的栈深度用于存储数值，其所需要的最大深度在编译期间就定义好，保存在方法的code属性中，为max_stack的值。
4. 栈中的数据类型可以是任何一个java数据类型。
   1. 32bit的类型占用一个栈单位深度。
   2. 64bit类型占据两个栈单位的深度。

5. 操作数栈并非采用访问索引的方式来进行数据访问的，而只是通过标准的入栈和出栈操作来完成数据的访问。

### 代码追踪

Init()方法是jvm为我们当前的类默认提供的构造器。

**代码展示**

~~~ java
public class OperantStackTest {
    public static void main(String[] args) {
        byte i=15;
        int j=8;
        int m=i+j;
    }
}
//编译结果
Classfile /D:/intellij/ideaWork/jvm/character01/target/classes/qq/com/rzf/OperantStackTest.class
  Last modified 2020-4-17; size 481 bytes
  MD5 checksum 775fd5fe9a4b1c367860391e1435daef
  Compiled from "OperantStackTest.java"
public class qq.com.rzf.OperantStackTest
  minor version: 0
  major version: 49
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #3.#21         // java/lang/Object."<init>":()V
   #2 = Class              #22            // qq/com/rzf/OperantStackTest
   #3 = Class              #23            // java/lang/Object
   #4 = Utf8               <init>
   #5 = Utf8               ()V
   #6 = Utf8               Code
   #7 = Utf8               LineNumberTable
   #8 = Utf8               LocalVariableTable
   #9 = Utf8               this
  #10 = Utf8               Lqq/com/rzf/OperantStackTest;
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               args
  #14 = Utf8               [Ljava/lang/String;
  #15 = Utf8               i
  #16 = Utf8               I
  #17 = Utf8               j
  #18 = Utf8               m
  #19 = Utf8               SourceFile
  #20 = Utf8               OperantStackTest.java
  #21 = NameAndType        #4:#5          // "<init>":()V
  #22 = Utf8               qq/com/rzf/OperantStackTest
  #23 = Utf8               java/lang/Object
{
  public qq.com.rzf.OperantStackTest();
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
            0       5     0  this   Lqq/com/rzf/OperantStackTest;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=4, args_size=1
         0: bipush        8
         2: istore_1
         3: bipush        9
         5: istore_2
         6: iload_1
         7: iload_2
         8: iadd
         9: istore_3
        10: return
      LineNumberTable:
        line 5: 0
        line 6: 3
        line 7: 6
        line 8: 10
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  args   [Ljava/lang/String;
            3       8     1     i   I
            6       5     2     j   I
           10       1     3     m   I
}
SourceFile: "OperantStackTest.java"
//byte,short,char,boolean像数组中存放都以int类型保存
~~~

局部变量表是从1开始的，因为0位置存储的是this指针。

1. 把15存储到局部变量表中

![1630672787504](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/04/152712-809113.png)

2. 存储8到局部变量表中

![1630672870910](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/204112-924220.png)

3. 计算结果存储到栈顶

![1630672915566](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/204156-73396.png)

4. 存储最终结果到局部变量表中

![1630672947183](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/03/204228-518629.png)

注：bipush:表示存储`byte`类型数据。Istore_1表示存储的是int类型的数据。

### 栈顶缓存技术

- 基于栈架构的虚拟机所使用的零地址更加紧凑，但是完成一项操作的时候必然需要使用更多的入栈和出栈指令，同时也就意味着将需要更多的指令分派次数和内存的读写次数。

- 由于操作数是存储在内存当中的，因此频繁的执行内存的读写操作必然会影响执行速度，为了解决这个问题，hotspot虚拟机的设计者们提出了栈顶缓存的技术，**将站顶的元素全部缓存在物理的**cpu**寄存器上，以此降低对内存的读写次数，提升执行引擎的执行效率。**

### 动态连接（指向运行时常量池的方法引用）

- 每一个栈帧的内部都包含一个指向运行时常量池中该帧所属方法的引用，包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接，比如invokedynamic指令。

- 在java源文件被编译到字节码文件中时，所有的变量和方法的引用都作为符号引用（symbolic references），符号引用即以#开头额引用，保存在class文件的常量池中，在描述一个方法调用另一个方法的时候，就是通过class文件常量池中指向方法的符号引用来标示的，那么动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用。

为什么需要常量池呢？

- 常量池的作用：就是为了提供一些符号常量，以便于指令的识别，节省存储空间。在class字节码文件中常量池中的符号引用被加载到内存时就会放到方法区的常量池中，然后栈帧中需要符号引用时就直接引用方法区中的符号引用。方法区的常量池称作运行时的常量池。动态连接的目的就是将这些符号的引用转换为调用方法的直接引用。也就是说方法区的常量池信息就是字节码文件中常量池的信息。如下图所示：

![1630715509062](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/04/152721-295083.png)

**代码示例**

~~~ java
public class DynamicLinking {
    private int num=10;
    public static void main(String[] args) {

    }
    public void methodA(){
        System.out.println("methodA()");
    }
    public void methodB(){
        System.out.println("methodB()");
        methodA();
        num++;
    }
}
//反编译结果
Classfile /D:/intellij/ideaWork/jvm/character01/target/classes/qq/com/rzf/DynamicLinking.class
  Last modified 2020-4-17; size 796 bytes//字节码文件开始信息  
  MD5 checksum 27df0439f1609f367ea5d285f4b09bdd
  Compiled from "DynamicLinking.java"
public class qq.com.rzf.DynamicLinking
  minor version: 0
  major version: 49
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool://常量池，#代表是符号引用
   #1 = Methodref          #9.#27         // java/lang/Object."<init>":()V
   #2 = Fieldref           #8.#28         // qq/com/rzf/DynamicLinking.num:I
   #3 = Fieldref           #29.#30        // java/lang/System.out:Ljava/io/PrintStream;
   #4 = String             #31            // methodA()
   #5 = Methodref          #32.#33        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #6 = String             #34            // methodB()
   #7 = Methodref          #8.#35         // qq/com/rzf/DynamicLinking.methodA:()V
   #8 = Class              #36            // qq/com/rzf/DynamicLinking
   #9 = Class              #37            // java/lang/Object
  #10 = Utf8               num
  #11 = Utf8               I  //代表int类型变量
  #12 = Utf8               <init>
  #13 = Utf8               ()V//代表空参和void返回类型
  #14 = Utf8               Code
  #15 = Utf8               LineNumberTable
  #16 = Utf8               LocalVariableTable
  #17 = Utf8               this
  #18 = Utf8               Lqq/com/rzf/DynamicLinking;
  #19 = Utf8               main
  #20 = Utf8               ([Ljava/lang/String;)V
  #21 = Utf8               args
  #22 = Utf8               [Ljava/lang/String;
  #23 = Utf8               methodA
  #24 = Utf8               methodB
  #25 = Utf8               SourceFile
  #26 = Utf8               DynamicLinking.java
  #27 = NameAndType        #12:#13        // "<init>":()V
  #28 = NameAndType        #10:#11        // num:I
  #29 = Class              #38            // java/lang/System
  #30 = NameAndType        #39:#40        // out:Ljava/io/PrintStream;
  #31 = Utf8               methodA()
  #32 = Class              #41            // java/io/PrintStream
  #33 = NameAndType        #42:#43        // println:(Ljava/lang/String;)V
  #34 = Utf8               methodB()
  #35 = NameAndType        #23:#13        // methodA:()V
  #36 = Utf8               qq/com/rzf/DynamicLinking
  #37 = Utf8               java/lang/Object
  #38 = Utf8               java/lang/System
  #39 = Utf8               out
  #40 = Utf8               Ljava/io/PrintStream;
  #41 = Utf8               java/io/PrintStream
  #42 = Utf8               println
  #43 = Utf8               (Ljava/lang/String;)V
{
  public qq.com.rzf.DynamicLinking();
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
        line 3: 0
        line 4: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lqq/com/rzf/DynamicLinking;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 7: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       1     0  args   [Ljava/lang/String;

  public void methodA();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #4                  // String methodA()
         5: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 9: 0
        line 10: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lqq/com/rzf/DynamicLinking;

  public void methodB();//methodB()方法字节码
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=1, args_size=1
         0: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #6                  // String methodB()
         5: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: aload_0
         9: invokevirtual #7  //#7代表符号引用                // Method methodA:()V，在这里调用methodA()
        12: aload_0
        13: dup
        14: getfield      #2//调用常量池中#2                  // Field num:I //获取num
        17: iconst_1
        18: iadd
        19: putfield      #2                  // Field num:I
        22: return
      LineNumberTable:
        line 12: 0
        line 13: 8
        line 14: 12
        line 15: 22
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      23     0  this   Lqq/com/rzf/DynamicLinking;
}
SourceFile: "DynamicLinking.java"
~~~

### 方法的调用，解析与分派

在jvm中，将符号引用转为调用方法的直接引用于方法的绑定机制相关。

**静态链接**

当一个字节码文件被装载到jvm内部的时候，如果被调用的目标方法在编译期间就明确的可以知道，并且在运行期间保持不变，这种情况下将调用方法的符号引用转换为直接引用的过程称为静态链接。

**动态链接**

如果被调用的方法在编译期间无法被确定下来，也就是说，只能够在程序的运行期间将调用方法的符号引用转换为直接引用，由于这种引用转换过程是在运行时确定，具备动态性，因此也就称为动态链接。

> 小结：
>
> 从方法中的符号引用根据符号在常量池中找到具体对应的方法就叫做符号引用到直接引用的转换。
>
> 符号引用转换为直接引用：如果在编译期间确定下来，就是静态链接，如果是在运行期间确定下来，就是动态链接

**方法的绑定**

对应的方法绑定机制为：早起绑定（early binding）和晚期绑定（late binding），绑定是一个字段，方法，或者类在符号引用被替换为直接引用的过程，仅仅发生一次。

- 早期绑定：早期绑定就是指被调用的目标方法如果在编译期间可知，且在运行期间保持不变时，即可以将这个方法与所属的类型进行绑定，这样一    来，由于明确了被调用的目标方法究竟是哪一个，因此也就可以使用静态链    接的方式将符号引用转换为直接引用。、
- 晚期绑定：如果被调用的方法在编译期间无法被确定下来，只能够在程序运行期间根据实际的类型绑定相关的方法，这种绑定方式就被定义为晚期绑定。

> 小结：
>
> 编译期间决定---->早期绑定
>
> 运行期间决定---->晚期绑定
>
> 晚期绑定体现在多态处。构造器表现为早期绑定。

**虚方法与非虚方法**

子类对象的多态性的使用前提，类的继承关系，方法的重写。

- 非虚方法：如果方法在编译期间就确定了具体的调用版本，这个版本在运行时是不可以改变的，这样的方法称为非虚方法。
  - 静态方法，私有方法，final方法，实例构造器，父类方法都属于非虚方法。
  - 其他的方法称为虚方法。
  - 非虚方法都不能被重写，因此在编译期间就可以被确定，而虚方法可以被重写，因此在运行期间才可以确定。
  - Java中如果不想让方法为虚方法，可以用final关键字修饰方法

**虚拟机中提供了以下几条方法调用指令。**

- 普通调用指令。
  - invokestatic：调用静态方法，解析阶段确定唯一方法的版本。
  - invokespecial：调用init方法，私有以及父类方法，解析阶段确定的唯一方法版本。
  - invokevirtual：调用所有虚方法。
  - invokeinterface：调用接口方法。
- 动态调用指令：
  - invokedynamic：动态解析出需要调用的方法，然后执行。

前面四条指令固化在虚拟机的内部，方法的执行不可以人为的干预，而invokedynamic指令则支持用户确定方法的版本，其中invokestatic和invokespecial指令调用的方法称为非虚方法。其余的称为虚方法（final修饰的除外，应为不可以被继承重写）。

**关于invokedynamic指令：**

-  jvm字节码指令集一直比较稳定，一直到java7才增加了一个invokedynamic指令，这是java为了实现*动态类型语言*支持做的一种改进。
- 但是在java7中并没有提供直接生成invokedynamic指令的方法，需要借助底层ASM这种底层字节码工具来产生invokedynamic指令，直到java8的lambda表达式出现，invokedynamic指令的生成，在java中才有了直接生成的方式。

**动态类型语言和静态类型的语言**

- 动态类型语言和静态类型语言两者的区别就是在于**对类型的检查是在编译期间还是在运行期间，在编译期间就是静态类型语言，运行期间就是动态类型语言**比如python。
- **静态类型语言是判断变量自身的类型信息，动态类型语言是判断变量值的类型信息**，变量没有类型信息，变量值才有类型信息，这是动态语言的重要特性。根据变量值来确定类型。

**java语言中方法重写的本质。**

- 找到操作数栈顶的第一个元素所执行的对象的实际类型，记为c。

- 如果在类型c中找到与常量中描述符合简单名称都相符的方法，则进行访问权限验证，如果通过就返回这个方法的直接引用，查找过程结束，如果不通过，就直接返回java.lang.illegalaccesserror异常。

- 否则，按照继承关系从上往下依次对c的各个父类进行第二步的搜索和验证过程。

- 如果始终没有找到合适的方法，就抛出java.lang.abstractmethoderror异常。

**java.lang.illegalaccesserror介绍。**

程序试图访问或者修改一个属性或者调用一个方法，这个属性或者方法，如果你没有访问权限，一般的，这个会引起编译器异常，这个错误如果发生在运行时，就说明这个类发生了不兼容的改变。

**虚方法表**

- 在面向对象的编程中，会很频繁的使用动态分派，如果每一次动态分派的过程中都需要重新在类的方法元数据中搜索合适的目标的话就很可能影响到执行效率，因此，为了提高性能，jvm采用在类的方法区建立一个虚方法表（非虚方法不会出现在表中）来实现，使用索引表来代替查找。

- 每一个类中都有一个虚方法表，表中存放着各个方法的实际入口。

- 虚方法表会在类加载的连接阶段被创建并开始初始化，类的变量初始值准备完成之后，jvm会把该类的方法表也初始化完毕。

**虚方法表案例**

虚方法表：friendly是一个接口

![1630717093707](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/04/152726-459185.png)

dog的虚方法表，其中dog重写了object类的tostring()和sayHello()方法。

![1630717065471](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/04/152753-982990.png)

也就是说dog类实现了父类的那些方法，dog类的虚方法表中的函数地址就直接指向dog类，调用的时候直接调用dog实现的方法，如果dog没有实现，就调用dog父类的方法。

具体狗的实现类，没有重写父类dog的tostring()方法，那么此方法就指向父类的tostring方法，实现了父类sayHello()和sayGoodbye方法，那么这两个方法就指向具体的实现类，如果用其他的方法，都是调用object()中的方法。

cat实现类:cat实现类重写的方法全部指向cat类，其余指向object类。这就是cat的虚方法表。

![1630717195263](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/04/152740-888632.png)

### 方法返回地址

- 本质上，方法的退出就是当前栈帧出栈的过程，此时，需要恢复上层方法的局部变量表，操作数栈，将返回值压入调用者栈的操作数栈，设置pc寄存器值等，让调用者的方法继续执行下去。
- 存放调用该方法的pc1寄存器的值。
- 一个方法的结束有两种方式
  - 正常执行完毕，执行return语句。
  - 出现未处理的异常，非正常退出。
- 无论通过哪种方式退出，在方法退出后都会返回到该方法被调用的位置，方法正常退出时，调用者的pc计数器的值作为返回地址，即调用该方法的指令的下一条指令的地址，而通过异常退出的，返回地址是要通过异常表来确定，栈帧中一般不会保存这部分信息。

- 当一个方法开始执行后，只有两种方式可以退出这个方法：
  - 执行引擎遇到任意一个方法返回的字节码指令(return),会有返回值传递给上层的方法调用者，（正常完成的出口）。
    - 一个方法在正常调用完成之后究竟需要使用哪一个返回指令还需要根据方法返回值的实际数据类型而决定。
    - 在字节码指令中，返回指令包含ireturn（当返回值是boolean,byte,char,short,int），String类型和Data类型都是作为引用类型进行返回，即areturn返回指令，lreturn,freturn,dreturn,areturn另外还有一个return指令供声明为void的方法，实例初始化方法，类和接口的初始化方法使用。（也就是没有返回类型的函数，在字节码文件中最后用return语句返回）构造器使用return指令返回。
  - 在方法执行的过程中遇到了异常（exception），并且这个异常并没有在这个方法内部进行处理，也就是只要在本地方法的异常表中没有搜索到匹配的异常处理器，就会导致方法退出，（异常完成出口）。
  - 方法执行过程中抛出异常时的异常处理，存储在一个异常处理表中，方便在发生异常的时候找到处理异常的代码。下面这张图就是异常表：

![1630717501223](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/04/152758-836750.png)

前面两行数字都是字节码指令的地址，也就是说如果指令从4-16出现异常，那么就按照19行处进行处理。

注意：正常完成出口和异常完成出口的区别在于：通过异常完成的出口退出不会给他的上层调用者产生任何的返回值信息。

### 一些附加信息

栈帧中还允许携带与java虚拟机实现相关的一些附加信息。比如对调试程序支持的一些信息。

### 栈相关面试题

1. 开发中遇到的异常有哪些？（高级一点的异常：java虚拟机中的异常）

递归函数调用通常易发生栈溢出的情况。stackoverflowerror,一个一个的添加栈帧，当栈空间不足的时候，会发生栈溢出情况，通过-Xss来设置java栈的大小。栈空间可以是固定大小，也可以是动态变化的，当设置固定大小时，可能发生栈溢出情况，当设置动态变化时，可能发生内存溢出情况。

2. 调整栈大小，就可以保证不出现溢出情况么？

不可以，调整栈空间的大小，可以让stackoverflowerror出现的时间晚一点，但是不能完全避免。就比如给你500元你一周可以用完，但是给你1000元，你还会把钱用完，只不过用完时间晚一点而已。

**案例**

~~~ java
/**
 * 栈内存溢出
 * 默认不设置栈内存大小：num最大值是9654
 * 设置栈的大小：-Xss256k,num值是：2469
 */
public class StackErrorFlow {
    private static  int num=0;
    public static void main(String[] args) {
        System.out.println(num);
        num++;
        main(args);//主函数递归调用自己
    }
}
~~~

3. 垃圾回收机制是否会涉及到虚拟机栈？

不会，栈可能发生内存溢出，但是没有垃圾回收。

小结：

| 内存块区域                          | 栈溢出或者内存溢出 | GC     |
| ----------------------------------- | ------------------ | ------ |
| Pc寄存器                            | 不存在             | 不存在 |
| 虚拟机栈                            | 存在               | 不存在 |
| 本地方法栈（调用本地c函数使用的栈） | 存在               | 不存在 |
| 堆                                  | 存在               | 存在   |
| 方法区                              | 存在               | 存在   |

4. 分配栈内存空间越大越好么？

不一定，因为栈的总容量大小是一定的，如果给某一个线程的栈容量空间分配的很大，会导致其他线程的栈空间容量变小发生溢出情况，对于当前的线程，分配更大的栈内存。可以让stackoverflowerror出现的时间晚一点，并不能完全避免。

5. 方法中定义的局部变量是否是线程安全的？

这个问题要具体问题具体分析。线程安全：如果只有一个线程才可以操作此数据，则必定是线程安全的，如果是多个线程操作此数据，则此数据是共享数据，如果不考虑同步机制的话，会存在线程安全问题。

**案例**

~~~ java
//但是stringBuilder的声明方式是线程安全的
//因为method1()归某一个栈帧单独所有，其他线程不能够访问，所以是线程安全的，变量多个线程共享的时候存在线程安全问题。
    public static void mothod1(){
        //StringBuilder类本身线程不安全
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.append("aa");
        stringBuilder.append("bb");
    }
//method2()的stringBuilder声明方式是不安全的,参数是从外面传进来，不归一个线程单独所有，可能多个线程共享，所以不安全
    public static void mothod2( StringBuilder stringBuilder){
        //StringBuilder类本身线程不安全
        stringBuilder.append("aa");
        stringBuilder.append("bb");
    }
//method3()的stringBuilder声明方式是不安全的，虽然stringBuilder是在方法里面创建的，但是有返回值，可能被其他线程所调用修改，所以不安全。
    public static StringBuilder mothod3(){
        //StringBuilder类本身线程不安全
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.append("aa");
        stringBuilder.append("bb");
        return stringBuilder;
    }
//method4()的stringBuilder声明方式是安全的，Method3不安全，应为直接将stringBuilder进行返回，但是method4是安全的，应为在方法里面，stringBuilder已经不存在了，方法里面的StringbUILDER已经消亡，在这里stringBuilder安全，但是string可能不安全，应为tostring()方法有创建一个方法返回，可能多个线程共享string
    public static String mothod4(){
        //StringBuilder类本身线程不安全
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.append("aa");
        stringBuilder.append("bb");
        return stringBuilder.toString();
    }
~~~

小结：对象如果内部产生，内部消亡，就是安全的。不是内部产生的，或者内部产生，但又返回生命周期没有结束就是不安全的。

## 本地方法栈

1. Java虚拟机栈用于管理Java方法的调用，而本地方法栈用于管理本地方法的调用
2. 本地方法栈，也是线程私有的。
3. 允许被实现成固定或者是可动态拓展的内存大小。（在内存溢出方面是相同的）
   1. 如果线程请求分配的栈容量超过本地方法栈允许的最大容量，Java虚拟机将会抛出一个StackOverFlowError异常。
   2. 如果本地方法栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的本地方法栈，那么java虚拟机将会抛出一个OutOfMemoryError异常。
4. 本地方法是使用C语言实现的
5. 它的具体做法是Native Method Stack中登记native方法，在Execution Engine执行时加载本地方法库。
6. 当某个线程调用一个本地方法时，它就进入了一个全新的并且不再受虚拟机限制的世界。它和虚拟机拥有同样的权限。
   1.  本地方法可以通过本地方法接口来访问虚拟机内部的运行时数据区。
   2. 它甚至可以直接使用本地处理器中的寄存器。
   3. 直接从本地内存的堆中分配任意数量的内存。
7. 并不是所有的JVM都支持本地方法。因为Java虚拟机规范并没有明确要求本地方法栈的使用语言、具体实现方式、数据结构等。如果JVM产品不打算支持native方法，也可以无需实现本地方法栈。
8. 在hotSpot JVM中，直接将本地方法栈和虚拟机栈合二为一。

本地方法栈如下图所示：

![1630718225164](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/04/152801-369861.png)

也就是说本地方法栈主要和本地方法库，本地方法接口打交道。

## 本地方法接口

![1630718277975](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/04/152804-705014.png)

**什么是本地方法？**

- 简单来讲，**一个Native Method就是一个java程序的非java代码的接口**，一个Native Method 是这样一个java方法：该方法的实现由非Java语言实现，比如C。这个特征并非java特有，很多其他的编程语言都有这一机制，比如在C++ 中，你可以用extern “C” 告知C++ 编译器去调用一个C的函数。
- 在定义一个native method时，并不提供实现体（有些像定义一个Java interface），因为其实现体是由非java语言在外面实现的。
- 本地接口的作用是融合不同的编程语言为java所用，它的初衷是融合C/C++程序。
- 标识符native可以与其他所有的java标识符连用，但是abstract除外。

**代码案例**

~~~ java
public class NativeTest {
    //abstract 没有方法体
//    public abstract void abstractMethod(int x);
// 此时的本地方法不意味着是抽象方法
    //native 和 abstract不能共存，native是有方法体的，表名调用是一个本地方法，由C语言来实现
//    但是抽象方法是没有方法体的
    public native void Native1(int x);
//native和static可以共存
    native static public long Native2();
    native synchronized private float Native3(Object o);
//本地方法还可以抛出异常
    native void Native4(int[] array) throws Exception;
}
~~~

**为什么需要本地方法**

java使用起来非常方便，然而有些层次的任务用java实现起来不容易，或者我们对程序的效率很在意时，问题就来了。

- 与java环境外交互：

有时java应用需要与java外面的环境交互，这是本地方法存在的主要原因。 你可以想想java需要与一些底层系统，如擦偶系统或某些硬件交换信息时的情况。本地方法正是这样的一种交流机制：它为我们提供了一个非常简洁的接口，而且我们无需去了解java应用之外的繁琐细节。

- 与操作系统交互

JVM支持着java语言本身和运行库，它是java程序赖以生存的平台，它由一个解释器（解释字节码）和一些连接到本地代码的库组成。然而不管怎样，它毕竟不是一个完整的系统，它经常依赖于一些底层系统的支持。这些底层系统常常是强大的操作系统。通过使用本地方法，我们得以用java实现了jre的与底层系统的交互，甚至jvm的一些部分就是用C写的。还有，如果我们要使用一些java语言本身没有提供封装的操作系统特性时，我们也需要使用本地方法。

- Sun’s Java

Sun的解释器是用C实现的，这使得它能像一些普通的C一样与外部交互。jre大部分是用java实现的，它也通过一些本地方法与外界交互。例如：类java.lang.Thread的setPriority()方法是用Java实现的，但是它实现调用的事该类里的本地方法setPriority0（）。这个本地方法是用C实现的，并被植入JVM内部，在Windows 95的平台上，这个本地方法最终将调用Win32 SetProority（）API。这是一个本地方法的具体实现由JVM直接提供，更多的情况是本地方法由外部的动态链接库（external dynamic link library）提供，然后被JVM调用。

- 现状

目前该方法的使用越来越少了，除非是与硬件有关的应用，比如通过java程序驱动打印机或者java系统管理生产设备，在企业级应用已经比较少见。因为现在的异构领域间的通信很发达，比如可以使用Socket通信，也可以是用Web Service等等，不多做介绍。