## Flink监控与优化
<!-- TOC -->

- [Flink监控与优化](#flink监控与优化)
  - [Flink-Metrics监控](#flink-metrics监控)
    - [什么是 Metrics？](#什么是-metrics)
      - [Metric Types](#metric-types)
    - [WebUI监控](#webui监控)
  - [Flink性能优化](#flink性能优化)
    - [History Server](#history-server)
    - [序列化](#序列化)
    - [复用对象](#复用对象)
    - [数据倾斜](#数据倾斜)
  - [Flink-内存管理](#flink-内存管理)
    - [问题引入](#问题引入)
    - [内存划分](#内存划分)
    - [堆外内存](#堆外内存)
    - [序列化和反序列化](#序列化和反序列化)
    - [操纵二进制数据](#操纵二进制数据)
    - [面试](#面试)
  - [Flink VS Spark](#flink-vs-spark)
    - [应用场景](#应用场景)
    - [原理对比](#原理对比)
    - [运行角色](#运行角色)
    - [生态](#生态)
    - [运行模型](#运行模型)
    - [任务调度原理](#任务调度原理)
    - [时间机制对比](#时间机制对比)
    - [容错机制](#容错机制)
    - [窗口](#窗口)
    - [整合kafka](#整合kafka)
    - [Back pressure背压/反压](#back-pressure背压反压)

<!-- /TOC -->
### Flink-Metrics监控

#### 什么是 Metrics？

由于集群运行后很难发现内部的实际状况，跑得慢或快，是否异常等，开发人员无法实时查看所有的 Task 日志，比如作业很大或者有很多作业的情况下，该如何处理？此时 Metrics 可以很好的帮助开发人员了解作业的当前状况。

Flink 提供的 Metrics 可以在 Flink 内部收集一些指标，通过这些指标让开发人员更好地理解作业或集群的状态。

![1623562512223](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/133516-429630.png)

##### Metric Types

Metrics 的类型如下：

1. 常用的如 Counter，写过 mapreduce 作业的开发人员就应该很熟悉 Counter，其实含义都是一样的，就是对一个计数器进行累加，即对于多条数据和多兆数据一直往上加的过程。
2. Gauge，Gauge 是最简单的 Metrics，它反映一个值。比如要看现在 Java heap 内存用了多少，就可以每次实时的暴露一个 Gauge，Gauge 当前的值就是heap使用的量。
3. Meter，Meter 是指统计吞吐量和单位时间内发生“事件”的次数。它相当于求一种速率，即事件次数除以使用的时间。
4. Histogram，Histogram 比较复杂，也并不常用，Histogram 用于统计一些数据的分布，比如说 Quantile、Mean、StdDev、Max、Min 等。

Metric 在 Flink 内部有多层结构，以 Group 的方式组织，它并不是一个扁平化的结构，Metric Group + Metric Name 是 Metrics 的唯一标识。

#### WebUI监控

在flink的UI的界面上点击任务详情，然后点击Task Metrics会弹出如下的界面，在 add metic按钮上可以添加我需要的监控指标。

自定义监控指标

○  案例：在map算子内计算输入的总数据

○  设置MetricGroup为：flink_test_metric

○  指标变量为：mapDataNub

**代码实现**

```java
public class WordCount5_Metrics {
    public static void main(String[] args) throws Exception {
        //1.准备环境-env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);

        //2.准备数据-source
        //2.source
        DataStream<String> linesDS = env.socketTextStream("node1", 9999);
        //3.处理数据-transformation
        DataStream<String> wordsDS = linesDS.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
                //value就是一行行的数据
                String[] words = value.split(" ");
                for (String word : words) {
                    out.collect(word);//将切割处理的一个个的单词收集起来并返回
                }
            }
        });
        //3.2对集合中的每个单词记为1
        DataStream<Tuple2<String, Integer>> wordAndOnesDS = wordsDS.map(new RichMapFunction<String, Tuple2<String, Integer>>() {
            Counter myCounter;
            @Override
            public void open(Configuration parameters) throws Exception {
                myCounter= getRuntimeContext().getMetricGroup().addGroup("myGroup").counter("myCounter");
            }

            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                myCounter.inc();
                //value就是进来一个个的单词
                return Tuple2.of(value, 1);
            }
        });
        //3.3对数据按照单词(key)进行分组
        KeyedStream<Tuple2<String, Integer>, String> groupedDS = wordAndOnesDS.keyBy(t -> t.f0);
        //3.4对各个组内的数据按照数量(value)进行聚合就是求sum
        DataStream<Tuple2<String, Integer>> result = groupedDS.sum(1);

        //4.输出结果-sink
        result.print().name("mySink");

        //5.触发执行-execute
        env.execute();
    }
}
// /export/server/flink/bin/yarn-session.sh -n 2 -tm 800 -s 1 -d
// /export/server/flink/bin/flink run --class cn.itcast.hello.WordCount5_Metrics /root/metrics.jar
// 查看WebUI
```

○  程序启动之后就可以在任务的ui界面上查看

![1623562965529](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/134247-604597.png)







### Flink性能优化

#### History Server

flink的HistoryServer主要是用来存储和查看任务的历史记录

```java
# Directory to upload completed jobs to. Add this directory to the list of
# monitored directories of the HistoryServer as well (see below).
# 将已完成的作业上传到的目录
jobmanager.archive.fs.dir: hdfs://node01:8020/completed-jobs/

# The address under which the web-based HistoryServer listens.
# 基于 Web 的 HistoryServer 的地址
historyserver.web.address: 0.0.0.0

# The port under which the web-based HistoryServer listens.
# 基于 Web 的 HistoryServer 的端口号
historyserver.web.port: 8082

# Comma separated list of directories to monitor for completed jobs.
# 以逗号分隔的目录列表，用于监视已完成的作业
historyserver.archive.fs.dir: hdfs://node01:8020/completed-jobs/

# Interval in milliseconds for refreshing the monitored directories.
# 刷新受监控目录的时间间隔（以毫秒为单位）
historyserver.archive.fs.refresh-interval: 10000

```

- 参数释义
  - jobmanager.archive.fs.dir：flink job运行完成后的日志存放目录
  - historyserver.archive.fs.dir：flink history进程的hdfs监控目录
  - historyserver.web.address：flink history进程所在的主机
  - historyserver.web.port：flink history进程的占用端口
  - historyserver.archive.fs.refresh-interval：刷新受监视目录的时间间隔（以毫秒为单位）。
- 默认启动端口8082：
  - bin/historyserver.sh (start|start-foreground|stop)

#### 序列化

- 首先说一下 Java 原生的序列化方式：
  - 优点：好处是比较简单通用，只要对象实现了 Serializable 接口即可；
  - 缺点：效率比较低，而且如果用户没有指定 serialVersionUID的话，很容易出现作业重新编译后，之前的数据无法反序列化出来的情况（这也是 Spark Streaming Checkpoint 的一个痛点，在业务使用中经常出现修改了代码之后，无法从 Checkpoint 恢复的问题）

对于分布式计算来讲，数据的传输效率非常重要。好的序列化框架可以通过较低的序列化时间和较低的内存占用大大提高计算效率和作业稳定性。

在数据序列化上，Flink 和 Spark 采用了不同的方式 

- Spark 对于所有数据默认采用 Java 原生序列化方式，用户也可以配置使用 Kryo；相比于 Java 原生序列化方式，无论是在序列化效率还是序列化结果的内存占用上，Kryo 则更好一些（Spark 声称一般 Kryo 会比 Java 原生节省 10x 内存占用）；Spark 文档中表示它们之所以没有把 Kryo 设置为默认序列化框架的唯一原因是因为 Kryo 需要用户自己注册需要序列化的类，并且建议用户通过配置开启 Kryo。
- Flink 则是自己实现了一套高效率的序列化方法。

#### 复用对象

```java
stream
    .apply(new WindowFunction<WikipediaEditEvent, Tuple2<String, Long>, String, TimeWindow>() {
        @Override
        public void apply(String userName, TimeWindow timeWindow, Iterable<WikipediaEditEvent> iterable, Collector<Tuple2<String, Long>> collector) throws Exception {
            long changesCount = ...
            // A new Tuple instance is created on every execution
            collector.collect(new Tuple2<>(userName, changesCount));
        }
    }

```

可以看出，apply函数每执行一次，都会新建一个Tuple2类的实例，因此增加了对垃圾收集器的压力。解决这个问题的一种方法是反复使用相同的实例：

```java
stream
        .apply(new WindowFunction<WikipediaEditEvent, Tuple2<String, Long>, String, TimeWindow>() {
    // Create an instance that we will reuse on every call
    private Tuple2<String, Long> result = new Tuple<>();
    @Override
    public void apply(String userName, TimeWindow timeWindow, Iterable<WikipediaEditEvent> iterable, Collector<Tuple2<String, Long>> collector) throws Exception {
        long changesCount = ...
        // Set fields on an existing object instead of creating a new one
        result.f0 = userName;
        // Auto-boxing!! A new Long value may be created
        result.f1 = changesCount;
        // Reuse the same Tuple2 object
        collector.collect(result);
    }
}
```

这种做法其实还间接创建了Long类的实例。

为了解决这个问题，Flink有许多所谓的value class:IntValue、LongValue、StringValue、FloatValue等。下面介绍一下如何使用它们：

```java
stream
        .apply(new WindowFunction<WikipediaEditEvent, Tuple2<String, Long>, String, TimeWindow>() {
    // Create a mutable count instance
    private LongValue count = new LongValue();
    // Assign mutable count to the tuple
    private Tuple2<String, LongValue> result = new Tuple<>("", count);

    @Override
    // Notice that now we have a different return type
    public void apply(String userName, TimeWindow timeWindow, Iterable<WikipediaEditEvent> iterable, Collector<Tuple2<String, LongValue>> collector) throws Exception {
        long changesCount = ...

        // Set fields on an existing object instead of creating a new one
        result.f0 = userName;
        // Update mutable count value
        count.setValue(changesCount);

        // Reuse the same tuple and the same LongValue instance
        collector.collect(result);
    }
}
```

#### 数据倾斜

我们的flink程序中如果使用了keyBy等分组的操作，很容易就出现数据倾斜的情况，数据倾斜会导致整体计算速度变慢，有些子节点甚至接受不到数据，导致分配的资源根本没有利用上。

- 带有窗口的操作
  - 带有窗口的每个窗口中所有数据的分布不平均，某个窗口处理数据量太大导致速率慢
  - 导致Source数据处理过程越来越慢
  - 再导致所有窗口处理越来越慢
- 不带有窗口的操作
  - 有些子节点接受处理的数据很少，甚至得不到数据，导致分配的资源根本没有利用上

**webui体现**

![1623564161947](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/140257-96492.png)

WebUI中Subtasks中打开每个窗口可以看到每个窗口进程的运行情况：如上图，数据分布很不均匀，导致部分窗口数据处理缓慢

优化方式：

- 对key进行均匀的打散处理（hash，加盐等）
- 自定义分区器
- 使用Rebalabce

注意：Rebalance是在数据倾斜的情况下使用，不倾斜不要使用，否则会因为shuffle产生大量的网络开销

- 合理调整并行度：数据过滤之后可以减少并行度，数据合并之后并且在处理之前可以增加并行度，大量小文件写入到hdfs可以减少并行度
- 异步io

**设置并行度的方式以及优先级**

![1623564728315](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623564728315.png)

### Flink-内存管理

#### 问题引入

Flink本身基本是以Java语言完成的，理论上说，直接使用JVM的虚拟机的内存管理就应该更简单方便，但Flink还是单独抽象出了自己的内存管理

因为Flink是为大数据而产生的，而大数据使用会消耗大量的内存，而JVM的内存管理管理设计是兼顾平衡的，不可能单独为了大数据而修改，这对于Flink来说，非常的不灵活，而且频繁GC会导致长时间的机器暂停应用，这对于大数据的应用场景来说也是无法忍受的。

JVM在大数据环境下存在的问题:

1. Java 对象存储密度低。在HotSpot JVM中，每个对象占用的内存空间必须是8的倍数,那么一个只包含 boolean 属性的对象就要占用了16个字节内存：对象头占了8个，boolean 属性占了1个，对齐填充占了7个。而实际上我们只想让它占用1个bit。
2. 在处理大量数据尤其是几十甚至上百G的内存应用时会生成大量对象，Java GC可能会被反复触发，其中Full GC或Major GC的开销是非常大的，GC 会达到秒级甚至分钟级。
3. OOM 问题影响稳定性。OutOfMemoryError是分布式计算框架经常会遇到的问题，当JVM中所有对象大小超过分配给JVM的内存大小时，就会发生OutOfMemoryError错误，导致JVM崩溃，分布式框架的健壮性和性能都会受到影响。

#### 内存划分

![1623564913926](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/145718-143625.png)

注意:Flink的内存管理是在JVM的基础之上,自己进行的管理,但是还没有逃脱的JVM,具体怎么实现,现阶段我们搞不定

1. 网络缓冲区Network Buffers：这个是在TaskManager启动的时候分配的，这是一组用于缓存网络数据的内存，每个块是32K，默认分配2048个，可以通过“taskmanager.network.numberOfBuffers”修改
2. 内存池Memory Manage pool：大量的Memory Segment块，用于运行时的算法（Sort/Join/Shufflt等），这部分启动的时候就会分配。默认情况下，占堆内存的70% 的大小。
3. 用户使用内存Remaining (Free) Heap: 这部分的内存是留给用户代码以及 TaskManager的数据使用的。

#### 堆外内存

除了JVM之上封装的内存管理,还会有个一个很大的堆外内存,用来执行一些IO操作

启动超大内存（上百GB）的JVM需要很长时间，GC停留时间也会很长（分钟级）。

使用堆外内存可以极大地减小堆内存（只需要分配Remaining Heap），使得 TaskManager 扩展到上百GB内存不是问题。

进行IO操作时，使用堆外内存(可以理解为使用操作系统内存)可以zero-copy，使用堆内JVM内存至少要复制一次(需要在操作系统和JVM直接进行拷贝)。

堆外内存在进程间是共享的。

 总结:

Flink相对于Spark,堆外内存该用还是用, 堆内内存管理做了自己的封装,不受JVM的GC影响

#### 序列化和反序列化

Flink除了对堆内内存做了封装之外,还实现了自己的序列化和反序列化机制

序列化与反序列化可以理解为编码与解码的过程。序列化以后的数据希望占用比较小的空间，而且数据能够被正确地反序列化出来。为了能正确反序列化，序列化时仅存储二进制数据本身肯定不够，需要增加一些辅助的描述信息。此处可以采用不同的策略，因而产生了很多不同的序列化方法。

Java本身自带的序列化和反序列化的功能，但是辅助信息占用空间比较大，在序列化对象时记录了过多的类信息。

Flink实现了自己的序列化框架，使用TypeInformation表示每种数据类型，所以可以只保存一份对象Schema信息，节省存储空间。又因为对象类型固定，所以可以通过偏移量存取。

TypeInformation 支持以下几种类型：

- BasicTypeInfo: 任意Java 基本类型或 String 类型。
- BasicArrayTypeInfo: 任意Java基本类型数组或 String 数组。
- WritableTypeInfo: 任意 Hadoop Writable 接口的实现类。
- TupleTypeInfo: 任意的 Flink Tuple 类型(支持Tuple1 to Tuple25)。Flink tuples 是固定长度固定类型的Java Tuple实现。
- CaseClassTypeInfo: 任意的 Scala CaseClass(包括 Scala tuples)。
- PojoTypeInfo: 任意的 POJO (Java or Scala)，例如，Java对象的所有成员变量，要么是 public 修饰符定义，要么有 getter/setter 方法。
- GenericTypeInfo: 任意无法匹配之前几种类型的类。(除了该数据使用kyro序列化.上面的其他的都是用二进制)

![1623565054635](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/145706-528459.png)

针对前六种类型数据集，Flink皆可以自动生成对应的TypeSerializer，能非常高效地对数据集进行序列化和反序列化。对于最后一种数据类型，Flink会使用Kryo进行序列化和反序列化。每个TypeInformation中，都包含了serializer，类型会自动通过serializer进行序列化，然后用Java Unsafe接口(具有像C语言一样的操作内存空间的能力)写入MemorySegments。

![1623565090162](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/145705-247120.png)

![1623565110988](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623565110988.png)

Flink通过自己的序列化和反序列化,可以将数据进行高效的存储,不浪费内存空间

#### 操纵二进制数据

Flink中的group、sort、join 等操作可能需要访问海量数据。以sort为例。

首先，Flink 会从 MemoryManager 中申请一批 MemorySegment，用来存放排序的数据。

这些内存会分为两部分:

一个区域是用来存放所有对象完整的二进制数据。

另一个区域用来存放指向完整二进制数据的指针以及定长的序列化后的key（key+pointer）。

将实际的数据和point+key分开存放有两个目的:

第一，交换定长块（key+pointer）更高效，不用交换真实的数据也不用移动其他key和pointer。

第二，这样做是缓存友好的，因为key都是连续存储在内存中的，可以增加cache命中。 排序会先比较 key 大小，这样就可以直接用二进制的 key 比较而不需要反序列化出整个对象。访问排序后的数据，可以沿着排好序的key+pointer顺序访问，通过 pointer 找到对应的真实数据。

在交换过程中，只需要比较key就可以完成sort的过程，只有key1 == key2的情况，才需要反序列化拿出实际的对象做比较，而比较之后只需要交换对应的key而不需要交换实际的对象

![1623565154769](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/145705-326837.png)

#### 面试

1. 减少full gc时间：因为所有常用数据都在Memory Manager里，这部分内存的生命周期是伴随TaskManager管理的而不会被GC回收。其他的常用数据对象都是用户定义的数据对象，这部分会快速的被GC回收
2. 减少OOM：所有的运行时的内存应用都从池化的内存中获取，而且运行时的算法可以在内存不足的时候将数据写到堆外内存
3. 节约空间：由于Flink自定序列化/反序列化的方法，所有的对象都以二进制的形式存储，降低消耗
4. 高效的二进制操作和缓存友好：二进制数据以定义好的格式存储，可以高效地比较与操作。另外，该二进制形式可以把相关的值，以及hash值，键值和指针等相邻地放进内存中。这使得数据结构可以对CPU高速缓存更友好，可以从CPU的 L1/L2/L3 缓存获得性能的提升,也就是Flink的数据存储二进制格式符合CPU缓存的标准,非常方便被CPU的L1/L2/L3各级别缓存利用,比内存还要快!

### Flink VS Spark

#### 应用场景

![1623568868438](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/15/092029-435975.png)

#### 原理对比

**spark原理**

![1623568901946](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/152146-552965.png)

![1623569060038](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/152430-199766.png)

**flink原理**

![1623569134415](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/152537-947902.png)

#### 运行角色

- Spark Streaming 运行时的角色(standalone 模式)主要有：
  - Master:主要负责整体集群资源的管理和应用程序调度；
  - Worker:负责单个节点的资源管理，driver 和 executor 的启动等；
  - Driver:用户入口程序执行的地方，即 SparkContext 执行的地方，主要是 DAG 生成、stage 划分、task 生成及调度；
  - Executor:负责执行 task，反馈执行状态和执行结果。
- Flink 运行时的角色(standalone 模式)主要有:
  - Jobmanager: 协调分布式执行，他们调度任务、协调 checkpoints、协调故障恢复等。至少有一个 JobManager。高可用情况下可以启动多个 JobManager，其中一个选举为 leader，其余为 standby；
  - Taskmanager: 负责执行具体的 tasks、缓存、交换数据流，至少有一个 TaskManager；
  - Slot: 每个 task slot 代表 TaskManager 的一个固定部分资源，Slot 的个数代表着 taskmanager 可并行执行的 task 数。

#### 生态

![1623569442344](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/145703-623207.png)

#### 运行模型

Spark Streaming 是微批处理，运行的时候需要指定批处理的时间，每次运行 job 时处理一个批次的数据，流程如图所示：

![1623569483393](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623569483393.png)

Flink 是基于事件驱动的，事件可以理解为消息。事件驱动的应用程序是一种状态应用程序，它会从一个或者多个流中注入事件，通过触发计算更新状态，或外部动作对注入的事件作出反应

![1623569524761](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/153213-338705.png)

#### 任务调度原理

Spark 任务调度

Spark Streaming 任务如上文提到的是基于微批处理的，实际上每个批次都是一个 Spark Core 的任务。对于编码完成的 Spark Core 任务在生成到最终执行结束主要包括以下几个部分：

- 构建 DGA 图；
- 划分 stage；
- 生成 taskset；
- 调度 task。

![1623569694360](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/153456-774428.png)

对于 job 的调度执行有 fifo 和 fair 两种模式，Task 是根据数据本地性调度执行的。 假设每个 Spark Streaming 任务消费的 kafka topic 有四个分区，中间有一个 transform操作（如 map）和一个 reduce 操作，如图所示：

![1623569808023](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/153649-307830.png)

假设有两个 executor，其中每个 executor 三个核，那么每个批次相应的 task 运行位置是固定的吗？是否能预测？由于数据本地性和调度不确定性，每个批次对应 kafka 分区生成的 task运行位置并不是固定的。

**Flink 任务调度**

对于 flink 的流任务客户端首先会生成StreamGraph，接着生成 JobGraph，然后将jobGraph 提交给 Jobmanager 由它完成jobGraph 到 ExecutionGraph 的转变，最后由 jobManager 调度执行。

![1623569884621](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/153806-945881.png)

如图所示有一个由 data source、MapFunction和 ReduceFunction 组成的程序，data source 和 MapFunction 的并发度都为 4，而 ReduceFunction 的并发度为 3。一个数据流由 Source-Map-Reduce 的顺序组成，在具有 2 个TaskManager、每个 TaskManager 都有 3 个 Task Slot 的集群上运行。

可以看出 flink 的拓扑生成提交执行之后，除非故障，否则拓扑部件执行位置不变，并行度由每一个算子并行度决定，类似于 storm。而 spark Streaming 是每个批次都会根据数据本地性和资源情况进行调度，无固定的执行拓扑结构。 flink 是数据在拓扑结构里流动执行，而 Spark Streaming 则是对数据缓存批次并行处理。

#### 时间机制对比

流处理的时间

流处理程序在时间概念上总共有三个时间概念：

**处理时间**

- 处理时间是指每台机器的系统时间，当流程序采用处理时间时将使用运行各个运算符实例的机器时间。处理时间是最简单的时间概念，不需要流和机器之间的协调，它能提供最好的性能和最低延迟。然而在分布式和异步环境中，处理时间不能提供消息事件的时序性保证，因为它受到消息传输延迟，消息在算子之间流动的速度等方面制约。

**事件时间**

- 事件时间是指事件在其设备上发生的时间，这个时间在事件进入 flink 之前已经嵌入事件，然后 flink 可以提取该时间。基于事件时间进行处理的流程序可以保证事件在处理的时候的顺序性，但是基于事件时间的应用程序必须要结合 watermark 机制。基于事件时间的处理往往有一定的滞后性，因为它需要等待后续事件和处理无序事件，对于时间敏感的应用使用的时候要慎重考虑。

**注入时间**

- 注入时间是事件注入到 flink 的时间。事件在 source 算子处获取 source 的当前时间作为事件注入时间，后续的基于时间的处理算子会使用该时间处理数据。相比于事件时间，注入时间不能够处理无序事件或者滞后事件，但是应用程序无序指定如何生成 watermark。在内部注入时间程序的处理和事件时间类似，但是时间戳分配和 watermark 生成都是自动的。

![1623569995260](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/154003-388315.png)

**Spark 时间机制**

Spark Streaming 只支持处理时间，Structured streaming 支持处理时间和事件时间，同时支持 watermark 机制处理滞后数据。

**Flink 时间机制**

flink 支持三种时间机制：事件时间，注入时间，处理时间，同时支持 watermark 机制处理滞后数据。

#### 容错机制

![1623570216954](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/154339-488135.png)

#### 窗口

![1623572163119](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/145702-367121.png)



#### 整合kafka

sparkstreaming整合kafka，支持offset自动维护和手动维护，支持动态分区检测，不需要进行配置。

Flink整合kafka，支持offset自动维护和手动维护（一般自动由checkpoint自动维护即可），支持动态分区自动检测，需要进行配置。

#### Back pressure背压/反压

消费者消费的速度低于生产者生产的速度，为了使应用正常，消费者会反馈给生产者来调节生产者生产的速度，以使得消费者需要多少，生产者生产多少。

back pressure 后面一律称为背压。

**Spark Streaming 的背压**

Spark Streaming 跟 kafka 结合是存在背压机制的，目标是根据当前 job 的处理情况来调节后续批次的获取 kafka 消息的条数。为了达到这个目的，Spark Streaming 在原有的架构上加入了一个 RateController，利用的算法是 PID，需要的反馈数据是任务处理的结束时间、调度时间、处理时间、消息条数，这些数据是通过 SparkListener 体系获得，然后通过 PIDRateEsimator 的 compute 计算得到一个速率，进而可以计算得到一个 offset，然后跟限速设置最大消费条数比较得到一个最终要消费的消息最大 offset。

**Flink 的背压**

与 Spark Streaming 的背压不同的是，Flink 1.5 之后实现了自己托管的 credit – based 流控机制，在应用层模拟 TCP 的流控机制，就是每一次 ResultSubPartition 向 InputChannel 发送消息的时候都会发送一个 backlog size 告诉下游准备发送多少消息，下游就会去计算有多少的 Buffer 去接收消息，算完之后如果有充足的 Buffer 就会返还给上游一个 Credit 告知他可以发送消息

jobmanager 针对每一个 task，每 50ms 触发 100 次 Thread.getStackTrace() 调用，求出阻塞的占比。过程如图 16 所示：

![1623573551196](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/145436-87303.png)

**补充**

![1623573085238](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/163129-179167.png)