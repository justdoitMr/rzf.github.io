# JVM调优

![1620562220208](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/09/201021-987065.png)

## 概述

### 背景说明

1. 生产环境发生了内存溢出该如何处理?
2. 生产环境应该给服务器分配多少内存合适？
3. 如何对垃圾回收器的性能进行调优？
4. 生产环境CPU负载飙高该如何处理？
5. 生产环境应该给应用分配多少线程合适？
6. 不加log，如何确定请求是否执行了某一行代码？
7. 不加log，如何实时查看某个方法的入参与返回值？

### 为什么要调优

1. 防止出现OOM
2. 解决OOM
3. 减少Full GC出现的频率

### 不同阶段的考虑

1. 上线前
2. 项目运行阶段
3. 线上出现OOM

### 监控的依据

1. 运行日志
2. 异常堆栈
3. GC日志
4. 线程快照
5. 堆转储快照

### 调优的大方向

1. 合理地编写代码
2. 充分并合理的使用硬件资源
3. 合理地进行JVM调优

### 性能优化步骤

#### （发现问题）：性能监控

1. GC频繁
2. cpu load过高
3. OOM
4. 内存泄露
5. 死锁
6. 程序响应时间较长

#### （排查问题）：性能分析

1. 打印GC日志，通过GCviewer或者`http://gceasy.io`来分析异常信息
2. 灵活运用命令行工具、jstack、jmap、jinfo等
3. dump出堆文件，使用内存分析工具分析文件
4. 使用阿里Arthas、jconsole、JVisualVM来实时查看JVM状态
5. jstack查看堆栈信息

#### （解决问题）：性能调优

1. 适当增加内存，根据业务背景选择垃圾回收器
2. 优化代码，控制内存使用
3. 增加机器，分散节点压力
4. 合理设置线程池线程数量
5. 使用中间件提高程序效率，比如缓存、消息队列等

### 性能评价/测试指标

1. 停顿时间（或响应时间），提交请求和相应该请求之间使用的时间，一般关注平均相应时间。
2. 吞吐量：
   1. 对单位时间内完成的工作量（请求）的量度
   2. 在GC中：运行用户代码的事件占总运行时间的比例（总运行时间：程序的运行时间+内存回收的时间）
      吞吐量为1-1/(1+n)，其中-XX::GCTimeRatio=n
3. 并发数：同一时刻，对服务器有实际交互的请求数
4. 内存占用：Java堆区所占的内存大小
5. 相互之间的关系：以高速公路通行状况为例，吞吐量就像每天通过高速公路的车辆数，并发数就是当前高速公路上车的数量，相应时间就是当前的车速。并发数增大，吞吐量就会增大，但是相应时间会降低。

## JVM监控及诊断工具-命令行篇

### 概述

性能诊断是软件工程师在日常工作中需要经常面对和解决的问题，在用户体验至上的今天，解决好应用的性能问题能带来非常大的收益。

Java作为最流行的编程语言之一，其应用性能诊断一直受到业界广泛关注。可能造成Java应用出现性能问题的因素非常多，例如线程控制、磁盘读写、数据库访问、网络I/O、垃圾收集等。想要定位这些问题，一款优秀的性能诊断工具必不可少。

- 体会1: 使用数据说明问题，使用知识分析问题，使用工具处理问题。

- 体会2: 无监控、不调优！

### 命令行工具

java的命令行工具在bin目录下面。命令对应的源文件存储在lib目录下面。

各种命令对应的源码地址：`<https://hg.openjdk.java.net/jdk/jdk11/file/1ddf9a99e4ad/src/jdk.jcmd/share/classes/sun/tools>`

#### jps:查看正在运行的Java进程

jps(Java Process Staflus):显示指定系统内所有的HotSpot虚拟机进程（查看**虚拟机进程**信息）,可用于查询正在运行的虚拟机进程。

~~~ java
jps [options] [hostid]
~~~

- 说明：对于本地虚拟机进程来说，进程的**本地虚拟机ID**与操作系统的**进程ID**是一致的，是唯一的。

##### 基本语法

~~~ java
jps -help

C:\Users\MrR>jps -help
usage: jps [-help]
       jps [-q] [-mlvV] [<hostid>]

Definitions:
    <hostid>:      <hostname>[:<port>]
~~~

##### options参数

**-q**

仅仅显示LVMID(local virtual machine id),即本地虚拟机唯一id。不显示主类的名称等

~~~ java
jps -q //也就是仅仅显示进程的id号
~~~

**-l**

输出应用程序主类的全类名或如果进程执行的是jar包，则输出jar完整路径

~~~ java
jps -l

C:\Users\MrR>jps -l
14280 sun.tools.jps.Jps
18232
~~~

**-m**

输出虚拟机进程启动时传递给主类main()的参数

~~~ java
jps -m
~~~

**-v**

列出虚拟机进程启动时的JVM参数

比如：-Xms20m-Xmx50m是启动程序指定的jvm参数

~~~ java
jps -v

C:\Users\MrR>jps -v
18232  -Dfile.encoding=UTF-8 -Xms128m -Xmx1024m -XX:MaxPermSize=256m
15340 Jps -Dapplication.home=D:\soft\jdk1.8 -Xms8m
~~~

- 说明：以上参数可以综合使用。

- 补充：
   如果某Java进程关闭了默认开启的`UsePerfData`参数（即使用参数`-XX:-UsePerfData`) , 那么 `jps` 命令以及 `jstat` 将无法探知该Java 进程。

设置 VM 参数 和 传入参数

![1620621359720](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/10/123600-632054.png)

上面参数中，-q参数单独使用，-mlvv参数可以搭配使用。

##### hostid参数

- RMI注册表中注册的主机名。

- 如果想要远程监控主机上的 java 程序，需要安装 jstatd。

- 对于具有更严格的安全实践的网络场所而言，可能使用一个自定义的策略文件来显示对特定的可信主机或网络的访问，尽管这种技术容易受到IP地址欺诈攻击。

- 如果安全问题无法使用一个定制的策略文件来处理，那么最安全的操作是不运行jstatd服务器，而是在本地使用jstat和jps工具。

#### jstat:查看JVM统计信息

以后使用以下程序做测试

~~~ java
public class Test {
    public static void main(String[] args) {

        Scanner scanner = new Scanner(System.in);
        String s = scanner.nextLine();
    }
}
~~~

##### 基本情况

- jstat(JVM Statistics Monitoring Tool): 用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。
- 在没有GUI图形界面，只提供了纯文本控制台环境的服务器上，它将是运行期定位虚拟机性能问题的首选工具。常用于检测垃圾回收问题以及内存泄漏问题。

##### 基本语法

~~~ java
jstat --help|-options

jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
~~~

##### Options选项

选项option可以由以下值构成 :

###### **类装载相关的：**

~~~ java
jstat -class pid //可以查看类装载信息
~~~

显示ClassLoader的相关信息：**类的装载**、**卸载数量**、**总空间**、**类装载所消耗的时间**等

~~~ java
jstat -class PID
  
C:\Users\MrR>jps
16816
21952 Launcher
18232
14684 Jps
668 Test

//这里使用的进程号是Test程序的进程号
C:\Users\MrR>jstat -class 668
Loaded  Bytes  Unloaded  Bytes     Time
   708  1410.5        0     0.0       0.14
//从左到右表示：加载类的个数，加载类所占的空间，卸载类个数
~~~

###### **垃圾回收相关**

**使用的代码**

~~~ java
//参数配置：-Xms60m -Xmx60m -XX:SurvivorRatio=8
public class GCTest {

    public static void main(String[] args) {
        ArrayList<byte[]> list = new ArrayList<>();
        int num = 1000;
        for (int i = 0; i < num; i++) {
            //100KB
            byte[] arr = new byte[1024 * 100];
            list.add(arr);

            try{
                Thread.sleep(100);
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
    }
}
~~~

**新生代相关**

- S0C是第一个幸存者区的大小（字节）
- S1C是第二个幸存者区的大小（字节）
- S0U是第一个幸存者区已使用的大小（字节）
- S1U是第二个幸存者区已使用的大小（字节）
- EC是Eden空间的大小（字节）
- EU是Eden空间已使用大小（字节）

**老年代相关**

- OC是老年代的大小（字节）
- OU是老年代已使用的大小（字节）

**方法区（元空间）相关**

- MC是方法区的大小
- MU是方法区已使用的大小
- CCSC是压缩类空间的大小
- CCSU是压缩类空间已使用的大小

**其他**

- YGC是从应用程序启动到采样时young gc的次数
- YGCT是指从应用程序启动到采样时young gc消耗时间（秒）
- FGC是从应用程序启动到采样时full gc的次数
- FGCT是从应用程序启动到采样时的full gc的消耗时间（秒）
- GCT是从应用程序启动到采样时gc的总时间

-gc:显示与GC相关的堆信息。包括Eden区、两个Survivor区、老年代、永久代等的容量、已用空间、GC时间合计等信息

![1620645190834](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/10/191312-406010.png)

-gccapacity:显示内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大、最小空间

![1620645612161](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/10/192016-475155.png)

-gcutil :显示内容与 `-gc` 基本相同，但输出主要关注已使用空间占总空间的百分比

![1620645291707](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/10/191452-38095.png)

-gccause :与 `-gcutil` 功能一样，但是会额外输出导致最后一次或当前正在发生的GC产生的原因

![1620645514598](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/10/191835-244600.png)

-gcnew :显示新生代GC状况

-gcnewcapacity :显示内容与-gcnew基本相同，输出主要关注使用到的最大、最小空间

-gcold :显示老年代GC状况

-gcoldcapacity :显示内容与-gcold基本相同，输出主要关注使用到的最大、最小空间

-gcpermcapacity :显示jdk8以前永久代使用到的最大、最小空间

-gcmetacapacity :显示 jdk8以后元空间使用到的最大、最小空间







###### **JIT及时编译相关**

显示JIT编译器编译过的方法、耗时等信息

~~~ java
jstat -compiler PID

C:\Users\MrR>jstat -compiler 21268
Compiled Failed Invalid   Time   FailedType FailedMethod
      76      0       0     0.05          0
~~~

输出已经被JIT编译的方法

~~~ java
jstat -printcompilation PID

C:\Users\MrR>jstat -printcompilation 12360
Compiled  Size  Type Method
      76     95    1 java/util/ArrayList add
~~~

##### interval参数

用于指定输出统计数据的周期，单位为毫秒。即：查询间隔，也就是多长时间打印一次统计信息。可以进行持续监控。

~~~ java
jstat -class 668 1000
//设置一秒钟打印一次
~~~

##### count参数

~~~ java
jstat -class 668 1000 10
//每隔一秒钟打印一次的基础上打印10次
~~~

##### -t参数

可以在输出信息前加上一个Timestamp列，显示程序的运行时间。单位：秒

~~~ java
jstat -class -t 668
Timestamp       Loaded  Bytes  Unloaded  Bytes     Time
          734.4    708  1410.5        0     0.0       0.14
//Timestamp时间戳打印程序刚开始执行到结束执行的时间
~~~

**经验**

- 可以比较 Java 进程的启动时间 以及 总GC 时间( GCT列 ), 或者 两次测量的间隔时间 以及 总GC时间 的增量, 来得出GC时间占运行时间的比例

- 如果该比例超过20%, 则说明目前堆的压力较大;
- 如果该比例超过90%, 则说明堆里几乎没有可用空间, 随时都可能抛出 OOM 异常

##### -h参数

可以在周期性数据输出时，输出多少行数据后输出一个表头信息,注意一点，是周期性的打印信息。每隔多少条数据，打印一次表头信息。

~~~ java
C:\Users\MrR>jstat -class -t -h3 668 1000 10
Timestamp       Loaded  Bytes  Unloaded  Bytes     Time
          914.0    708  1410.5        0     0.0       0.14
          915.0    708  1410.5        0     0.0       0.14
          916.0    708  1410.5        0     0.0       0.14
Timestamp       Loaded  Bytes  Unloaded  Bytes     Time
          917.0    708  1410.5        0     0.0       0.14
          918.0    708  1410.5        0     0.0       0.14
          919.0    708  1410.5        0     0.0       0.14
~~~

**补充**

jstat 还可以用来判断是否出现内存泄漏 :

- 第1步：
  - 在长时间运行的 Java 程序中，我们可以运行 jstat 命令连续获取多行性能数据，并取这几行数据中OU列（即已占用的老年代内存）的最小值。

- 第2步：
  - 然后，我们每隔一段较长的时间重复一次上述操作，来获得多组OU最小值。
  - 如果这些值呈上涨趋势，则说明该Java程序的老年代内存已使用量在不断上涨，这意味着无法回收的对象在不断增加，因此很有可能存在内存泄漏。

#### jinfo:实时查看和修改JVM配置参数

##### 基本说明

jinfo(Configuralion Info for Java) : 查看虚拟机配置参数信息，也可用于调整虚拟机的配置参数。

在很多情况下，Java应用程序不会指定所有的Java虚拟机参数。而此时，开发人员可能不知道某一个具体的Java虚拟机参数的默认值。

在这种情况下，可能需要通过查找文档获取某个参数的默认值。这个查找过程可能是非常艰难的。但有了jinfo工具，开发人员可以很方便地找到Java虚拟机参数的当前值。

##### 基本语法

~~~ java
jinfo [ options ] pid
//必须要添加java程序的pid号
~~~

**选项信息**

选项	选项说明
no option	输出全部的参数和系统属性
-flag name	输出对应名称的参数
-flag [±]name	开启或者关闭对应名称的参数只有被标记为manageable的参数才可以被动态修改
-flag name=value	设定对应名称的参数
-flags	输出全部的参数
-sysprops	输出系统属性

##### 查看

可以查看由System.getProperties()取得的参数

~~~ java
jinfo -sysprops PID 
~~~

![1620646539768](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/10/193540-443822.png)

查看曾经赋过值的一些参数

~~~ java
jinfo -flags PID
~~~

![1620646616411](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/10/193930-462553.png)

查看某个java进程的具体参数的值

~~~ java
jinfo -flag 具体参数(也就是要查看的参数) PID
~~~

![1620646812136](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1620646812136.png)

##### 修改

jinfo不仅可以查看运行时某一个Java虚拟机参数的实际取值，甚至可以在运行时修改部分参数，并使之立即生效。

但是，并非所有参数都支持动态修改。参数只有被标记为manageable的flag可以被实时修改。其实，这个修改能力是极其有限的。

可以查看被标记为manageable的参数

~~~ java
java-XX:+PrintFlagsFinal-version|grep manageable
~~~

![1620647221694](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/10/194705-974158.png)

**针对非boolean类型**

~~~ java
jinfo -flag 具体参数=具体参数值 PID
~~~

![1620647360051](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/10/194920-437770.png)

**针对boolean类型**

~~~ java
jinfo -flag [+-]具体参数 PID
~~~

![1620647445501](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/10/195047-770164.png)

##### 拓展

查看所有JVM参数启动的初始值

~~~ java
java -XX:+PrintFlagsInitial
~~~

查看所有JVM 参数的最终值

~~~ java
java -XX:+PrintFlagsFinal
~~~

查看那些已经被用户或者JVM设置过的详细的XX参数的名称和值

~~~ java
java -参数名称:+PrintCommandLineFlags
~~~

#### jmap:导出内存映像文件&内存使用情况

##### 基本说明

- jmap(JVM Memory Map) : 作用一方面是获取dump文件（堆转储快照文件，二进制文件）,它还可以获取目标Java进程的内存相关信息，包括Java堆各区域的使用情况、堆中对象的统计信息、类加载信息等。

- 开发人员可以在控制台中输入命令“ jmap-help ”查阅jmap工具的具体使用方式和一些标准选项配置。

##### 基本语法

~~~ java
jmap [option] <pid>
jmap [option] <executable <core> 
jmap [option] [server_id]remote server IP or hostname>
~~~

###### -dump

- 生成Java堆转储快照：dump文件
- 特别的：-dump:live只保存堆中的存活对象

###### -heap

- 输出整个堆空间的详细信息，包括GC的使用、堆配置信息，以及内存的使用信息等

###### -histo

输出堆中对象的同级信息，包括类、实例数量和合计容量

特别的：-histo:live只统计堆中的存活对象

###### -permstat

以ClassLoader为统计口径输出永久代的内存状态信息

仅linux/solaris平台有效

###### -finalizerinfo

显示在F-Queue中等待Finalizer线程执行finalize方法的对象

仅linux/solaris平台有效

###### -F

当虚拟机进程对-dump选项没有任何响应时，可使用此选项强制执行生成dump文件

仅linux/solaris平台有效

###### -h | -help

jamp工具使用的帮助命令

###### `-J <flag>`

传递参数给jmap启动的jvm

##### 使用1：导出内存映像文件

**手动的方式**

~~~ JAVA
jmap -dump:format=b,file=<filename.hprof> <pid>
//format:表示导出的是标准格式
//file:表示导出后文件的存储路径
jmap -dump:format=b,file=d:\

jmap -dump:live,format=b,file=<filename.hprof> <pid>
//添加live参数表示只是打印堆内存中存活的对象，这样做保存的文件不至于太大
~~~

> 生成的dump文件可以通过jprofiler工具打开

**手动的方式**

- 当程序发生OOM退出系统时，一些瞬时信息都随着程序的终止而消失，而重现OOM问题往往比较困难或者耗时。此时若能在OOM时，自动导出dump文件就显得非常迫切。

- 这里介绍一种比较常用的取得堆快照文件的方法，即使用：在程序发生OOM时，导出应用程序的当前堆快照 :

~~~ java
-XX:+HeapDumpOnOutOfMemoryError
~~~

可以指定堆快照的保存位置

~~~ java
-XX:HeapDumpPath==<filename.hprof>
~~~

**案例**

~~~ java
-Xmx100m -XX:+HeapDumpOnOutOfMemoryError 
-XX:HeapDumpPath=D:\Date\github\java\JVM\src\com\cpucode\java\command\tools\jmap\AutoDump.hprof
~~~

##### 使用2：显示堆内存相关信息

查看各个区的大小

~~~ java
jmap -heap 进程id
~~~

![1620651377252](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/10/205618-12391.png)

所有类型使用的内存信息

~~~ java
jmap -histo 进程id
~~~

![1620651414753](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/10/205655-172522.png)

##### 其他作用

查看系统的ClassLoader信息

~~~ java
jmap -permstat pid 
~~~

查看堆积在finalizer队列中的对象

~~~ java
jmap -finalizerinfo
~~~

##### 小结

- 由于 jmap 将访问堆中的所有对象，为了保证在此过程中不被应用线程干扰，jmap需要借助安全点机制，让所有线程停留在不改变堆中数据的状态。也就是说，由 jmap 导出的堆快照必定是安全点位置的。这可能导致基于该堆快照的分析结果存在偏差。

- 举个例子，假设在编译生成的机器码中，某些对象的生命周期在两个安全点之间，那么 :live 选项将无法探知到这些对象。

- 另外，如果某个线程长时间无法跑到安全点， jmap 将一直等下去。与前面讲的 jstat 则不同 , 垃圾回收器会主动将 jstat 所需要的摘要数据保存至固定位置之中，而 jstat 只需直接读取即可。

