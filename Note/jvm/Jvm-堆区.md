# 深入理解JVM堆区

* [深入理解JVM堆区](#深入理解jvm堆区)
  * [堆的核心概念](#堆的核心概念)
    * [JvisualVM可视化查看堆内存](#jvisualvm可视化查看堆内存)
    * [堆内存的细分](#堆内存的细分)
  * [设置堆内存的大小与oom](#设置堆内存的大小与oom)
    * [查看堆内存的大小](#查看堆内存的大小)
  * [年轻代与老年代](#年轻代与老年代)
  * [图解对象内存的分配过程](#图解对象内存的分配过程)
    * [图解对象分配过程(一般情况)](#图解对象分配过程一般情况)
    * [图解对象分配过程(特殊情况)](#图解对象分配过程特殊情况)
    * [调优工具](#调优工具)
  * [Minor GC，Major GC，Full GC](#minor-gcmajor-gcfull-gc)
  * [堆空间的分代思想](#堆空间的分代思想)
  * [内存分配策略](#内存分配策略)
  * [为对象分配内存，tlab](#为对象分配内存tlab)
  * [堆空间参数设置](#堆空间参数设置)
  * [堆是分配对象的唯一选择么](#堆是分配对象的唯一选择么)
  * [代码优化](#代码优化)
    * [栈上分配](#栈上分配)
    * [同步省略](#同步省略)
    * [分离对象或标量替换](#分离对象或标量替换)
  * [逃逸分析小结](#逃逸分析小结)
  * [小结](#小结)



![1607685908513](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/192524-902905.png)

堆空间和方法区空间是进程独有的，也就是说一个进程中的所有线程需要共享着两个方法区。灰色区域是每一个线程都有一份，线程独有。

## 堆的核心概念

堆针对一个`jvm`进程来说是唯一的，一个进程对应一个`jvm`实例，一个运行时数据区，又包含多个线程，这些线程共享了方法区和堆，每个线程包含了程序计数器、本地方法栈和虚拟机栈。

- 当程序启动的时候，一个`jvm`实例就被启动起来，所以此时运行时数据区也就被创建，数据区中里面的各个区域也被创建，数据区的大小此时也被确定。
- 一个`jvm`实例只存在一个堆内存，堆也是`java`内存管理的核心区域
- `Java`堆区在`JVM`启动的时候即被创建，其空间大小也就确定了。是`JVM`管理的最大一块内存空间（堆内存的大小是可以调节的）可以对我们的程序设置堆内存的大小;
- 《Java虚拟机规范》规定，堆可以处于物理上不连续的内存空间中，但在逻辑上它应该被视为连续的。
- 所有的线程共享`java`堆，在这里还可以划分线程私有的缓冲区（`TLAB:Thread Local
  Allocation Buffer`）.（面试问题：堆空间一定是所有线程共享的么？不是，TLAB线程在堆中独有的）。
- 《Java虚拟机规范》中对`java`堆的描述是：所有的==对象实例以及数组==都应当在运行时分配在堆上。
  - 从实际使用的角度看，“几乎”所有的对象的实例都在这里分配内存 （‘几乎’是因为可能存储在栈上）
- 数组或对象永远不会存储在栈上，因为栈帧中保存引用，这个引用指向对象或者数组在堆中的位置。
- 在方法结束后，堆中的对象不会马上被移除，仅仅在垃圾收集的时候才会被移除。
- 堆，是`GC(Garbage Collection`，垃圾收集器)执行垃圾回收的重点区域。频繁的垃圾回收会影响我们用户线程执行的性能。
- 代码演示

~~~ java
public class SimpleHeap {
    private int id;//属性、成员变量

    public SimpleHeap(int id) {
        this.id = id;
    }

    public void show() {
        System.out.println("My ID is " + id);
    }
    public static void main(String[] args) {
        SimpleHeap sl = new SimpleHeap(1);
        SimpleHeap s2 = new SimpleHeap(2);

        int[] arr = new int[10];

        Object[] arr1 = new Object[10];
    }
}
~~~

![1607686300783](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/193143-47380.png)

当我们使用`new`关键字创建对象的时候，在底层其实就是在堆空间中去开辟一块空间创建一个对象。

![1607686337916](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/193218-10965.png)

### JvisualVM可视化查看堆内存

- 代码

~~~ java
public class HeapDemo {
    public static void main(String[] args) {
        System.out.println("start...");
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("end...");
    }
}
~~~

- 配置`jvm`

![1607686508337](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/193510-40598.png)



注：写完程序后直接进行编译，然后打开`run--->edit configurations`选项--->`jvm options`选项，配置堆内存`-Xms 10m -Xmx 10m`，标示堆内存最小10 m,最大10m。

- 运行我们的程序，并且在`jdk`的安装目录的`bin`目录中打开`jvisualvm`，查看进程，可以看到我们设置的配置信息`（C:\Program Files\Java\jdk1.8.0_102\bin）`。

![1607686598468](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/193640-643697.png)



可以查看到我们配置的内存信息。也可以查看堆空间中的具体分配信息。

v![1607686620071](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/193701-111333.png)

### 堆内存的细分

现代垃圾收集器大部分都是基于分代收集理论的设计，堆空间细分为：

- `JDK7`以前：新生区+养老区+永久区
  - Young Generation Space：又被分为Eden区和Survior区 **==Young/New==**，新生代
  - Tenure generation Space： ==**Old/Tenure**==，老年代
  - Permanent Space： ==**Perm**==,永久代

![1607686840339](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/194042-526462.png)

- JDK 8以后： 新生区+养老区+元空间
  - Young Generation Space：又被分为Eden区和Survior区 ==Young/New==，新生代
  - Tenure generation Space： ==Old/Tenure==，老年代
  - Meta Space： ==Meta==，元空间

![1607686900072](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/194141-695740.png)

也就是说`jdk7`中的永久代在`jdk8`中修改为元空间。

- ==约定：新生区=新生代=年轻代，养老区=老年代=老年区，永久区=永久代==

- 在堆区计算内存时是不计算元空间数据区或者永久代数据区的。也就是说只有两部分：新生代+老年代。永久代和元空间是方法区的实现，在`jdk8`中，元空间的实现是本地内存。

![1607687062626](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/194424-384077.png)

- 其中在新生区又分为三部分：`Eden`区域，`Survivor0 `和`Surviver1`区域。

- 在这三个区域中可以存放对象的只能是`eden`区域+(`surviver0`或者`surviver1`区域二选一）

## 设置堆内存的大小与oom

`Java`堆区用于存储`java`对象实例，堆的大小在`jvm`启动时就已经设定好了，可以通过 `"-Xmx"`和 `"-Xms"`来进行设置。

-  `-Xms`用于表示堆的起始内存，等价于` -XX:InitialHeapSize`

  - `-Xms`用来设置堆空间（年轻代+老年代）的初始内存大小，不包括元空间或者是永久代，在`jdk8`后元空间内实际是在本地内存中。而在`jdk7`及以前，永久代的实现还是在堆内存中。
  - `-X` 是`jvm`的运行参数
  - `ms` 是`memory start`

-  `-Xmx`用于设置堆的最大内存，等价于` -XX:MaxHeapSize`

-  一旦堆区中的内存大小超过` -Xmx`所指定的最大内存时，将会抛出`OOM`异常

-  通常会将`-Xms`和`-Xmx`两个参数配置相同的值，其目的就是为了能够在`java`垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小，从而提高性能。

-  默认情况下，初始内存大小：物理内存大小/64;最大内存大小：物理内存大小/4

-  手动设置：`-Xms600m -Xmx600m`

-  查看设置的参数

  - 方式一： 终端输入`jps`， 然后` jstat -gc 进程id`


![1607689602120](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/202643-459093.png)

> jps：查看java进程
>
> jstat：查看某进程内存使用情况

~~~ java
SOC: S0区总共容量
S1C: S1区总共容量
S0U: S0区使用的量
S1U: S1区使用的量
EC: 伊甸园区总共容量
EU: 伊甸园区使用的量
OC: 老年代总共容量
OU: 老年代使用的量
1、

25600+25600+153600+409600 = 614400K

614400 /1024 = 600M

2、

25600+153600+409600 = 588800K

588800 /1024 = 575M

3、

并非巧合，S0区和S1区两个只有一个能使用，另一个用不了
~~~

  - 方式二：（控制台打印）`Edit Configurations->VM Options `添加`-XX:+PrintGCDetails`参数。

~~~ java
//运行时设置堆内存：-Xmx 600m -Xms 600m
public class HeapSpaceInitial {
    public static void main(String[] args) {
//Runtime类就相当于与一个运行时数据区，是一个单例的类，也就是只有一个实例对象
        //返回Java虚拟机中的堆内存总量
        long initialMemory = Runtime.getRuntime().totalMemory() / 1024 / 1024;
        //返回Java虚拟机试图使用的最大堆内存量
        long maxMemory = Runtime.getRuntime().maxMemory() / 1024 / 1024;

        System.out.println("-Xms : " + initialMemory + "M");//-Xms : 245M
        System.out.println("-Xmx : " + maxMemory + "M");//-Xmx : 3641M

        System.out.println("系统内存大小为：" + initialMemory * 64.0 / 1024 + "G");//系统内存大小为：15.3125G
        System.out.println("系统内存大小为：" + maxMemory * 4.0 / 1024 + "G");//系统内存大小为：14.22265625G

        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
~~~

- 控制台打印结果

![1607689696455](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/202817-315404.png)

### 查看堆内存的大小

补充：在面试过程中，如果回答异常，即空指针异常，类型转换异常，数组越界异常之外，也可以说错误，比如栈溢出错误，内存溢出错误，堆空间的内存溢出。

~~~ JAVA
public class OOMTest {
    public static void main(String[] args) {
        ArrayList<Picture> list = new ArrayList<>();
        while(true){
            try {
                Thread.sleep(20);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            list.add(new Picture(new Random().nextInt(1024 * 1024)));
        }
    }
}

class Picture{
    private byte[] pixels;

    public Picture(int length) {
        this.pixels = new byte[length];
    }
}
//设置虚拟机参数
-Xms600m -Xmx600m
~~~

- 堆内存变化图

![1607689463908](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/202424-319494.png)

- 原因：大对象导致堆内存溢出

![1607689505578](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/202506-852042.png)

## 年轻代与老年代

堆空间是由年轻代和老年代构成，不包括元数据区或者永久代（方法区的实现而已）。

- 存储在`JVM`中的`java`对象可以被划分为两类：

>1. 一类是生命周期较短的瞬时对象，这类对象的创建和消亡都非常迅速
>2. 另外一类对象时生命周期非常长，在某些情况下还能与JVM的生命周期保持一致 

- `Java`堆区进一步细分可以分为年轻代（`YoungGen`）和老年代（`OldGen`）
- 其中年轻代可以分为`Eden`空间、`Survivor0`空间和`Survivor1`空间（有时也叫`from`区，`to`区）这两个内存空间大小一致，只有一个被使用。逻辑上堆空间包括方法区，实际上是不存在的。

![1607690100840](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/203503-57222.png)

- 配置新生代与老年代在堆结构的占比
  - 默认-XX：`NewRatio=2`，表示新生代占1，老年代占2，新生代占整个堆的1/3。
  - 可以修改`-XX:NewRatio=4`，表示新生代占1，老年代占4，新生代占整个堆的1/5
  - `NewRatio`标示老年代所占据的比例。

![1607690478253](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/204120-119387.png)

`Java`虚拟机默认新生代和老年代的比例是1:2。也就是新生代占据1/3,老年代占据2/3。

- 在`hotSpot`中，`Eden`空间和另外两个`Survivor`空间缺省所占的比例是8：1：1（测试的时候是6：1：1），因为在这里涉及一个自适应机制，`jvm`自动进行调节，也可以选择关闭，开发人员可以通过选项
  `-XX:SurvivorRatio` 调整空间比例，如`-XX:SurvivorRatio=8。`
- 几乎所有的`Java`对象都是在`Eden`区被`new`出来的。`EDEN`放不下的时候就直接放入老年代。直接放入老年代的是大对象，比如数组对象。
- 绝大部分的`Java`对象都销毁在新生代了（`IBM`公司的专门研究表明，新生代80%的对象都是“朝生夕死”的）。
- 可以使用选项`-Xmn`设置新生代最大内存大小（这个参数一般使用默认值就好了）。
- 下面的图表示：新`new`出来的对象大部分是在`eden`区域的，当`eden`区域快满的时候，如果仍然没有被销毁，会放入`survivor`区域，如果对象还没有被销毁，就会放入老年代存储区域。

![1607690762170](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/204604-844183.png)

~~~ java
/**
 * -Xms600m -Xmx600m
 *
 * -XX:NewRatio ： 设置新生代与老年代的比例。默认值是2.
 * -XX:SurvivorRatio ：8:设置新生代中Eden区与Survivor区的比例。默认值是8，但是测试的时候是6，如果想要是8 ，需要显示指明。
 * -XX:-UseAdaptiveSizePolicy ：关闭自适应的内存分配策略 '-'关闭,'+'打开  （暂时用不到）
 * -Xmn:设置新生代的空间的大小。 （一般不设置）
 *
 */
public class EdenSurvivorTest {
    public static void main(String[] args) {
        System.out.println("我只是来打个酱油~");
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
~~~

打开`dos`命令，通过`jinfo -flag NewRatio` 进程`id`查看新生代和老年代占比结构。

## 图解对象内存的分配过程

为新对象分配内存是件非常严谨和复杂的任务，`JVM`的设计者们不仅需要考虑内存如何分配、在哪里分配的问题，并且由于内存分配算法与内存回收算法密切相关，所以还需要考虑`GC`执行完内存回收后是否会在内存空间中产生内存碎片。

- 具体分配过程

1. `new`的对象先放`eden`区。此区有大小限制。
2. 当伊甸园的空间填满时，程序又需要创建对象，`JVM`的垃圾回收器将对伊甸园区进行垃圾回收（`Minor GC`),将伊甸园区中的不再被其他对象所引用的对象进行销毁。再加载新的对象放到伊甸园区，没有被销毁的对象放入`survivor`区域。
3. 然后将伊甸园中的剩余对象移动到幸存者`survivor0`区。
4. 如果再次触发垃圾回收，此时上次幸存下来的放到幸存者`survivor0`区的，如果没有回收，就会放到幸存者`survivor1`区。
5. 如果再次经历垃圾回收，此时会重新放回幸存者`survivor0`区，接着再去幸存者`survivor1`区。
6. 啥时候能去养老区呢？可以设置次数。默认是15次。·可以设置参数：`-XX:MaxTenuringThreshold`=进行设置。
7. 在养老区，相对悠闲。当老年区内存不足时，再次触发`GC：Major GC`，进行养老区的内存清理。
8. 若养老区执行了`Major GC`之后发现依然无法进行对象的保存，就会产生`OOM`异常。

**总结**
    **针对幸存者s0,s1区：复制之后有交换，谁空谁是to**
    **关于垃圾回收：频繁在新生区收集，很少在养老区收集，几乎不再永久区/元空间收集。在新生代发生的GC叫做minor GC,在养老区发生的GC叫做major GC。**

### 图解对象分配过程(一般情况)

![1607691742080](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/27/081153-978539.png)

1. 我们创建的对象，一般都是存放在`Eden`区的，**当我们`Eden`区满了后，就会触发`GC`操作**，一般被称为 `YGC / Minor GC`操作
2. 当我们进行一次垃圾收集后，红色的对象将会被回收，而绿色的独享还被占用着，存放在`S0(Survivor From)`区。同时我们给每个对象设置了一个年龄计数器，经过一次回收后还存在的对象，将其年龄加 1。
3. 同时`Eden`区继续存放对象，当`Eden`区再次存满的时候，又会触发一个`MinorGC`操作，此时`GC`将会把 `Eden`和`Survivor From`中的对象进行一次垃圾收集，把存活的对象放到 `Survivor To（S1）`区，同时让存活的对象年龄 + 1

> 下一次再进行GC的时候，
>
> 1、这一次的s0区为空，所以成为下一次GC的S1区
>
> 2、这一次的s1区则成为下一次GC的S0区
>
> 3、也就是说s0区和s1区在互相转换。

4. 我们继续不断的进行对象生成和垃圾回收，当`Survivor`中的对象的年龄达到15的时候，将会触发一次 `Promotion` 晋升的操作，也就是将年轻代中的对象晋升到老年代中

关于垃圾回收：频繁在新生区收集，很少在养老区收集，几乎不在永久区/元空间收集

### 图解对象分配过程(特殊情况)

1. 如果来了一个新对象，先看看 `Eden` 是否放的下？
   - 如果` Eden` 放得下，则直接放到 `Eden` 区
   - 如果 `Eden `放不下，则触发` YGC` ，执行垃圾回收，看看还能不能放下？
2. 将对象放到老年区又有两种情况：
   - 如果 `Eden `执行了 `YGC `还是无法放不下该对象，那没得办法，只能说明是超大对象，只能直接放到老年代
   - 那万一老年代都放不下，则先触发`FullGC` ，再看看能不能放下，放得下最好，但如果还是放不下，那只能报`OOM `
3. 如果 `Eden `区满了，将对象往幸存区拷贝时，发现幸存区放不下啦，那只能便宜了某些新对象，让他们直接晋升至老年区

![1607692309539](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/211200-285076.png)

`YGC`是针对年轻代的垃圾回收，`FGC`是针对老年代的垃圾回收。在`Servivor`中的垃圾回收放置在`s0/s1`区域针对的是把新生代的对象放到s0或者s1区域。如果放不下，就直接把对象从新生代放入到老年代。对象存活超过阈值指的是对象的`age`是否超过16。

- 测试代码

~~~ java
public class HeapInstanceTest {
    byte[] buffer = new byte[new Random().nextInt(1024 * 200)];

    public static void main(String[] args) {
        ArrayList<HeapInstanceTest> list = new ArrayList<HeapInstanceTest>();
        while (true) {
            list.add(new HeapInstanceTest());
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
~~~

- 内存状况

![1607692439411](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/211401-271410.png)

查看我们的`eden`区域，发现内存使用是线性增长，每次在峰底部都是进行了`GC`操作，每一次在新生代进行`GC`后，没有被回收的对象都会放入我们的`s0`或者`s1`区域，对于老年代来说，其数据是不断的增长。程序报出内存溢出实际是发生在老年代区域。元空间是对我们类信息的加载，一般变化不大。

### 调优工具

- JDK命令行
- Eclipse：Memory Analyzer Tool
- Jconsole
- VisualVM
- Jprofiler
- Java Flight Recorder
- GCViewer
- GC Easy

## Minor GC，Major GC，Full GC

1.       我们都知道，`JVM`的调优的一个环节，也就是垃圾收集，我们需要尽量的避免垃圾回收，因为在垃圾回收的过程中，容易出现`STW（Stop the World）`的问题，而 `Major GC` 和 `Full GC`出现`STW`的时间，是`Minor GC`的10倍以上
2.       `JVM`在进行`GC`时，并非每次都针对上面三个内存区域（新生代、老年代、方法区）一起回收的，大部分时候回收都是指新生代。
3.       针对`hotSpot VM`的实现，它里面的`GC`按照回收区域又分为两大种类型：一种是部分收集（`Partial GC`），一种是整堆收集（`Full GC`）
4.       部分收集：不是完整收集整个`Java`堆的垃圾收集。其中又分为：
         1.       新生代收集（`Minor GC/Young GC`）：只是新生代（`eden,s0,s1`）的垃圾收集
         2.       老年代收集（`Major GC/Old GC`）：只是老年代的垃圾收集
                  1.       目前，只有`CMS GC`会有单独收集老年代的行为
                  2.       注意，很多时候`Major GC` 会和` Full GC`混淆使用，需要具体分辨是老年代回收还是整堆回收
         3.       混合收集（`Mixed GC`）：收集整个新生代以及部分老年代的垃圾收集
                  1.       目前，只有`G1 GC`会有这种行为
5.       整堆收集（`Full GC`）：收集整个`java`堆和方法区的垃圾收集
6.       **年轻代`GC（Minor GC）`触发机制**：
         1.       当年轻代空间不足时，就会触发`Minor GC`，这里的年轻代满指的是`Eden`代满，`Survivor`满不会引发`GC`.(每次`Minor GC`会清理年轻代的内存，`Survivor`是被动`GC`，不会主动`GC`)
         2.       因为`Java`对象大多都具备朝生夕灭的特性，所以`Monor GC` 非常频繁，一般回收速度也比较快，这一定义既清晰又利于理解。
         3.       `Minor GC` 会引发`STW（Stop the World）`，暂停其他用户的线程，等垃圾回收结束，用户线程才恢复运行。

![1607692893463](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1607692893463.png)

7. **老年代`GC(Major GC/Full GC)`触发机制**

   1. 指发生在老年代的`GC`,对象从老年代消失时，`Major GC` 或者 `Full GC` 发生了
   2. 出现了`Major GC`，经常会伴随至少一次的`Minor GC`（不是绝对的，在`Parallel Scavenge` 收集器的收集策略里就有直接进行`Major GC`的策略选择过程）
      - 也就是老年代空间不足时，会先尝试触发`Minor GC`。如果之后空间还不足，则触发`Major GC`
   3. `Major GC`速度一般会比`Minor GC`慢10倍以上，`STW`时间更长
   4. 如果`Major GC`后，内存还不足，就报`OOM`了

8. **Full GC**触发机制（涉及我们整个的堆空间）

   触发`Full GC`执行的情况有以下五种:

   1. 调用`System.gc()`时，系统建议执行`Full GC`，但是不必然执行。
   2. 老年代空间不足。
   3. 方法区空间不足。（不同的jdk实现不同）
   4. 通过`Minor GC`后进入老年代的平均大小小于老年代的可用内存。
   5. 由`Eden`区，`Survivor S0（from）`区向`S1（to）`区复制时，对象大小由于大于`ToSpace`可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小。

   说明：`Full GC` 是开发或调优中尽量要避免的，这样暂停时间会短一些。因为老年代占据空间更大，进行垃圾回收耗时间，`stw`时间就更长。

![1607696152919](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/11/221554-968236.png)

- 对于`Survivor0`区域的对象，如果其阈值达到15的话，就会直接放入老年代。而阈值没有达到15的话，就会放入`survivor1`区域。
- 在出现`oom`的情况之前，一定会出现`full GC,`报错`oom`一般是老年代空间不足，所以需要进行`full GC`,一般针对整个堆空间进行垃圾回收。并且`eden`区域的`GC`触发是主动的行为，但是幸存者区的`GC`是被动的行为。
- `Young GC ->Full GC -> OOM`

- 测试分代回收

~~~ java
/** 测试GC分代回收
 * 测试MinorGC 、 MajorGC、FullGC
 * -Xms9m -Xmx9m -XX:+PrintGCDetails
 */
public class GCTest {
    public static void main(String[] args) {
        int i = 0;
        try {
            List<String> list = new ArrayList<>();
            String a = "testGC";
            while (true) {
                list.add(a);
                a = a + a;
                i++;
            }

        } catch (Throwable t) {
            t.printStackTrace();
            System.out.println("遍历次数为：" + i);
        }
    }
}
~~~

- 输出

~~~ java
[GC (Allocation Failure) [PSYoungGen: 2037K->504K(2560K)] 2037K->728K(9728K), 0.0455865 secs] [Times: user=0.00 sys=0.00, real=0.06 secs] 
[GC (Allocation Failure) [PSYoungGen: 2246K->496K(2560K)] 2470K->1506K(9728K), 0.0009094 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 2294K->488K(2560K)] 3305K->2210K(9728K), 0.0009568 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 1231K->488K(2560K)] 7177K->6434K(9728K), 0.0005594 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 488K->472K(2560K)] 6434K->6418K(9728K), 0.0005890 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 472K->0K(2560K)] [ParOldGen: 5946K->4944K(7168K)] 6418K->4944K(9728K), [Metaspace: 3492K->3492K(1056768K)], 0.0045270 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(1536K)] 4944K->4944K(8704K), 0.0004954 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) java.lang.OutOfMemoryError: Java heap space
    at java.util.Arrays.copyOf(Arrays.java:3332)
    at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
    at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:448)
    at java.lang.StringBuilder.append(StringBuilder.java:136)
    at com.atguigu.java1.GCTest.main(GCTest.java:20)
[PSYoungGen: 0K->0K(1536K)] [ParOldGen: 4944K->4877K(7168K)] 4944K->4877K(8704K), [Metaspace: 3492K->3492K(1056768K)], 0.0076061 secs] [Times: user=0.00 sys=0.02, real=0.01 secs] 
遍历次数为：16
Heap
 PSYoungGen      total 1536K, used 60K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 1024K, 5% used [0x00000000ffd00000,0x00000000ffd0f058,0x00000000ffe00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
 ParOldGen       total 7168K, used 4877K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 68% used [0x00000000ff600000,0x00000000ffac3408,0x00000000ffd00000)
 Metaspace       used 3525K, capacity 4502K, committed 4864K, reserved 1056768K
  class space    used 391K, capacity 394K, committed 512K, reserved 1048576K

~~~

- `[PSYoungGen:      2037K->504K(2560K)]`：年轻代总空间为 2560K ，当前占用 2037K ，经过垃圾回收后剩余504K
- `2037K->728K(9728K)`：堆内存总空间为 9728K ，当前占用2037K ，经过垃圾回收后剩余728K

## 堆空间的分代思想

为什么要把`Java`堆分代？不分代就不能正常工作了么

- 经研究，不同对象的生命周期不同。70%-99%的对象都是临时对象。
  - 新生代：有`Eden`、`Survivor`构成（`s0,s1 `又称为`from to`），to总为空
  - 老年代：存放新生代中经历多次依然存活的对象，也可以说是存储大对象。

![1607743991682](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/12/113315-723479.png)

- 其实不分代完全可以，分代的唯一理由就是优化`GC`性能。如果没有分代，那所有的对象都在一块，就如同把一个学校的人都关在一个教室。`GC`的时候要找到哪些对象没用，这样就会对堆的所有区域进行扫描。
- 而很多对象都是朝生夕死的，如果分代的话，把新创建的对象放到某一地方，当`GC`的时候先把这块存储“朝生夕死”对象的区域进行回收，这样就会腾出很大的空间出来。（多回收新生代，少回收老年代，性能会提高很多）

![1607744056924](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/12/113419-228473.png)



## 内存分配策略

- 如果对象在`Eden`出生并经过第一次`Minor GC`后依然存活，并且能被`Survivor`容纳的话，将被移动到`Survivor`空间中，把那个将对象年龄设为1.对象在`Survivor`区中每熬过一次`MinorGC`，年龄就增加一岁，当它的年龄增加到一定程度（默认15岁，其实每个`JVM`、每个`GC`都有所不同）时，就会被晋升到老年代中

  - 对象晋升老年代的年龄阈值，可以通过选项 `-XX：MaxTenuringThreshold`来设置

- 针对不同年龄段的对象分配原则如下：

  - 优先分配到`Eden`
  - 大对象直接分配到老年代（一般指的是需要连续的空间对象，字符串或者数组都是大对象）, 尽量避免程序中出现过多的大对象
  - 长期存活的对象分配到老年代
  - 动态对象年龄判断
    - 如果`Survivor`区中相同年龄的所有对象大小的总和大于`Survivor`空间的一半，年龄大于或等于该年龄的对象可以直接进入到老年代。无需等到`MaxTenuringThreshold`中要求的年龄
  - 空间分配担保:`-XX:HandlePromotionFailure`

- 代码测试

  分配60m堆空间，新生代 20m ，Eden 16m， s0 2m， s1 2m，buffer对象20m，Eden 区无法存放buffer， 直接晋升老年代。

~~~ java
/** 测试：大对象直接进入老年代
 * -Xms60m -Xmx60m -XX:NewRatio=2 -XX:SurvivorRatio=8 -XX:+PrintGCDetails 此参数可以控制打印垃圾回收的日志
 */
public class YoungOldAreaTest {
    // 新生代 20m ，Eden 16m， s0 2m， s1 2m
    // 老年代 40m
    public static void main(String[] args) {
        //Eden 区无法存放buffer  晋升老年代
        byte[] buffer = new byte[1024 * 1024 * 20];//20m
    }
}
~~~

- 日志输出

![1607744357956](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/12/113919-186246.png)



## 为对象分配内存，tlab

![1607744417329](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/12/114019-28441.png)

为什么有`TLAB（Thread Local Allocation Buffer）`

- 堆区是线程共享区域，任何线程都可以访问到堆区中的共享数据。
- 由于对象实例的创建在`JVM`中非常频繁，在并发环境下从堆区中划分内存空间是线程不安全的。
- 为避免多个线程操作同一地址，需要使用加锁等机制，进而影响分配速度

什么是`TLAB`

- 从内存模型而不是垃圾收集的角度，对`Eden`区域继续进行划分，`JVM`为每个线程分配了一个私有缓存区域，它包含在`Eden`空间内。
- 多线程同时分配内存时，使用`TLAB`可以避免一系列的非线程安全问题，同时还能够提升内存分配的吞吐量，因此我们可以将这种内存分配方式称之为快速分配策略。
- 所有`OpenJDK`衍生出来的`JVM`都提供了`TLAB`的设计。
- 当某一个线程的`tlab`的空间占用满的时候，可以使用蓝色的公共空间。

说明

- 尽管不是所有的对象实例都能够在`TLAB`中成功分配内存，单`JVm`明确是将`TLAB`作为内存分配的首选.
- 在程序中，开发人员可以通过选项`“-XX:UseTLAB“ `设置是够开启`TLAB`空间
- 默认情况下，`TLAB`空间的内存非常小，仅占有整个`EDen`空间的1%，当然我们可以通过选项 ”-`XX:TLABWasteTargetPercent“ `设置`TLAB`空间所占用`Eden`空间的百分比大小.
- 一旦对象在`TLAB`空间分配内存失败时，`JVM`就会尝试着通过使用加锁机制确保数据操作的原子性，从而直接在`Eden`空间中分配了内存

**对象的分配过程（堆空间不一定是共享的，比如`tlab`是线程私有的）**

![1607744787534](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/jvm/202012/12/114628-570348.png)

- 代码演示
  - 终端输入 `jsp`，查看`TLABArgsTest`进程`id`
  - `jinfo -flag UseTLAB 64566（进程id）`，输出`-XX:+UseTLAB`，证明`TLAB`默认是开启的

~~~ java
/**
 * 测试-XX:UseTLAB参数是否开启的情况:默认情况是开启的
 */
public class TLABArgsTest {
    public static void main(String[] args) {
        System.out.println("我只是来打个酱油~");
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
~~~

## 堆空间参数设置

堆空间`jvm`的参数设置多一点，在方法区也有参数的设置，垃圾回收也有参数设置。

~~~ java
-XX:+PrintFlagsInitial: 查看所有参数的默认初始值:实例：-XX:+PrintFlagsInitial -XX:SurvivorRatio=5
-XX:+PrintFlagsFinal：查看所有的参数的最终值（可能会存在修改，不再是初始值）：-XX:+PrintFlagsFinal -XX:SurvivorRatio=5
    具体查看某个参数的指令：jps：查看当前运行中的进程
                           jinfo -flag SurvivorRatio 进程id： 查看新生代中Eden和S0/S1空间的比例
-Xms: 初始堆空间内存（默认为物理内存的1/64）
-Xmx: 最大堆空间内存（默认为物理内存的1/4）
-Xmn: 设置新生代大小（初始值及最大值）
-XX:NewRatio: 配置新生代与老年代在堆结构的占比，新生带占1，老年代占2
-XX:SurvivorRatio：设置新生代中Eden和S0/S1空间的比例：8:1：1
-XX:MaxTenuringThreshold：设置新生代垃圾的最大年龄(默认15)
-XX:+PrintGCDetails：输出详细的GC处理日志
打印gc简要信息：① -XX:+PrintGC   ② -verbose:gc
-XX:HandlePromotionFailure：是否设置空间分配担保
~~~

- 空间分配担保说明：

  在发生`Minor Gc`之前，虚拟机会检查老年代最大可用的连续空间是否大于新生代所有对象的总空间。

  - 如果大于，则此次`Minor GC`是安全的。
  - 如果小于，则虚拟机会查看`-XX:HandlePromotionFailure`设置值是否允许担保失败。（`JDK 7`以后的规则`HandlePromotionFailure`可以认为就是`true`）
    - 如果`HandlePromotionFailure=true`,那么会继续检查老年代最大可用连续空间是否大于历次晋升到老年代的对象的平均大小。
      - 如果大于，则尝试进行一次`Minor GC`,但这次`Minor GC`依然是有风险的；
      - 如果小于，则改为进行一次`Fu11 GC`。
    - 如果`HandlePromotionFailure=false`,则改为进行一次`Fu11 GC`。

在`JDK6 Update24之后（JDK7`），`HandlePromotionFailure`参数不会再影响到虚拟机的空间分配担保策略，观察`openJDK`中的源码变化，虽然源码中还定义了`HandlePromotionFailure`参数，但是在代码中已经不会再使用它。`JDK6 Update24`之后的规则变为只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小就会进行`Minor GC`,否则将进行`Full GC`。

## 堆是分配对象的唯一选择么

​	在《深入理解`Java`虚拟机》中关于`Java`堆内存有这样一段描述：随着`JIT`编译期的发展与逃逸分析技术逐渐成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了。 

​	在`Java`虚拟机中，对象是在`Java`堆中分配内存的，这是一个普遍的常识。但是，有一种特殊情况，那就是如果经过逃逸分析（`Escape Analysis`)后发现，一个对象并没有逃逸出方法的话，那么就可能被优化成栈上分配。这样就无需在堆上分配内存，也无须进行垃圾回收了。这也是最常见的堆外存储技术。

​	此外，前面提到的基于`OpenJDK`深度定制的`TaoBaoVM,`其中创新的`GCIH(GCinvisible heap)`技术实现`off-heap`,将生命周期较长的`Java`对象从`heap`中移至`heap`外，并且`GC`不能管理`GCIH`内部的`Java`对象，以此达到降低`GC`的回收频率和提升`GC`的回收效率的目的。

- 如何将堆上的对象分配到栈，需要使用逃逸分析手段。
- 这是一种可以有效减少`Java`程序中同步负载和内存堆分配压力的跨函数全局数据流分析算法。
- 通过逃逸分析，`Java Hotspot`编译器能够分析出一个新的对象的引用的使用范围从而决定是否要将这个对象分配到堆上。
- 逃逸分析的基本行为就是分析对象动态作用域：
  - 当一个对象在方法中被定义后，对象只在方法内部使用，则认为没有发生逃逸。
  - 当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸。例如作为调用参数传递到其他地方中。
- 如何快速的判断是否发生了逃逸分析，就看`new`的对象实体是否有可能在方法外被调用

没有发生逃逸的对象，则可以分配到栈上，随着方法执行的结束，栈空间就被移除，自然对象分配的空间也就释放了。

- 逃逸分析代码演示

~~~ java
public static StringBuffer createStringBuffer(String s1,String s2){
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    return sb;
}
~~~

由于上述方法返回的`sb`在方法外被使用，发生了逃逸，上述代码如果想要`StringBuffer sb`不逃出方法，可以这样写：

~~~ java
public static String createStringBuffer(String s1,String s2){
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    return sb.toString();
}
~~~

其中修改后的代码重新返回一个新的`String`对象，而在方法体内部`new`出来的对象在方法执行完毕后已经释放其空间，所以对象也就没有逃逸出方法体。

~~~ java
/**
 * 逃逸分析
 *
 *  如何快速的判断是否发生了逃逸分析，就看new的对象实体是否有可能在方法外被调用。
 */
public class EscapeAnalysis {

    public EscapeAnalysis obj;

    /*
    方法返回EscapeAnalysis对象，发生逃逸，也就是在方法体内部new出来的对象，现在已经被传出到方法体外部，作用域变化，所以已经逃逸。
     */
    public EscapeAnalysis getInstance(){
        return obj == null? new EscapeAnalysis() : obj;
    }
    /*
    为成员属性赋值，发生逃逸，同样在方法体内部new出来的对象赋值给成员变量，作用域自然发生变化，也就是发生逃逸。
     */
    public void setObj(){
        this.obj = new EscapeAnalysis();
    }
    //思考：如果当前的obj引用声明为static的？仍然会发生逃逸。

    /*
    对象的作用域仅在当前方法中有效，没有发生逃逸，方法执行完毕之后，对象占据的空间也被销毁。
     */
    public void useEscapeAnalysis(){
        EscapeAnalysis e = new EscapeAnalysis();
    }
    /*
    引用成员变量的值，发生逃逸，在这里是调用上面创建对象的方法，但是我们说的逃逸是针对对象个体的，在其他方法创建的对象，已经返回到另一个方法中，所以作用域已经发生变化，也就发生了逃逸。
     */
    public void useEscapeAnalysis1(){
        EscapeAnalysis e = getInstance();
        //getInstance().xxx()同样会发生逃逸
    }
}
~~~

只要对象没有发生逃逸，我们就可以使用栈上内存的分配，不必给对象分配内存在堆空间上面。

- 参数设置

  -  在`JDK 6u23`版本之后，`HotSpot`中默认就已经开启了逃逸分析
  - 如果使用了较早的版本，开发人员可以通过
    - `-XX:DoEscapeAnalysis `显式开启逃逸分析
    - ` -XX:+PrintEscapeAnalysis`查看逃逸分析的筛选结果

- 结论

  开发中能使用局部变量的，就不要使用在方法外定义

## 代码优化

- 使用逃逸分析，编译器可以对代码做如下优化：
  - 栈上分配：将堆分配转化为栈分配。如果一个对象在子线程中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配。
  - 同步省略：如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步。
  - 分离对象或标量替换：有的对象可能不需要作为一个连续的内存结构存在也可以北方问道，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。

### 栈上分配

- `JIT`编译器在编译期间根据逃逸分析的结果，发现如果一个对象并没有逃逸出方法的话，就可能被优化成栈上分配。分配完成之后，继续在调用栈内执行，最后线程结束，栈空间被回收，局部变量对象也被回收。这样就无须进行垃圾回收了

- 常见的栈上分配场景：给成员变量赋值、方法返回值、实例引用传递

- 代码分析

  以下代码，关闭逃逸分析（`-XX:-DoEscapeAnalysi`），维护10000000个对象，如果开启逃逸分析，只维护少量对象（`JDK7 `逃逸分析默认开启）

~~~ java
public class StackAllocation {
    /**
     * 栈上分配测试，-DoEscapeAnalysis：-号表示关闭逃逸分析
     * 首先先不开启逃逸分析，也就是说对象的分配全部在堆内存空间
     * 1，-DoEscapeAnalysis：关闭逃逸分析
     * 2，+DoEscapeAnalysis：开启逃逸分析，也就是当前对象的分配是在栈上
     * 3，把堆内存的设置为256m
     * 4,关闭逃逸分析，发现堆内存有垃圾回收的行为
     * 5，打开逃逸分析，发现堆内存没有垃圾回收的行为
     * -Xmx256m -Xms256m -XX:+DoEscapeAnalysis -XX:+PrintGCDetails
     * -Xmx1G -Xms1G -XX:-DoEscapeAnalysis -XX:+PrintGCDetails
     * 在抽样器中查看内存以及对象的情况
     */
    public static void main(String[] args) {
//        起始时间
        long start = System.currentTimeMillis();

        for (int i = 0; i < 10000000; i++) {
            alloc();
        }
        // 查看执行时间
        long end = System.currentTimeMillis();
        System.out.println("花费的时间为： " + (end - start) + " ms");
        // 为了方便查看堆内存中对象个数，线程sleep
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
    }
    private static void alloc() {
        User user = new User();//未发生逃逸
    }

    static class User {

    }
}
~~~

- 日志输出

~~~ java
[GC (Allocation Failure) [PSYoungGen: 33280K->808K(38400K)] 33280K->816K(125952K), 0.0483350 secs] [Times: user=0.00 sys=0.00, real=0.06 secs] 
[GC (Allocation Failure) [PSYoungGen: 34088K->808K(38400K)] 34096K->816K(125952K), 0.0008411 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 34088K->792K(38400K)] 34096K->800K(125952K), 0.0008427 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 34072K->808K(38400K)] 34080K->816K(125952K), 0.0012223 secs] [Times: user=0.08 sys=0.00, real=0.00 secs] 
花费的时间为： 114 ms
~~~

1，`JVM` 参数设置

`-Xmx128m -Xms128m -XX:-DoEscapeAnalysis -XX:+PrintGCDetails`

2、日志打印：发生了 `GC `，耗时 114ms

**开启逃逸分析的情况**

输出结果：花费的时间为： 5 ms

1、参数设置

`-Xmx128m -Xms128m -XX:+DoEscapeAnalysis -XX:+PrintGCDetails`

2、日志打印：并没有发生 GC ，耗时5ms 。

### 同步省略

线程同步的代价是相当高的，同步的后果是降低并发性和性能

在动态编译同步块的时候，`JIT`编译器可以借助逃逸分析来判断同步块所使用的锁对象是否只能够被一个线程访问而没有被发布到其他线程。如果没有，那么`JIT`编译器在编译这个同步块的时候就会取消对这部分代码的同步。这样就能大大提高并发性和性能。这个取消同步的过程就叫同步省略，也叫==锁消除==

~~~ java
public class SynchronizedTest {
    public void f() {
        Object hollis = new Object();
        synchronized(hollis) {
            System.out.println(hollis);
        }
    }
    //代码中对hollis这个对象进行加锁，但是hollis对象的生命周期只在f（）方法中
    //并不会被其他线程所访问控制，所以在JIT编译阶段就会被优化掉。
    //优化为 ↓
    public void f2() {
        Object hollis = new Object();
        System.out.println(hollis);
    }
}
~~~

### 分离对象或标量替换

- 标量`Scalar`是指一个无法在分解成更小的数据的数据。`Java`中的原始数据类型就是标量
- 相对的，那些还可以分解的数据叫做聚合量(`Aggregate`)，`Java`中对象就是聚合量，因为它可以分解成其他聚合量和标量
- 在JIT阶段，如果经过逃逸分析，发现一个对象不会被外界访问的话，那么经过JIT优化，就会把这个对象拆解成若干个其中包含的若干个成员变量来替代。这个过程就是标量替换

~~~ java
public class ScalarTest {
    public static void main(String[] args) {
        alloc();   
    }
    public static void alloc(){
        Point point = new Point(1,2);
    }
}
class Point{
    private int x;
    private int y;
    public Point(int x,int y){
        this.x = x;
        this.y = y;
    }
}
~~~

以上代码，经过标量替换后，就会变成

~~~ java
public static void alloc(){
    int x = 1;
    int y = 2;
}
~~~

可以看到，`Point`这个聚合量经过逃逸分析后，发现他并没有逃逸，就被替换成两个标量了。那么标量替换有什么好处呢？就是可以大大减少堆内存的占用。因为一旦不需要创建对象了，那么就不再需要分配堆内存了。
标量替换为栈上分配提供了很好的基础。

~~~ java
/**
 * 标量替换测试
 *  -Xmx100m -Xms100m -XX:+DoEscapeAnalysis -XX:+PrintGC -XX:-EliminateAllocations
 */
public class ScalarReplace {
    public static class User {
        public int id;//标量（无法再分解成更小的数据）
        public String name;//聚合量（String还可以分解为char数组）
    }

    public static void alloc() {
        User u = new User();//未发生逃逸
        u.id = 5;
        u.name = "www.atguigu.com";
    }

    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000000; i++) {
            alloc();
        }
        long end = System.currentTimeMillis();
        System.out.println("花费的时间为： " + (end - start) + " ms");
    }
}
~~~

对于`jvm`的参数设置，如果先不开启标量分析，那么创建对象是在堆上分配内存，中间会发生垃圾回收。如果打开标量分析，那么就是在栈上分配对象内存（变换一种形式的栈上分配），所以中间不会触发垃圾回收机制。

## 逃逸分析小结

- 关于逃逸分析的论文在1999年就已经发表了，但直到`JDK1.6`才有实现，而且这项技术到如今也并不是十分成熟的。
- 其根本原因就是无法保证逃逸分析的性能消耗一定能高于他的消耗。虽然经过逃逸分析可以做标量替换、栈上分配、和锁消除。但是逃逸分析自身也是需要进行一系列复杂的分析的，这其实也是一个相对耗时的过程。
- 一个极端的例子，就是经过逃逸分析之后，发现没有一个对象是不逃逸的。那这个逃逸分析的过程就白白浪费掉了。
- 虽然这项技术并不十分成熟，但是它也是即时编译器优化技术中一个十分重要的手段。
- 注意到有一些观点，认为通过逃逸分析，`JVM`会在栈上分配那些不会逃逸的对象，这在理论上是可行的，但是取决于`JVM`设计者的选择。据我所知，`Oracle HotspotJVM`中并未这么做，这一点在逃逸分析相关的文档里已经说明，所以可以明确所有的对象实例都是创建在堆上。
- 目前很多书籍还是基于`JDK7`以前的版本，`JDK`已经发生了很大变化，`intern`字符串的缓存和静态变量曾经都被分配在永久代上，而永久代已经被元数据区取代。但是，`intern`字符串缓存和静态变量并不是被转移到元数据区，而是直接在堆上分配，所以这一点同样符合前面一点的结论：对象实例都是分配在堆上。

## 小结

- **年轻代是对象的诞生、成长、消亡的区域，一个对象在这里产生、应用、最后被垃圾回收器收集、结束生命。**

- **老年代防止长生命周期对象，通常都是从Survivor区域筛选拷贝过来的Java对象。**

- **当然，也有特殊情况，我们知道普通的对象会被分配在TLAB上。**

- **如果对象较大，JVM会试图直接分配在Eden其他位置上；**

- **如果对象太大，完全无法在新生代找到足够长的连续空闲空间，JVM就会直接分配到老年代。**

- **当GC只发生在年轻代中，回收年轻对象的行为被称为MinorGC。当GC发生在老年代时则被称为MajorGC或者FullGC（回收整个堆空间：包括新生代，老年代，方法区）。一般的，MinorGC的发生频率要比MajorGC高很多，即老年代中垃圾回收发生的频率大大低于年轻代。**

