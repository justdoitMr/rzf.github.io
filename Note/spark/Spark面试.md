<!-- TOC -->

- [Spark整理](#spark整理)
  - [Spark有哪两种算子？](#spark有哪两种算子)
  - [Spark有哪些聚合类的算子,我们应该尽量避免什么类型的算子？](#spark有哪些聚合类的算子我们应该尽量避免什么类型的算子)
  - [RDD创建方式？](#rdd创建方式)
  - [Spark并行度怎么设置比较合适？](#spark并行度怎么设置比较合适)
  - [Spark如何处理不能被序列化的对象？](#spark如何处理不能被序列化的对象)
  - [collect功能是什么，其底层是怎么实现的？](#collect功能是什么其底层是怎么实现的)
  - [Spark on Mesos中，什么是的粗粒度分配，什么是细粒度分配，各自的优点和缺点是什么？](#spark-on-mesos中什么是的粗粒度分配什么是细粒度分配各自的优点和缺点是什么)
  - [Driver的功能是什么？](#driver的功能是什么)
  - [Spark技术栈有哪些组件，每个组件都有什么功能，适合什么应用场景？](#spark技术栈有哪些组件每个组件都有什么功能适合什么应用场景)
  - [Spark中Worker的主要工作是什么？](#spark中worker的主要工作是什么)
  - [Mapreduce和Spark的都是并行计算，那么他们有什么相同和区别？](#mapreduce和spark的都是并行计算那么他们有什么相同和区别)
  - [spark的有几种部署模式，每种模式特点？](#spark的有几种部署模式每种模式特点)
    - [本地模式](#本地模式)
    - [standalone模式](#standalone模式)
    - [Spark on yarn模式](#spark-on-yarn模式)
    - [Spark On Mesos模式](#spark-on-mesos模式)
  - [通常来说，Spark与MapReduce相比，Spark运行效率更高。请说明效率更高来源于Spark内置的哪些机制？](#通常来说spark与mapreduce相比spark运行效率更高请说明效率更高来源于spark内置的哪些机制)
  - [spark有哪些组件？](#spark有哪些组件)
  - [通常来说，Spark 与MapReduce 相比，Spark 运行效率更高。请说明效率更高来源于Spark 内置的哪些机制？](#通常来说spark-与mapreduce-相比spark-运行效率更高请说明效率更高来源于spark-内置的哪些机制)
  - [Spark和Hadoop的应用场景](#spark和hadoop的应用场景)
  - [hadoop和spark的相同点和不同点？](#hadoop和spark的相同点和不同点)
  - [Spark RDD 和 MapReduce2的区别？](#spark-rdd-和-mapreduce2的区别)
  - [Spark中的RDD](#spark中的rdd)
  - [Spark master HA主从切换过程不会影响到集群已有作业的运行，为什么](#spark-master-ha主从切换过程不会影响到集群已有作业的运行为什么)
  - [数据本地性是在哪个环节确定的？](#数据本地性是在哪个环节确定的)
  - [RDD的弹性表现在哪几点？](#rdd的弹性表现在哪几点)
  - [RDD 中reduceBykey 与groupByKey 哪个性能好，为什么？](#rdd-中reducebykey-与groupbykey-哪个性能好为什么)
  - [**为什么要设计宽窄依赖**](#为什么要设计宽窄依赖)
  - [DAG 是什么？](#dag-是什么)
  - [DAG 分 中为什么要划分 Stage ？](#dag-分-中为什么要划分-stage-)
  - [DAG 为 划分为Stage 的算法了解吗？](#dag-为-划分为stage-的算法了解吗)
  - [对于Spark 中的数据倾斜问题你有什么好的方案？](#对于spark-中的数据倾斜问题你有什么好的方案)
  - [Spark中的OOM问题](#spark中的oom问题)
  - [Spar 程序执行，有时候默认为什么会产生很多 task ，怎么修改默认 task 执行个数？](#spar-程序执行有时候默认为什么会产生很多-task-怎么修改默认-task-执行个数)
  - [介绍一下join 操作优化经验？](#介绍一下join-操作优化经验)
  - [简单说一下hadoop和spark的shuffle相同和差异？](#简单说一下hadoop和spark的shuffle相同和差异)
  - [Spark工作机制](#spark工作机制)
  - [spark解决了hadoop的哪些问题？](#spark解决了hadoop的哪些问题)
  - [Spark的优化怎么做？](#spark的优化怎么做)
  - [RDD中reduceBykey与groupByKey哪个性能好，为什么](#rdd中reducebykey与groupbykey哪个性能好为什么)
  - [Spark的Shuffle过程](#spark的shuffle过程)
    - [Shuffle中的任务个数](#shuffle中的任务个数)
    - [reduce端数据的读取](#reduce端数据的读取)
    - [Shuffle过程介绍](#shuffle过程介绍)
    - [shuffer后续优化方向](#shuffer后续优化方向)
  - [Spark的数据本地性有哪几种？](#spark的数据本地性有哪几种)
  - [RDD 持久化或者说是缓存原理？](#rdd-持久化或者说是缓存原理)
  - [Spark为什么要持久化，一般什么场景下要进行persist操作？](#spark为什么要持久化一般什么场景下要进行persist操作)
  - [Checkpoint 检查点机制？](#checkpoint-检查点机制)
  - [Checkpoint 和持久化机制的区别？](#checkpoint-和持久化机制的区别)
  - [Spark 主备切换机制原理知道吗？](#spark-主备切换机制原理知道吗)
  - [Spark Master 使用 Zookeeper 进行 HA ，有哪些源数据保存到Zookeeper 里面？](#spark-master-使用-zookeeper-进行-ha-有哪些源数据保存到zookeeper-里面)
  - [描述Yarn执行一个任务的过程？](#描述yarn执行一个任务的过程)
  - [Spark on Yarn 模式有哪些优点？](#spark-on-yarn-模式有哪些优点)
  - [谈谈你对container的理解？](#谈谈你对container的理解)
  - [介绍parition和block有什么关联关系？](#介绍parition和block有什么关联关系)
  - [Spark应用程序的执行过程是什么？](#spark应用程序的执行过程是什么)
  - [不需要排序的hash shuffle是否一定比需要排序的sort shuffle速度快？](#不需要排序的hash-shuffle是否一定比需要排序的sort-shuffle速度快)
  - [spark.storage.memoryFraction参数的含义,实际生产中如何调优？](#sparkstoragememoryfraction参数的含义实际生产中如何调优)
  - [什么是RDD宽依赖和窄依赖？](#什么是rdd宽依赖和窄依赖)
  - [cache后面能不能接其他算子，它是不是action操作？](#cache后面能不能接其他算子它是不是action操作)
  - [Yarn中的container是由谁负责销毁的，在Hadoop Mapreduce中container可以复用么？](#yarn中的container是由谁负责销毁的在hadoop-mapreduce中container可以复用么)
  - [提交任务时，如何指定Spark Application的运行模式？](#提交任务时如何指定spark-application的运行模式)
  - [不启动Spark集群Master和work服务，可不可以运行Spark程序？](#不启动spark集群master和work服务可不可以运行spark程序)
  - [spark on yarn Cluster 模式下，ApplicationMaster和driver是在同一个进程么？](#spark-on-yarn-cluster-模式下applicationmaster和driver是在同一个进程么)
  - [Executor启动时，资源通过哪几个参数指定？](#executor启动时资源通过哪几个参数指定)
  - [为什么会产生yarn，解决了什么问题，有什么优势？](#为什么会产生yarn解决了什么问题有什么优势)
  - [一个task的map数量由谁来决定？](#一个task的map数量由谁来决定)
  - [导致Executor产生FULL gc 的原因，可能导致什么问题？](#导致executor产生full-gc-的原因可能导致什么问题)
  - [Spark累加器有哪些特点？](#spark累加器有哪些特点)
  - [spark hashParitioner的弊端是什么？](#spark-hashparitioner的弊端是什么)
  - [RangePartitioner分区的原理及特点？](#rangepartitioner分区的原理及特点)
  - [如何理解Standalone模式下，Spark资源分配是粗粒度的？](#如何理解standalone模式下spark资源分配是粗粒度的)
  - [窄依赖父RDD的partition和子RDD的parition是不是都是一对一的关系？](#窄依赖父rdd的partition和子rdd的parition是不是都是一对一的关系)
  - [Hadoop中，Mapreduce操作的mapper和reducer阶段相当于spark中的哪几个算子？](#hadoop中mapreduce操作的mapper和reducer阶段相当于spark中的哪几个算子)
  - [Spark中的HashShufle的有哪些不足？](#spark中的hashshufle的有哪些不足)
  - [spark.default.parallelism这个参数有什么意义，实际生产中如何设置？](#sparkdefaultparallelism这个参数有什么意义实际生产中如何设置)
  - [spark.shuffle.memoryFraction参数的含义，以及优化经验？](#sparkshufflememoryfraction参数的含义以及优化经验)
  - [Spark中standalone模式特点，有哪些优点和缺点？](#spark中standalone模式特点有哪些优点和缺点)
  - [FIFO调度模式的基本原理、优点和缺点？](#fifo调度模式的基本原理优点和缺点)
  - [FAIR调度模式的优点和缺点？](#fair调度模式的优点和缺点)
  - [RDD的数据结构是怎么样的？ 一个RDD对象，包含如下5个核心属性。](#rdd的数据结构是怎么样的-一个rdd对象包含如下5个核心属性)
  - [说说你对Hadoop生态的认识。](#说说你对hadoop生态的认识)
  - [Spark streaming以及基本工作原理？](#spark-streaming以及基本工作原理)
  - [DStream以及基本工作原理？](#dstream以及基本工作原理)
  - [Spark SQL 执行的流程？](#spark-sql-执行的流程)
  - [Spark SQL 到是如何将数据写到Hive 表的？](#spark-sql-到是如何将数据写到hive-表的)

<!-- /TOC -->


## Spark整理

### Spark有哪两种算子？

Transformation（转化）算子和Action（执行）算子。

### Spark有哪些聚合类的算子,我们应该尽量避免什么类型的算子？

在我们的开发过程中，能避免则尽可能避免使用reduceByKey、join、distinct、repartition等会进行
shuffle的算子，尽量使用map类的非shuffle算子。这样的话，没有shuffle操作或者仅有较少shuffle操
作的Spark作业，可以大大减少性能开销。

### RDD创建方式？

**从集合创建rdd**

```scala
val rdd: RDD[Int] = sc.parallelize(Array(1, 2, 1 3, 4, 5, 6, 7, 8))
```

**从外部存储系统的数据集创建rdd**
由外部存储系统的数据集创建RDD包括：本地的文件系统，还有所有Hadoop支持的数据集，比如
HDFS、HBase等

```scala
//读取文件。input为集群路径：hdfs://hadoop102:9000/input
val lineWordRdd: RDD[String] = sc.textFile("input")
```

**从其它rdd创建**

```scala
//创建一个RDD
val rdd: RDD[Int] = sc.makeRDD(1 to 4,2)
//调用map方法，每个元素乘以2
val mapRdd: RDD[Int] = rdd.map(_ * 2)
```

### Spark并行度怎么设置比较合适？

spark并行度，每个core承载2~4个partition,如，32个core，那么64~128之间的并行度，也就是设置
64~128个partion，并行读和数据规模无关，只和内存使用量和cpu使用时间有关。

### Spark如何处理不能被序列化的对象？

将不能序列化的内容封装成object。

### collect功能是什么，其底层是怎么实现的？

driver通过collect把集群中各个节点的内容收集过来汇总成结果，collect返回结果是Array类型的，collect把各个节点上的数据抓过来，抓过来数据是Array型，collect对Array抓过来的结果进行合并，合并后Array中只有一个元素，是tuple类型（KV类型的）的。

### Spark on Mesos中，什么是的粗粒度分配，什么是细粒度分配，各自的优点和缺点是什么？

- 粗粒度：启动时就分配好资源， 程序启动，后续具体使用就使用分配好的资源，不需要再分配资源；
  - 好处：作业特别多时，资源复用率高，适合粗粒度；
  - 不好：容易资源浪费，假如一个job有1000个task，完成了999个，还有一个没完成，那么使用粗粒度，999个资源就会闲置在那里，资源浪费。
- 细粒度分配：用资源的时候分配，用完了就立即回收资源，启动会麻烦一点，启动一次分配一次，会比较麻烦

### Driver的功能是什么？

一个Spark作业运行时包括一个Driver进程，也是作业的主进程，具有main函数，并且有SparkContext
的实例，是程序的入口点，driver负责创建spark context。

功能：负责向master注册信息，负责了作业的调度，负责作业的解析、生成Stage并调度Task到Executor上，包括DAGScheduler，TaskScheduler。

### Spark技术栈有哪些组件，每个组件都有什么功能，适合什么应用场景？

可以画一个这样的技术栈图先，然后分别解释下每个组件的功能和场景

- Spark core：是其它组件的基础，spark的内核，主要包含：有向循环图、RDD、Lingage、Cache、broadcast等，并封装了底层通讯框架，是Spark的基础。
- Spark Streaming：是一个对实时数据流进行高通量、容错处理的流式处理系统，可以对多种数据
  源（如Kafka、Flume、Twitter、Zero和TCP 套接字）进行类似Map、Reduce和Join等复杂操作，将流式计算分解成一系列短小的批处理作业。
- Spark sql：Shark是SparkSQL的前身，Spark SQL的一个重要特点是其能够统一处理关系表和RDD，使得开发人员可以轻松地使用SQL命令进行外部查询，同时进行更复杂的数据分析
- BlinkDB：是一个用于在海量数据上运行交互式 SQL 查询的大规模并行查询引擎，它允许用户通过
  权衡数据精度来提升查询响应时间，其数据的精度被控制在允许的误差范围内。
- MLBase：是Spark生态圈的一部分专注于机器学习，让机器学习的门槛更低，让一些可能并不了解
  机器学习的用户也能方便地使用MLbase。MLBase分为四部分：MLlib、MLI、ML Optimizer和
  MLRuntime。
- GraphX：是Spark中用于图和图并行计算

### Spark中Worker的主要工作是什么？

**主要功能**

管理当前节点内存，CPU的使用状况，接收master分配过来的资源指令，通过ExecutorRunner启动程
序分配任务，worker就类似于包工头，管理分配新进程，做计算的服务，相当于process服务。

> 相当于yarn中的nodemanager角色，负责管理本节点资源。

**需要注意的点**

1. worker不会汇报当前信息给master，worker心跳给master主要只有workid，它不会发送资源信
   息以心跳的方式给mater，master分配的时候就知道work，只有出现故障的时候才会发送资源
2. worker不会运行代码，具体运行的是Executor是可以运行具体appliaction写的业务逻辑代码，操作
   代码的节点，它不会运行程序的代码的

### Mapreduce和Spark的都是并行计算，那么他们有什么相同和区别？

两者都是用mr模型来进行并行计算：

1. hadoop的一个作业称为job，job里面分为map task和reduce task，每个task都是在自己的**进程**中
   运行的，当task结束时，进程也会结束
2. spark用户提交的任务成为application，一个application对应一个SparkContext，app中存在多个
   job，每触发一次action操作就会产生一个job。这些job可以并行或串行执行，每个job中有多个stage，stage是shuffle过程中DAGSchaduler通过RDD之间的依赖关系划分job而来的，每个stage里面有多个task，组成taskset，有TaskSchaduler分发到各个executor中执行，executor的生命周期是和app一样的，即使没有job运行也是存在的，所以task可以快速启动读取内存进行计算
3. hadoop的job只有map和reduce操作，表达能力比较欠缺而且在mr过程中会重复的读写hdfs，造
   成大量的io操作，多个job需要自己管理关系
4. spark的迭代计算都是在内存中进行的，API中提供了大量的RDD操作如join，groupby等，而且通
   过DAG图可以实现良好的容错

### spark的有几种部署模式，每种模式特点？

#### 本地模式

Spark不一定非要跑在hadoop集群，可以在本地，起多个线程的方式来指定。将Spark应用以多线程的方式直接运行在本地，一般都是为了方便调试，本地模式分三类

- local：只启动一个executor
- local[k]：启动k个executor
- local[*]：启动跟cpu数目相同的 executor

#### standalone模式

分布式部署集群，自带完整的服务，资源管理和任务监控是Spark自己监控，这个模式也是其他模式的
基础。

#### Spark on yarn模式

分布式部署集群，资源和任务监控交给yarn管理，但是目前仅支持粗粒度资源分配方式，包含cluster和
client运行模式:

- cluster适合生产，driver运行在集群子节点，具有容错功能，
- client适合调试，dirver运行在客户端。

#### Spark On Mesos模式

官方推荐这种模式（当然，原因之一是血缘关系）。正是由于Spark开发之初就考虑到支持Mesos，因
此，目前而言，Spark运行在Mesos上会比运行在YARN上更加灵活，更加自然。用户可选择两种调度模
式之一运行自己的应用程序：

1. 粗粒度模式（Coarse-grained Mode）：每个应用程序的运行环境由一个Dirver和若干个Executor组成，其中，每个Executor占用若干资源，内部可运行多个Task（对应多少个“slot”）。应用程序的各个任务正式运行之前，需要将运行环境中的资源全部申请好，且运行过程中要一直占用这些资源，即使不用，最后程序运行结束后，回收这些资源。
2. 细粒度模式（Fine-grained Mode）：鉴于粗粒度模式会造成大量资源浪费，Spark On Mesos还提供了另外一种调度模式：细粒度模式，这种模式类似于现在的云计算，思想是按需分配。

### 通常来说，Spark与MapReduce相比，Spark运行效率更高。请说明效率更高来源于Spark内置的哪些机制？

spark是借鉴了Mapreduce,并在其基础上发展起来的，继承了其分布式计算的优点并进行了改进，spark生态更为丰富，功能更为强大，性能更加适用范围广，mapreduce更简单，稳定性好。主要区别

1. spark把运算的中间数据(shuffle阶段产生的数据)存放在内存，迭代计算效率更高，mapreduce的中间结果需要落地，保存到磁盘，这也是最重要的一点原因。
2. Spark容错性高，它通过**弹性分布式数据集RDD来实现高效容错**，RDD是一组分布式的存储在节点内存中的**只读性**的数据集，**这些集合是弹性的，某一部分丢失或者出错，可以通过整个数据集的计算流程的血缘关系来实现重建，mapreduce的容错只能重新计算**.，其中弹性还体现在中间的数据可以存储在内存中，如果内存不足的话会存放到磁盘上面。
3. Spark更通用，**提供了transformation和action这两大类的多功能api**，另外还有流式处sparkstreaming模块、图计算等等，mapreduce只提供了map和reduce两种操作，流计算及其他的模块支持比较缺乏
4. Spark框架和生态更为复杂，有RDD，血缘lineage（依赖链）、执行时的有向无环图DAG,stage划分等，很多时候spark作业都需要根据不同业务场景的需要进行调优以达到性能要求，mapreduce框架及其生态相对较为简单，对性能的要求也相对较弱，运行较为稳定，适合长期后台运行。
5. Spark计算框架对内存的利用和运行的并行度比mapreduce高，Spark运行容器为executor，内部ThreadPool中线程运行一个Task,mapreduce在线程内部运行container，container容器分类为MapTask和ReduceTask.程序运行并行度高
6. Spark对于executor的优化，在JVM虚拟机的基础上对内存弹性利用：storage memory与Execution memory的弹性扩容，使得内存利用效率更高

### spark有哪些组件？

1、Driver：是一个JVM Process 进程，编写的Spark应用程序就运行在Driver上，由Driver进程执行；

2）、Master(ResourceManager)：是一个JVM Process 进程，主要负责资源的调度和分配，并进行集群的监控等职责；

3）、Worker(NodeManager)：是一个JVM Process 进程，一个Worker运行在集群中的一台服务器上，主要负责两个职责，一个是用自己的内存存储RDD的某个或某些partition；另一个是启动其他进程和线程（Executor），对RDD上的partition进行并行的处理和计算。

4）、Executor：是一个JVM Process 进程，一个Worker(NodeManager)上可以运行多个Executor，Executor通过启动多个线程（task）来执行对RDD的partition进行并行计算，也就是执行我们对RDD定义的例如map、flatMap、reduce等算子操作。

5）、spark context：控制整个application的生命周期，包括dagsheduler和task scheduler等组件。

6）、client：用户提交程序的入口。

### 通常来说，Spark 与MapReduce 相比，Spark 运行效率更高。请说明效率更高来源于Spark 内置的哪些机制？

1. 基于内存计算，减少低效的磁盘交互；
2. 高效的调度算法，基于 DAG；
3. 容错机制 Linage。

> 重点部分就是 DAG 和 Lingae

### Spark和Hadoop的应用场景

Hadoop/MapReduce和Spark最适合的都是做**离线型**的数据分析，但Hadoop特别适合是单次分析的数据量“很大”的情景，而Spark则适用于数据量不是很大的情景。

- 一般情况下，对于中小互联网和企业级的大数据应用而言，单次分析的数量都不会“很大”，因此可以优先考虑使用Spark。
- 业务通常认为Spark更适用于机器学习之类的“迭代式”应用，80GB的压缩数据（解压后超过200GB），10个节点的集群规模，跑类似“sum+group-by”的应用，MapReduce花了5分钟，而spark只需要2分钟。

### hadoop和spark的相同点和不同点？

**Hadoop底层使用MapReduce计算架构，只有map和reduce两种操作，表达能力比较欠缺，而且在MR过程中会重复的读写hdfs，造成大量的磁盘io读写操作**，所以适合高时延环境下批处理计算的应用；

**Spark是基于内存的分布式计算架构，提供更加丰富的数据集操作类型，主要分成转化操作和行动操作**，包括map、reduce、filter、flatmap、groupbykey、reducebykey、union和join等，数据分析更加快速，所以适合低时延环境下计算的应用；

**spark与hadoop最大的区别在于迭代式计算模型**。基于mapreduce框架的Hadoop主要分为map和reduce两个阶段，两个阶段完了就结束了，所以在一个job里面能做的处理很有限；spark计算模型是基于内存的迭代式计算模型，可以分为n个阶段，根据用户编写的RDD算子和程序，在处理完一个阶段后可以继续往下处理很多个阶段，而不只是两个阶段。所以spark相较于mapreduce，计算模型更加灵活，可以提供更强大的功能。

但是spark也有劣势，由于spark基于内存进行计算，虽然开发容易，但是真正面对大数据的时候，在没有进行调优的情况下下，可能会出现各种各样的问题，**比如OOM内存溢出等情况**，导致spark程序可能无法运行起来，而mapreduce虽然运行缓慢，但是至少可以慢慢运行完。

### Spark RDD 和 MapReduce2的区别？

1. mr2只有2个阶段，数据需要大量访问磁盘，数据来源相对单一 ,spark RDD ,可以无数个阶段进行迭
   代计算，数据来源非常丰富，数据落地介质也非常丰富spark计算基于内存
2. MapReduce2需要频繁操作磁盘IO，需要大家明确的是如果是SparkRDD的话，你要知道每一种数
   据来源对应的是什么，RDD从数据源加载数据，将数据放到不同的partition针对这些partition中的数据
   进行迭代式计算计算完成之后，落地到不同的介质当中。

> 本质上是计算模型的不同。

### Spark中的RDD

rdd 分布式弹性数据集，简单的理解成一种数据结构，是 spark 框架上的通用货币。所有算子都是基于 rdd 来执行的，不同的场景会有不同的 rdd 实现类，但是都可以进行互相转换。rdd 执行过程中会形成 dag 图，然后形成 lineage保证容错性等。从物理的角度来看 rdd 存储的是 block 和 node 之间的映射。

RDD 是 spark 提供的**核心抽象**，全称为弹性分布式数据集。RDD 在逻辑上是一个 hdfs 文件，在抽象上是一种**元素集合**，包含了数据。它是被分区的，分为多个分区，每个分区分布在集群中的不同结点上，从而让 RDD 中的数据可以被并行操作（分布式数据集），比如有个 RDD 有 90W 数据，3 个 partition，则每个分区上有 30W 数据。RDD通常通过 Hadoop 上的文件，即 HDFS 或者 HIVE 表来创建，还可以通过应用程序中的集合来创建；RDD 最重要的特性就是**容错性**，可以自动从节点失败中恢复过来。即如果某个结点上的 RDD partition 因为节点故障，导致数据丢失，那么 RDD 可以通过自己的数据来源重新计算该 partition。这一切对使用者都是透明的。

**RDD 的数据默认存放在内存中，但是当内存资源不足时，spark 会自动将 RDD 数据写入磁盘**。比如某结点内存只能处理 20W 数据，那么这 20W 数据就会放入内存中计算，剩下 10W 放到磁盘中。**RDD 的弹性体现在于 RDD 上自动进行内存和磁盘之间权衡和切换的机制。**

### Spark master HA主从切换过程不会影响到集群已有作业的运行，为什么

不会的。

因为程序在运行之前，已经申请过资源了，driver和Executors通讯，不需要和master进行通讯的。

### 数据本地性是在哪个环节确定的？

具体的task运行在那他机器上，DAG划分stage的时候确定的，我们尽量移动计算，而不是移动数据，这样可以减少数据在节点之间传输的延迟。

### RDD的弹性表现在哪几点？

**存储的弹性：**

内存与磁盘的自动切换，Spark优先把数据放到内存中，如果内存放不下，就会放到磁盘里面，程序进行自动的存储切换

**容错的弹性：**

数据丢失可以自动恢复，在RDD进行转换和动作的时候，会形成RDD的Lineage依赖链，当某一个RDD失效的时候，可以通过重新计算上游的RDD来重新生成丢失的RDD数据，还可以设置检查点。

**计算的弹性：**计算出错重试机制

1. Task如果失败会自动进行特定次数的重试，RDD的计算任务如果运行失败，会自动进行任务的重新计算，默认次数是4次。
2. Stage如果失败会自动进行特定次数的重试，如果Job的某个Stage阶段计算失败，框架也会自动进行任务的重新计算，默认次数也是4次。

**分片的弹性：**

可根据需要重新分片，可以根据业务的特征，动态调整数据分片的个数，提升整体的应用执行效率

**Checkpoint和Persist可主动或被动触发**

RDD可以通过Persist持久化将RDD缓存到内存或者磁盘，当再次用到该RDD时直接读取就行。也可以将RDD进行检查点，检查点会将数据存储在HDFS中，该RDD的所有父RDD依赖都会被移除。

### RDD 中reduceBykey 与groupByKey 哪个性能好，为什么？

reduceByKey ：reduceByKey 会在结果发送至 reducer 之前会对每个 mapper 在本地进行 merge，有点类似于在 MapReduce 中的 combiner。这样做的好处在于，在 map 端进行一次 reduce 之后，数据量会大幅度减小，从而减小传输，保证reduce 端能够更快的进行结果计算。

groupByKey ：groupByKey 会对每一个 RDD 中的 value 值进行聚合形成一个序列(Iterator)，此操作发生在 reduce 端，所以势必会将所有的数据通过网络进行传输，造成不必要的浪费。同时如果数据量十分大，可能还会造成OutOfMemoryError。

所以在进行大量数据的 reduce 操作时候建议使用 reduceByKey。不仅可以提高速度，还可以防止使用 groupByKey 造成的内存溢出问题。

### **为什么要设计宽窄依赖**

### DAG 是什么？

DAG(Directed Acyclic Graph 有向无环图)指的是数据转换执行的过程，有方向，无闭环(其实就是 RDD 执行的流程)；

原始的 RDD 通过一系列的转换操作就形成了 DAG 有向无环图，任务执行时，可以按照 DAG 的描述，执行真正的计算(数据被操作的一个过程)。

> 本质上描述的是数据的流向。

###  DAG 分 中为什么要划分 Stage ？

并行计算 。

想象一个场景，如果spark中没有shuffle操作，那么在遇到这种一个分区的数据发往多个分区的时候，可能有的分区计算还没有完成，所以在下游的分区位置，就需要阻塞主，不能向下执行，那么在上游的算子中，各个算子的执行过程我们不能预知，所以为了优化执行，将数据流划分为多个阶段，每一个阶段内可能有多个分区的数据，分区和分区之间就可以形成一个pipline操作，优化执行。

一个复杂的业务逻辑如果有 shuffle，那么就意味着前面阶段产生结果后，才能执行下一个阶段，即下一个阶段的计算要依赖上一个阶段的数据。那么我们按照shuffle 进行划分(也就是按照宽依赖就行划分)，就可以将一个 DAG 划分成多个 Stage/阶段，在同一个 Stage 中，会有多个算子操作，可以形成一个
pipeline 流水线，流水线内的多个平行的分区可以并行执行。

### DAG 为 划分为Stage 的算法了解吗？

核心算法：回溯算法

从后往前回溯/ / 反向解析，遇到窄依赖加入本 Stage，遇见宽依赖进行Stage 切分 。

Spark 内核会从触发 Action 操作的那个 RDD 开始 从后往前推 ，首先会为最后一个 RDD 创建一个 Stage，然后继续倒推，如果发现对某个 RDD 是宽依赖，那么就会将宽依赖的那个 RDD 创建一个新的 Stage，那个 RDD 就是新的 Stage的最后一个 RDD。然后依次类推，继续倒推，根据窄依赖或者宽依赖进行 Stage的划分，直到所有的 RDD 全部遍历完成为止。

###  对于Spark 中的数据倾斜问题你有什么好的方案？

1. 前提是定位数据倾斜，是 OOM 了，还是任务执行缓慢，看日志，看 WebUI。
2. 解决方法，有多个方面:
  1. 避免不必要的 shuffle，如使用广播小表的方式，将 reduce-side-join提升为 map-side-join
  2.  改变并行度，可能并行度太少了，导致个别 task 数据压力大
  3. 两阶段聚合，先局部聚合，再全局聚合
  4. 自定义 paritioner，分散 key 的分布，使其更加均匀

### Spark中的OOM问题

map 类型的算子执行中内存溢出如 flatMap，mapPatitions

- 原因：map 端过程产生大量对象导致内存溢出：这种溢出的原因是在单个map 中产生了大量的对象导致的针对这种问题。

**解决方案：**

- 增加堆内内存。
- 在不增加内存的情况下，可以减少每个 Task 处理数据量，使每个 Task产生大量的对象时，Executor 的内存也能够装得下。具体做法可以在会
- 产生大量对象的 map 操作之前调用 repartition 方法，分区成更小的块传入 map。

shuffle 后内存溢出如 join，reduceByKey，repartition。

- shuffle 内存溢出的情况可以说都是 shuffle 后，单个文件过大导致的。在 shuffle 的使用，需要传入一个 partitioner，大部分 Spark 中的shuffle 操作，默认的 partitioner 都是 HashPatitioner，默认值是父
  RDD 中最大的分区数．这个参数 spark.default.parallelism 只对HashPartitioner 有效．如果是别的 partitioner 导致的 shuffle 内存溢出就需要重写 partitioner 代码了

driver 内存溢出

- 用户在 Dirver 端口生成大对象，比如创建了一个大的集合数据结构。解决方案：将大对象转换成 Executor 端加载，比如调用 sc.textfile 或者评估大对象占用的内存，增加 dirver 端的内存
- 从 Executor 端收集数据（collect）回 Dirver 端，建议将 driver 端对 collect 回来的数据所作的操作，转换成 executor 端 rdd 操作。

### Spar 程序执行，有时候默认为什么会产生很多 task ，怎么修改默认 task 执行个数？

输入数据有很多 task，尤其是有很多小文件的时候，有多少个输入 block就会有多少个 task 启动；

spark 中有 partition 的概念，每个 partition 都会对应一个 task，task 越多，在处理大规模数据的时候，就会越有效率。不过 task 并不是越多越好，如果平时测试，或者数据量没有那么大，则没有必要 task 数量太多。

参数可以通过 spark_home/conf/spark-default.conf 配置文件设置:

~~~ java
针对 spark sql 的 task 数量： spark.sql.shuffle.partitions=50
  非 spark sql 程序设置生效： spark.default.parallelism=10
~~~

所以说分区数量就可以影响Task的数量，所以我们可以通过再程序中设置分区的数量，可以从四个层面设置：

- 代码层面
- 配置文件层面
- 客户端提交时候设置
- 环境层面

> 上面的设置，优先级依次降低

### 介绍一下join 操作优化经验？

join 其实常见的就分为两类： map-side join 和 reduce-side join 。

当大表和小表 join 时，用 map-side join 能显著提高效率。那么在spark中，我们优先使用shuffle hashjoin和boardcast join这两种方法，对于达标join小表和小表join小表非常号，但是如果join的是两张大表的话，我们就需要使用sort shuffle join了。

将多份数据进行关联是数据处理过程中非常普遍的用法，不过在分布式计算系统中，这个问题往往会变的非常麻烦，因为框架提供的 join 操作一般会将所有数据根据 key 发送到所有的 reduce 分区中去，也就是 shuffle 的过程。造成大量的网络以及磁盘 IO 消耗，运行效率极其低下，这个过程一般被称为reduce-side-join。

如果其中有张表较小的话，我们则可以自己实现在 map 端实现数据关联，跳过大量数据进行 shuffle 的过程，运行时间得到大量缩短，根据不同数据可能会有几倍到数十倍的性能提升。在大数据量的情况下，join 是一中非常昂贵的操作，需要在 join 之前应尽可能的先缩小数据量。

对于缩小数据量，有以下几条建议 ：
1. 若两个 RDD 都有重复的 key，join 操作会使得数据量会急剧的扩大。所有，最好先使用 distinct 或者 combineByKey 操作来减少 key 空间或者用 cogroup 来处理重复的 key，而不是产生所有的交叉结果。在combine 时，进行机智的分区，可以避免第二次 shuffle。
2. 如果只在一个 RDD 出现，那你将在无意中丢失你的数据。所以使用外连接会更加安全，这样你就能确保左边的 RDD 或者右边的 RDD 的数据完整性，在 join 之后再过滤数据。
3. 如果我们容易得到 RDD 的可以的有用的子集合，那么我们可以先用filter 或者 reduce，如何在再用 join。

### 简单说一下hadoop和spark的shuffle相同和差异？

从 high-level 的角度来看，两者并没有大的差别。 都是将 mapper（Spark 里是 ShuffleMapTask）
的输出进行 partition，不同的 partition 送到不同的 reducer（Spark 里 reducer 可能是下一个 stage
里的 ShuffleMapTask，也可能是 ResultTask）。Reducer 以内存作缓冲区，边 shuffle 边 aggregate
数据，等到数据 aggregate 好以后进行 reduce() （Spark 里可能是后续的一系列操作）。

从 low-level 的角度来看，两者差别不小。 Hadoop MapReduce 是 sort-based，进入 combine()和 reduce() 的 records 必须先 sort，**分区内部数据有序**。这样的好处在于 combine/reduce() 可以处理大规模的数据，因为其输入数据可以通过外排得到（mapper 对每段数据先做排序，reducer 的 shuffle 对排好序的每段数据做归并）。目前的 Spark 默认选择的是 hash-based，通常使用 HashMap 来对 shuffle 来的数据进行 aggregate，不会对数据进行提前排序。如果用户需要经过排序的数据，那么需要自己调用类似
sortByKey() 的操作；如果你是Spark 1.1的用户，可以将spark.shuffle.manager设置为sort，则会对数据进行排序。在Spark 1.2中，sort将作为默认的Shuffle实现。

从实现角度来看，两者也有不少差别。 Hadoop MapReduce 将处理流程划分出明显的几个阶段：map(), spill, merge, shuffle, sort, reduce() 等。每个阶段各司其职，可以按照过程式的编程思想来逐一实现每个阶段的功能。在 Spark 中，没有这样功能明确的阶段，只有不同的 stage 和一系列的transformation()，所以 spill, merge, aggregate 等操作需要蕴含在 transformation() 中。

如果我们将 map 端划分数据、持久化数据的过程称为 shuffle write，而将 reducer 读入数据、aggregate 数据的过程称为 shuffle read。那么在 Spark 中，问题就变为怎么在 job 的逻辑或者物理执行图中加入 shuffle write 和 shuffle read的处理逻辑？以及两个处理逻辑应该怎么高效实现？

Shuffle write由于不要求数据有序，shuffle write 的任务很简单：将数据 partition 好，并持久化。之所以要持久化，一方面是要减少内存存储空间压力，另一方面也是为了 fault-tolerance。

### Spark工作机制

用户在client端提交作业后，会由Driver运行main方法并创建spark context上下文。执行add算子，每遇到一个行动算子，就会触发一个job然后形成dag图输入dagscheduler，按照add之间的依赖关系划分stage输入task scheduler。task scheduler会将stage划分为task set分发到各个节点的executor中执行。

![1635393855062](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/125451-838441.png)

1. 构建Application的运行环境，Driver创建一个SparkContext

~~~ java
val conf = new SparkConf();
conf.setAppName("test")
conf.setMaster("local")
val sc = new SparkContext(conf)
~~~

2. SparkContext向资源管理器（Standalone、Mesos、Yarn）申请Executor资源，资源管理器启动
   StandaloneExecutorbackend（Executor）
3. Executor向SparkContext申请Task
4. SparkContext将应用程序分发给Executor
5. SparkContext就建成DAG图，DAGScheduler将DAG图解析成Stage，每个Stage有多个task，形成
   taskset发送给task Scheduler，由task Scheduler将Task发送给Executor运行
6. Task在Executor上运行，运行完释放所有资源

### spark解决了hadoop的哪些问题？

**从API抽象层次来看**

- MR：抽象层次低，需要使用手工代码来完成程序编写，使用上难以上手；
- Spark：Spark采用RDD计算模型，简单容易上手。

**从数据的处理角度看**

- MR：只提供map和reduce两个操作，表达能力欠缺；
- Spark：Spark采用更加丰富的算子模型，包括map、flatmap、groupbykey、reducebykey等；

**从作业的角度看**

- MR：一个job只能包含map和reduce两个阶段，复杂的任务需要包含很多个job，这些job之间的管理以来需要开发者自己进行管理；
- Spark：Spark中一个job可以包含多个转换操作，在调度时可以生成多个stage，而且如果多个map操作的分区不变，是可以放在同一个task里面去执行；

**从数据的存储过程看**

- MR：中间结果存放在hdfs中，需要io，所以导致延迟很高。
- Spark：Spark的中间结果一般存在内存中，只有当内存不够了，才会存入本地磁盘，而不是hdfs；

**是否有shuffle操作**

- MR：只有等到所有的map task执行完毕后才能执行reduce task；中间的过程是shuffle操作。
- Spark：Spark中分区相同的转换构成流水线在一个task中执行，分区不同的需要进行shuffle操作，被划分成不同的stage需要等待前面的stage执行完才能执行。

**是否是批处理**

- MR：只适合batch批处理，时延高，对于交互式处理和实时处理支持不够；
- Spark：Spark streaming可以将流拆成时间间隔的batch进行处理，实时计算。

### Spark的优化怎么做？

Spark调优比较复杂，但是大体可以分为三个方面来进行

**平台层面的调优**：防止不必要的jar包分发，提高数据的本地性，选择高效的存储格式如parquet。

**应用程序层面的调优**：过滤操作符的优化降低过多小任务，降低单条记录的资源开销，处理数据倾
斜，复用RDD进行缓存，作业并行化执行等等。

**JVM层面的调优**：设置合适的资源量，设置合理的JVM，启用高效的序列化方法如kyro，增大off
head内存等等。

### RDD中reduceBykey与groupByKey哪个性能好，为什么

**reduceByKey**：reduceByKey会在结果发送至reducer之前会对每个mapper在本地进行merge，有点类似于在MapReduce中的combiner。这样做的好处在于，在map端进行一次reduce之后，数据量会大幅度减小，从而减小传输，保证reduce端能够更快的进行结果计算。

**groupByKey**：groupByKey会对每一个RDD中的value值进行聚合形成一个序列(Iterator)，此操作发生在reduce端，所以势必会将所有的数据通过网络进行传输，造成不必要的浪费。同时如果数据量十分大，可能还会造成OutOfMemoryError。

所以在进行大量数据的reduce操作时候建议使用reduceByKey。不仅可以提高速度，还可以防止使用groupByKey造成的内存溢出问题。

### Spark的Shuffle过程

**Shuffle核心要点**：ShuffleMapStage与ResultStage

![1635394763877](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/125455-204205.png)

在划分stage时，最后一个stage称为FinalStage，它本质上是一个ResultStage对象，前面的所有stage
被称为ShuffleMapStage。

ShuffleMapStage的结束伴随着shuffle文件的写磁盘。

ResultStage基本上对应代码中的action算子，即将一个函数应用在RDD的各个partition的数据集上，
意味着一个job的运行结束。

#### Shuffle中的任务个数

我们知道，Spark Shuffle分为map阶段和reduce阶段，或者称之为ShuffleRead阶段和ShuffleWrite阶
段，那么对于一次Shuffle，map过程和reduce过程都会由若干个task来执行，那么map task和reduce
task的数量是如何确定的呢？

假设Spark任务从HDFS中读取数据，那么初始RDD分区个数由该文件的split个数决定，也就是一个split
对应生成的RDD的一个partition，我们假设初始partition个数为N。我们也可以在程序种设置分区的个数。

初始RDD经过一系列算子计算后（假设没有执行repartition和coalesce算子进行重分区，则分区个数不
变，仍为N，如果经过重分区算子，那么分区个数变为M），我们假设分区个数不变，当执行到Shuffle
操作时，map端的task个数和partition个数一致，即map task为N个。

reduce端的stage默认取spark.default.parallelism这个配置项的值作为分区数，如果没有配置，则以
map端的最后一个RDD的分区数作为其分区数（也就是N），那么分区数就决定了reduce端的task的个
数。

#### reduce端数据的读取

根据stage的划分我们知道，map端task和reduce端task不在相同的stage中，map task位于
ShuffleMapStage，reduce task位于ResultStage，map task会先执行，那么后执行的reduce task如
何知道从哪里去拉取map task落盘后的数据呢？

1. map task执行完毕后会将计算状态以及磁盘小文件位置等信息封装到MapStatus对象中，然后由本
   进程中的MapOutPutTrackerWorker对象将mapStatus对象发送给Driver进程的
   MapOutPutTrackerMaster对象；
2. 在reduce task开始执行之前会先让本进程中的MapOutputTrackerWorker向Driver进程中的
   MapoutPutTrakcerMaster发动请求，请求磁盘小文件位置信息；
3. 当所有的Map task执行完毕后，Driver进程中的MapOutPutTrackerMaster就掌握了所有的磁盘小
   文件的位置信息。此时MapOutPutTrackerMaster会告诉MapOutPutTrackerWorker磁盘小文件的位置
   信息；
4. 完成之前的操作之后，由BlockTransforService去Executor0所在的节点拉数据，默认会启动五个子
   线程。每次拉取的数据量不能超过48M（reduce task每次最多拉取48M数据，将拉来的数据存储到
   Executor内存的20%内存中）。

#### Shuffle过程介绍

**Shuffle Writer**

Spark丰富了任务类型，有些任务之间数据流转不需要通过Shuffle，但是有些任务之间还是需要通过
Shuffle来传递数据，比如wide dependency的group by key。

Spark中需要Shuffle输出的Map任务会为每个Reduce创建对应的bucket，Map产生的结果会根据设置
的partitioner得到对应的bucketId，然后填充到相应的bucket中去。**每个Map的输出结果可能包含所有**
**的Reduce所需要的数据，所以每个Map会创建R个bucket（R是reduce的个数），M个Map总共会创建**
**M*R个bucket。**

Map创建的bucket其实对应磁盘上的一个文件，Map的结果写到每个bucket中其实就是写到那个磁盘
文件中，这个文件也被称为blockFile，是Disk Block Manager管理器通过文件名的Hash值对应到本地
目录的子目录中创建的。每个Map要在节点上创建R个磁盘文件用于结果输出，Map的结果是直接输出
到磁盘文件上的，100KB的内存缓冲是用来创建Fast Buffered OutputStream输出流。这种方式一个问
题就是Shuffle文件过多。

针对上述Shuffle过程产生的文件过多问题，Spark有另外一种改进的Shuffle过程：consolidation
Shuffle，以期显著减少Shuffle文件的数量。在consolidation Shuffle中每个bucket并非对应一个文
件，而是对应文件中的一个segment部分。Job的map在某个节点上第一次执行，为每个reduce创建
bucket对应的输出文件，把这些文件组织成ShuffleFileGroup，当这次map执行完之后，这个ShuffleFileGroup可以释放为下次循环利用；当又有map在这个节点上执行时，不需要创建新的bucket
文件，而是在上次的ShuffleFileGroup中取得已经创建的文件继续追加写一个segment；当前次map还
没执行完，ShuffleFileGroup还没有释放，这时如果有新的map在这个节点上执行，无法循环利用这个
ShuffleFileGroup，而是只能创建新的bucket文件组成新的ShuffleFileGroup来写输出。

比如一个Job有3个Map和2个reduce：

(1) 如果此时集群有3个节点有空槽，每个节点空闲了一个core，则3个Map会调度到这3个节点上执行，每个Map都会创建2个Shuffle文件，总共创建6个Shuffle文件；

(2) 如果此时集群有2个节点有空槽，每个节点空闲了一个core，则2个Map先调度到这2个节点上执行，
每个Map都会创建2个Shuffle文件，然后其中一个节点执行完Map之后又调度执行另一个Map，则这个
Map不会创建新的Shuffle文件，而是把结果输出追加到之前Map创建的Shuffle文件中；总共创建4个
Shuffle文件；

(3) 如果此时集群有2个节点有空槽，一个节点有2个空core一个节点有1个空core，则一个节点调度2个Map一个节点调度1个Map，调度2个Map的节点上，一个Map创建了Shuffle文件，后面的Map还是会创建新的Shuffle文件，因为上一个Map还正在写，它创建的ShuffleFileGroup还没有释放；总共创建6个Shuffle文件。

**Shuffle Fetcher**

Reduce去拖Map的输出数据，Spark提供了两套不同的拉取数据框架：

1. 通过socket连接去取数据
2. 使用netty框架去取数据

每个节点的Executor会创建一个BlockManager，其中会创建一个BlockManagerWorker用于响应请
求。当Reduce的GET_BLOCK的请求过来时，读取本地文件将这个blockId的数据返回给Reduce。如果
使用的是Netty框架，BlockManager会创建ShuffleSender用于发送Shuffle数据。并不是所有的数据都是通过网络读取，对于在本节点的Map数据，Reduce直接去磁盘上读取而不再通过网络框架。

Reduce拖过来数据之后以什么方式存储呢？

Spark Map输出的数据没有经过排序，Spark Shuffle过来的数据也不会进行排序，Spark认为Shuffle过程中的排序不是必须的，并不是所有类型的Reduce需要的数据都需要排序，强制地进行排序只会增加Shuffle的负担。Reduce拖过来的数据会放在一个HashMap中，HashMap中存储的也是<key, value>对，key是Map输出的key，Map输出对应这个key的所有value组成HashMap的value。Spark将Shuffle取过来的每一个<key, value>对插入或者更新到HashMap中，来一个处理一个。HashMap全部放在内存中。

Shuffle取过来的数据全部存放在内存中，对于数据量比较小或者已经在Map端做过合并处理的Shuffle
数据，占用内存空间不会太大，但是对于比如group by key这样的操作，Reduce需要得到key对应的所
有value，并将这些value组一个数组放在内存中，这样当数据量较大时，就需要较多内存。

当内存不够时，要不就失败，要不就用老办法把内存中的数据移到磁盘上放着。Spark意识到在处理数
据规模远远大于内存空间时所带来的不足，引入了一个具有外部排序的方案。Shuffle过来的数据先放在
内存中，当内存中存储的<key, value>对超过1000并且内存使用超过70%时，判断节点上可用内存如果
还足够，则把内存缓冲区大小翻倍，如果可用内存不再够了，则把内存中的<key, value>对排序然后写
到磁盘文件中。最后把内存缓冲区中的数据排序之后和那些磁盘文件组成一个最小堆，每次从最小堆中
读取最小的数据，这个和MapReduce中的merge过程类似。

**MapReduce和Spark的Shuffle过程对比**

|         | MapReduce                                         | Spark                                                        |
| ------- | ------------------------------------------------- | ------------------------------------------------------------ |
| collect | 在内存中构造了一块数据结构用于map输出的缓冲       | 没有在内存中构造一块数据结构用于map输出的缓冲，而是直接把输出写到磁盘文件 |
| sort    | map输出的数据有排序                               | map输出的数据没有排序                                        |
| merge   | 对磁盘上的多个spill文件最后进行合并成一个输出文件 | 在map端没有merge过程，在输出时直接是对应一个<reduce的数据写到一个文件中，这些文件同时存在并发写，最后不需要合并成一个 |
| copy框架 | jetty | netty或者直接socket流 |
| merge<br/>sort | 最后会对磁盘文件和内存中的数据进行合并排序                        | 对于采用另一种方式时也会有合并排序的过程                    |
| 对于本节点上的文件     | 仍然是通过网络框架拖取数据                        | 不通过网络框架，对于在本节点上的map输出文件，采用本地读取的方式 |
| copy过来的数据存放位置 | 先放在内存，内存放不下时写到磁盘                  | 一种方式全部放在内存；另一种方式先放在内存                   |
#### shuffer后续优化方向

通过上面的介绍，我们了解到，Shuffle过程的主要存储介质是**磁盘**，尽量的减少IO是Shuffle的主要优
化方向。我们脑海中都有那个经典的存储金字塔体系，Shuffle过程为什么把结果都放在磁盘上，那是因
为现在内存再大也大不过磁盘，内存就那么大，还这么多张嘴吃，当然是分配给最需要的了。如果具有
“土豪”内存节点，减少Shuffle IO的最有效方式无疑是尽量把数据放在内存中。下面列举一些现在看可以
优化的方面，期待经过我们不断的努力，TDW计算引擎运行地更好。

**MapReduce Shuffle后续优化方向**

- 压缩：对数据进行压缩，减少写读数据量；
- 减少不必要的排序：并不是所有类型的Reduce需要的数据都是需要排序的，排序这个过程如
  果不需要最好还是不要的好；
- 内存化：Shuffle的数据不放在磁盘而是尽量放在内存中，除非逼不得已往磁盘上放；当然了如果
  有性能和内存相当的第三方存储系统，那放在第三方存储系统上也是很好的；这个是个大招；
- 网络框架：netty的性能据说要占优了；
- 本节点上的数据不走网络框架：对于本节点上的Map输出，Reduce直接去读吧，不需要绕道网络
  框架。

**Spark Shuffle后续优化方向**

Spark作为MapReduce的进阶架构，对于Shuffle过程已经是优化了的，特别是对于那些具有争议的步
骤已经做了优化，但是Spark的Shuffle对于我们来说在一些方面还是需要优化的。

- 压缩：对数据进行压缩，减少写读数据量；
- 内存优化：Spark历史版本中是有这样设计的：Map写数据先把数据全部写到内存中，写完之后再把
  数据刷到磁盘上；考虑内存是紧缺资源，后来修改成把数据直接写到磁盘了；对于具有较大内存的
  集群来讲，还是尽量地往内存上写吧，内存放不下了再放磁盘。

### Spark的数据本地性有哪几种？

Spark中的数据本地性有三种：

1. PROCESS_LOCAL是指读取缓存在本地节点的数据
2. NODE_LOCAL是指读取本地节点硬盘数据
3. ANY是指读取非本地节点数据

通常读取数据PROCESS_LOCAL>NODE_LOCAL>ANY，尽量使数据以PROCESS_LOCAL或NODE_LOCAL方式读取。其中PROCESS_LOCAL还和cache有关，如果RDD经常用的话将该RDD cache到内存中，注意，由于cache是lazy的，所以必须通过一个action的触发，才能真正的将该RDD cache到内存中。

### RDD 持久化或者说是缓存原理？

spark 非常重要的一个功能特性就是可以将 RDD 持久化在内存中。

- 如果一个RDD需要重复使用，那么需要从头再次执行来获取数据

- RDD对象可以重用，但是数据不可以重用

- RDD通过Cache或者Persist方法讲前面计算的结果缓存，把数据以缓存在JVM的堆内存中

- 但是并不是这两方法被调用时立即缓存，而是触发后面的action算子时，该RDD将会被缓存在计算节点的内存中，供后面重用

- cache操作会增加血缘关系，不改变原来的血缘关系

持久化的作用 & 注意点：

1.  数据能够重复使用
2. cache默认持久化的操作，只能将数据保存到内存中
3.  如果想要保存磁盘文件，需要更改存储级别 StorageLevel.DISK_ONLY
4. 持久化操作必须在行动算子执行时完成
5.  RDD对象持久化不一定是为了重用数据，有些数据重要的场合也可以使用

调用 cache()和 persist()方法即可。cache()和 persist()的区别在于，cache()是 persist()的一种简化方式，cache()的底层就是调用 persist()的无参版本

persist(MEMORY_ONLY)，将数据持久化到内存中。

如果需要从内存中清除缓存，可以使用 unpersist()方法。RDD 持久化是可以手动选择不同的策略的。在调用 persist()时传入对应的 StorageLevel 即可。

### Spark为什么要持久化，一般什么场景下要进行persist操作？

**为什么要进行持久化？**

spark所有复杂一点的算法都会有persist身影，spark默认数据放在内存，spark很多内容都是放在内存
的，非常适合高速迭代，1000个步骤只有第一个输入数据，中间不产生临时数据，但分布式系统风险很
高，所以容易出错，就要容错，rdd出错或者分片可以根据血统算出来，如果没有对父rdd进行persist
或者cache的化，就需要重头做。

以下场景会使用persist

1. 某个步骤计算非常耗时，需要进行persist持久化
2. 计算链条非常长，重新恢复要算很多步骤，很好使，persist
3. checkpoint所在的rdd要持久化persist。checkpoint前，要持久化，写个rdd.cache或者rdd.persist，将结果保存起来，再写checkpoint操作，这样执行起来会非常快，不需要重新计算rdd链条了，checkpoint之前一定会进行persist
4. shuffle之后要persist，shuffle要进性网络传输，风险很大，数据丢失重来，恢复代价很大
5. shuffle之前进行persist，框架默认将数据持久化到磁盘，这个是框架自动做的。

### Checkpoint 检查点机制？

- 通过将RDD中间结果写入到磁盘

- 由于血缘依赖过长会造成容错成本过高，这样就不如在中间阶段左检查点容错

- 对RDD进行检查点操作并不会马上被执行，必须和执行Action操作才能触发


应用场景：当 spark 应用程序特别复杂，从初始的 RDD 开始到最后整个应用程序完成有很多的步骤，而且整个应用运行时间特别长，这种情况下就比较适合使用 checkpoint 功能。

原因：对于特别复杂的 Spark 应用，会出现某个反复使用的 RDD，即使之前持久化过但由于节点的故障导致数据丢失了，因为持久化的内存默认是存储在内存当中的，没有容错机制，所以需要重新计算一次数据。

Checkpoint 首先会调用 SparkContext 的 setCheckPointDIR()方法，设置一个容错的文件系统的目录，比如说 HDFS；然后对 RDD 调用 checkpoint()方法。之后在 RDD 所处的 job 运行结束之后，会启动一个单独的 job，来将checkpoint 过的 RDD 数据写入之前设置的文件系统，进行高可用、容错的类持久化操作。

检查点机制是我们在 spark streaming 中用来保障容错性的主要机制，它可以使 spark streaming 阶段性的把应用数据存储到诸如 HDFS 等可靠存储系统中，以供恢复时使用。具体来说基于以下两个目的服务：

1. 控制发生失败时需要重算的状态数。Spark streaming 可以通过转化图的谱系图来重算状态，检查点机制则可以控制需要在转化图中回溯多远。
2. 提供驱动器程序容错。如果流计算应用中的驱动器程序崩溃了，你可以重启驱动器程序并让驱动器程序从检查点恢复，这样 spark streaming 就可以读取之前运行的程序处理数据的进度，并从那里继续。

### Checkpoint 和持久化机制的区别？

最主要的区别在于**持久化只是将数据保存在 BlockManager** 中，但是 RDD 的lineage(血缘关系，依赖关系)是不变的。但是 checkpoint 执行完之后，rdd 已经没有之前所谓的依赖 rdd 了，而只有一个强行为其设置的 checkpointRDD，checkpoint 之后 rdd 的 lineage 就改变了。

持久化的数据丢失的可能性更大，因为节点的故障会导致磁盘、内存的数据丢失。但是 checkpoint 的数据通常是保存在高可用的文件系统中，比如 HDFS 中，所以数据丢失可能性比较低。

联系：两者都是做RDD持久化的

- 区别：
  - cache：
    - Cache缓存只是将数据保存起来，数据通常存储在磁盘、内存等地方，可靠性差；
    - 不会截断血缘关系，使用计算过程中的数据缓存
  - checkpoint：
  - ck的数据通常存储在HDFS等高可用、容错性好的文件系统；
  - 截断血缘关系，在checkpoint之前必须没有任何任务提交才会生效，ck过程会额外提交一次任务
  - 建议对ck的RDD使用Cache缓存，联合使用效果更好

### Spark 主备切换机制原理知道吗？

Master 实际上可以配置两个，Spark 原生的 standalone 模式是支持 Master主备切换的。当 Active Master 节点挂掉以后，我们可以将 Standby Master 切换为 Active Master。

Spark Master 主备切换可以基于两种机制，一种是基于文件系统的，一种是基于 ZooKeeper 的。

基于文件系统的主备切换机制，需要在 Active Master 挂掉之后手动切换到Standby Master 上；而基于 Zookeeper 的主备切换机制，可以实现自动切换 Master。

### Spark Master 使用 Zookeeper 进行 HA ，有哪些源数据保存到Zookeeper 里面？

spark 通过这个参数 spark.deploy.zookeeper.dir 指定 master 元数据在zookeeper 中保存的位置，包括 Worker，Driver 和 Application 以及Executors。standby 节点要从 zk 中，获得元数据信息，恢复集群运行状态，才能对外继续提供服务，作业提交资源申请等，在恢复前是不能接受请求的。

注：Master 切换需要注意 2 点：

1. 在 Master 切换的过程中，所有的已经在运行的程序皆正常运行！ 因为 SparkApplication 在运行前就已经通过 Cluster Manager 获得了计算资源，所以在运行时 Job 本身的 调度和处理和 Master 是没有任何关系。
2. 在 Master 的切换过程中唯一的影响是不能提交新的 Job：一方面不能够提交新的应用程序给集群， 因为只有 Active Master 才能接受新的程序的提交请求；另外一方面，已经运行的程序中也不能够因 Action 操作触发新的 Job 的提交请求。

### 描述Yarn执行一个任务的过程？

![1635399115690](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/133157-562171.png)

1. 客户端client向ResouceManager提交Application，ResouceManager接受Application并根据集群
   资源状况选取一个node来启动Application的任务调度器driver（ApplicationMaster）
2. ResouceManager找到那个node，命令其该node上的nodeManager来启动一个新的 JVM进程运行
   程序的driver（ApplicationMaster）部分，driver（ApplicationMaster）启动时会首先向ResourceManager注册，说明由自己来负责当前程序的运行
3. driver（ApplicationMaster）开始下载相关jar包等各种资源，基于下载的jar等信息决定向
   ResourceManager申请具体的资源内容
4. ResouceManager接受到driver（ApplicationMaster）提出的申请后，会最大化的满足资源分配请
   求，并发送资源的元数据信息给driver（ApplicationMaster）
5. driver（ApplicationMaster）收到发过来的资源元数据信息后会根据元数据信息发指令给具体机器
   上的NodeManager，让其启动具体的container
6. NodeManager收到driver发来的指令，启动container，container启动后必须向
   driver（ApplicationMaster）注册
7. driver（ApplicationMaster）收到container的注册，开始进行任务的调度和计算，直到任务完成。

> 如果ResourceManager第一次没有能够满足driver（ApplicationMaster）的资源请求 ，后续发
> 现有空闲的资源，会主动向driver（ApplicationMaster）发送可用资源的元数据信息以提供更多的资源用于当前程序的运行。

### Spark on Yarn 模式有哪些优点？

1. 与其他计算框架共享集群资源（Spark框架与MapReduce框架同时运行，如果不用Yarn进行资源分
   配，MapReduce分到的内存资源会很少，效率低下）；资源按需分配，进而提高集群资源利用等
2. 相较于Spark自带的Standalone模式，Yarn的资源分配更加细致
3. Application部署简化，例如Spark，Storm等多种框架的应用由客户端提交后，由Yarn负责资源的
   管理和调度，利用Container作为资源隔离的单位，以它为单位去使用内存,cpu等
4. Yarn通过队列的方式，管理同时运行在Yarn集群中的多个服务，可根据不同类型的应用程序负载情
   况，调整对应的资源使用量，实现资源弹性管理

### 谈谈你对container的理解？

1. Container作为资源分配和调度的基本单位，其中封装了的资源如内存，CPU，磁盘，网络带宽等。
   目前yarn仅仅封装内存和CPU
2. Container由ApplicationMaster向ResourceManager申请的，由ResouceManager中的资源调度器
   异步分配给ApplicationMaster
3. Container的运行是由ApplicationMaster向资源所在的NodeManager发起的，Container运行时需
   提供内部执行的任务命令。

### 介绍parition和block有什么关联关系？

1. hdfs中的block是分布式存储的最小单元，等分，可设置冗余，这样设计有一部分磁盘空间的浪费，
   但是整齐的block大小，便于快速找到、读取对应的内容
2. Spark中的partion是弹性分布式数据集RDD的最小单元，RDD是由分布在各个节点上的partion组成
   的。partion是指的spark在计算过程中，生成的数据在计算空间内最小单元，同一份数据（RDD）的
   partion大小不一，数量不定，是根据application里的算子和最初读入的数据分块数量决定
3. block位于存储空间、partion位于计算空间，block的大小是固定的、partion大小是不固定的，是
   从2个不同的角度去看数据。
4. partioion是一个逻辑上的概念，读取数据的时候，并不会真正的物理上对数据进行切分，而是逻辑上的切分，会将数据形成一个切分的规划文件，然后yarn根据合格切分的配置文件进行资源的申请。

### Spark应用程序的执行过程是什么？

1. 构建Spark Application的运行环境（启动SparkContext），SparkContext向资源管理器（可以是
   Standalone、Mesos或YARN）注册并申请运行Executor资源
2. 资源管理器分配Executor资源并启动StandaloneExecutorBackend，Executor运行情况将随着心跳
   发送到资源管理器上
3. SparkContext构建成DAG图，将DAG图分解成Stage，并把Taskset发送给Task Scheduler。Executor向SparkContext申请Task，Task Scheduler将Task发放给Executor运行同时SparkContext将应用程序代码发放给Executor
4. Task在Executor上运行，运行完毕释放所有资源。

> spark context是由driver负责创建的。

### 不需要排序的hash shuffle是否一定比需要排序的sort shuffle速度快？

不一定，当数据规模小，Hash shuffle快于Sorted Shuffle，数据规模大的时候；数据量大，sorted
Shuffle会比Hash shuffle快很多，因为数量大的有很多小文件，不均匀，甚至出现数据倾斜，消耗内存
大，1.x之前spark使用hash，适合处理中小规模，1.x之后，增加了Sorted shuffle，Spark更能胜任大
规模处理了。

### spark.storage.memoryFraction参数的含义,实际生产中如何调优？

用于设置RDD持久化数据在Executor内存中能占的比例，默认是0.6,默认Executor 60%的内存，可以用来保存持久化的RDD数据。根据你选择的不同的持久化策略，如果内存不够时，可能数据就不会持久化，或者数据会写入磁盘

如果持久化操作比较多，可以提高spark.storage.memoryFraction参数，使得更多的持久化数据保存在内存中，提高数据的读取性能，如果shuffle的操作比较多，有很多的数据读写操作到JVM中，那么应该调小一点，节约出更多的内存给JVM，避免过多的JVM gc发生。在web ui中观察如果发现gc时间很长，可以设置spark.storage.memoryFraction更小一点。

### 什么是RDD宽依赖和窄依赖？

RDD和它依赖的parent RDD(s)的关系有两种不同的类型，即窄依赖（narrow dependency）和宽依赖
（wide dependency）

1. 窄依赖指的是每一个parent RDD的Partition最多被子RDD的一个Partition使用，

![1635402199161](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/142320-641489.png)

2. 宽依赖指的是多个子RDD的Partition会依赖同一个parent RDD的Partition，简单来说就是广播。

![1635402242062](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/142403-782755.png)

具有宽依赖的 transformations 包括：sort，reduceByKey，groupByKey，join，和调用rePartition函数的任何操作。

### cache后面能不能接其他算子，它是不是action操作？

cache可以接其他算子，但是接了算子之后，起不到缓存应有的效果，因为会重新触发cache，
cache不是action操作

### Yarn中的container是由谁负责销毁的，在Hadoop Mapreduce中container可以复用么？

ApplicationMaster负责销毁，在Hadoop Mapreduce不可以复用，在spark on yarn程序container可
以复用。

### 提交任务时，如何指定Spark Application的运行模式？

- cluster模式：`./spark-submit --class xx.xx.xx --master yarn --deploy-mode cluster xx.jar`
- client模式：`./spark-submit --class xx.xx.xx --master yarn --deploy-mode client xx.jar`

### 不启动Spark集群Master和work服务，可不可以运行Spark程序？

可以，只要资源管理器第三方管理就可以，如由yarn管理，spark集群不启动也可以使用spark；spark
集群启动的是work和master，这个其实就是资源管理框架，yarn中的resourceManager相当于master，NodeManager相当于worker，做计算是Executor，和spark集群的work和manager可以没关系，归根接底还是JVM的运行，只要所在的JVM上安装了spark就可以。

### spark on yarn Cluster 模式下，ApplicationMaster和driver是在同一个进程么？

是，driver 位于ApplicationMaster进程中。该进程负责申请资源，还负责监控程序、资源的动态情
况。

如果是在client模式下，那么driver是运行在客户端进程中。

### Executor启动时，资源通过哪几个参数指定？

- num-executors：executor的数量
- executor-memory：每个executor使用的内存
- executor-cores：每个executor分配的CPU

### 为什么会产生yarn，解决了什么问题，有什么优势？

简单来说，yarn是为了针对MRV1的各种缺陷提出来的资源管理框架，简单来说就是一个资源管理和调度平台，可以想象为操作系统。

**Hadoop 和 MRv1 简单介绍**

Hadoop 集群可从单一节点（其中所有 Hadoop 实体都在同一个节点上运行）扩展到数千个节点（其中
的功能分散在各个节点之间，以增加并行处理活动）。图 1 演示了一个 Hadoop 集群的高级组件。

![1635403558931](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/144600-558211.png)

一个 Hadoop 集群可分解为两个抽象实体：MapReduce 引擎和分布式文件系统。MapReduce 引擎能
够在整个集群上执行 Map 和 Reduce 任务并报告结果，其中分布式文件系统提供了一种存储模式，可
跨节点复制数据以进行处理。Hadoop 分布式文件系统 (HDFS) 通过定义来支持大型文件（其中每个文
件通常为 64 MB 的倍数）。

当一个客户端向一个 Hadoop 集群发出一个请求时，此请求由 JobTracker 管理。JobTracker 与
NameNode 联合将工作分发到离它所处理的数据尽可能近的位置。NameNode 是文件系统的主系统，
提供元数据服务来执行数据分发和复制。JobTracker 将 Map 和 Reduce 任务安排到一个或多个
TaskTracker 上的可用插槽中。TaskTracker 与 DataNode（分布式文件系统）一起对来自 DataNode
的数据执行 Map 和 Reduce 任务。当 Map 和 Reduce 任务完成时，TaskTracker 会告知 JobTracker，
后者确定所有任务何时完成并最终告知客户作业已完成。

从上图中可以看到，MRv1 实现了一个相对简单的集群管理器来执行 MapReduce 处理。MRv1 提供了
一种分层的集群管理模式，其中大数据作业以单个 Map 和 Reduce 任务的形式渗入一个集群，并最后
聚合成作业来报告给用户。但这种简单性有一些隐秘，不过也不是很隐秘的问题。

**MRv1 的缺陷**
MapReduce 的第一个版本既有优点也有缺点。MRv1 是目前使用的标准的大数据处理系统。但是，这
种架构存在不足，主要表现在大型集群上。当集群包含的节点超过 4,000 个时（其中每个节点可能是多
核的），就会表现出一定的不可预测性。其中一个最大的问题是级联故障，由于要尝试复制数据和重载
活动的节点，所以一个故障会通过网络泛洪形式导致整个集群严重恶化。
但 MRv1 的最大问题是多租户。随着集群规模的增加，一种可取的方式是为这些集群采用各种不同的模
型。MRv1 的节点专用于 Hadoop，所以可以改变它们的用途以用于其他应用程序和工作负载。当大数
据和 Hadoop 成为云部署中一个更重要的使用模型时，这种能力也会增强，因为它允许在服务器上对
Hadoop 进行物理化，而无需虚拟化且不会增加管理、计算和输入/输出开销。
现在看看 YARN 的新架构，看看它如何支持 MRv2 和其他使用不同处理模型的应用程序。

**YARN (MRv2) 简介**
为了实现一个Hadoop集群的集群共享、可伸缩性和可靠性。设计人员采用一种分层的集群框架方法。
具体来讲，特定于MapReduce的功能已替换为一组新的守护程序，将框架向新的处理模型开放。
回想一下，由于限制了扩展以及网络开销所导致的某些故障模式，MRv1 JobTracker 和 TaskTracker 方
法曾是一个重要的缺陷。这些守护程序也是 MapReduce 处理模型所独有的。为了消除这一限制，
JobTracker 和 TaskTracker 已从 YARN 中删除，取而代之的是一组对应用程序不可知的新守护程序。

![1635403769163](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/144930-913918.png)

YARN 分层结构的本质是 ResourceManager。这个实体控制整个集群并管理应用程序向基础计算资源
的分配。ResourceManager 将各个资源部分（计算、内存、带宽等）精心安排给基础
NodeManager（YARN 的每节点代理）。ResourceManager 还与 ApplicationMaster 一起分配资源，
与 NodeManager 一起启动和监视它们的基础应用程序。在此上下文中，ApplicationMaster 承担了以
前的 TaskTracker 的一些角色，ResourceManager 承担了 JobTracker 的角色。
ApplicationMaster 管理一个在 YARN 内运行的应用程序的每个实例。ApplicationMaster 负责协调来
自 ResourceManager 的资源，并通过 NodeManager 监视容器的执行和资源使用（CPU、内存等的资
源分配）。请注意，尽管目前的资源更加传统（CPU 核心、内存），但未来会带来基于手头任务的新资
源类型（比如图形处理单元或专用处理设备）。从 YARN 角度讲，ApplicationMaster 是用户代码，因
此存在潜在的安全问题。YARN 假设 ApplicationMaster 存在错误或者甚至是恶意的，因此将它们当作
无特权的代码对待。

NodeManager 管理一个 YARN 集群中的每个节点。NodeManager 提供针对集群中每个节点的服务，
从监督对一个容器的终生管理到监视资源和跟踪节点健康。MRv1 通过插槽管理 Map 和 Reduce 任务
的执行，而 NodeManager 管理抽象容器，这些容器代表着可供一个特定应用程序使用的针对每个节点
的资源。YARN 继续使用 HDFS 层。它的主要 NameNode 用于元数据服务，而 DataNode 用于分散在
一个集群中的复制存储服务。
要使用一个 YARN 集群，首先需要来自包含一个应用程序的客户的请求。ResourceManager 协商一个
容器的必要资源，启动一个 ApplicationMaster 来表示已提交的应用程序。通过使用一个资源请求协
议，ApplicationMaster 协商每个节点上供应用程序使用的资源容器。执行应用程序时，
ApplicationMaster 监视容器直到完成。当应用程序完成时，ApplicationMaster 从 ResourceManager
注销其容器，执行周期就完成了。
通过这些讨论，应该明确的一点是，旧的 Hadoop 架构受到了 JobTracker 的高度约束，JobTracker 负
责整个集群的资源管理和作业调度。新的 YARN 架构打破了这种模型，允许一个新 ResourceManager
管理跨应用程序的资源使用，ApplicationMaster 负责管理作业的执行。这一更改消除了一处瓶颈，还
改善了将 Hadoop 集群扩展到比以前大得多的配置的能力。此外，不同于传统的 MapReduce，YARN
允许使用 Message Passing Interface 等标准通信模式，同时执行各种不同的编程模型，包括图形处
理、迭代式处理、机器学习和一般集群计算。

随着 YARN 的出现，开发者不再受到更简单的 MapReduce 开发模式约束，而是可以创建更复杂的分布
式应用程序。实际上，您可以将 MapReduce 模型视为 YARN 架构可运行的一些应用程序中的其中一
个，只是为自定义开发公开了基础框架的更多功能。这种能力非常强大，因为 YARN 的使用模型几乎没
有限制，不再需要与一个集群上可能存在的其他更复杂的分布式应用程序框架相隔离，就像 MRv1 一
样。甚至可以说，随着 YARN 变得更加健全，它有能力取代其他一些分布式处理框架，从而完全消除了
专用于其他框架的资源开销，同时还简化了整个系统。
归结而言，MRv1 框架下的问题仅是需要一个关联数组，而且这些问题有专门朝大数据操作方向演变的
倾向。但是，问题一定不会永远仅局限于此范式中，因为开发者现在可以更为简单地将它们抽象化，编
写自定义客户端、应用程序主程序，以及符合任何开发者想要的设计的应用程序。

### 一个task的map数量由谁来决定？

一般情况下，在输入源是文件的时候，一个task的map数量由splitSize来决定的，那么splitSize是由以
下几个来决定的

~~~ java
goalSize = totalSize / mapred.map.tasks
inSize = max {mapred.min.split.size, minSplitSize}
splitSize = max (minSize, min(goalSize, dfs.block.size))
~~~

一个task的reduce数量，由partition决定。

### 导致Executor产生FULL gc 的原因，可能导致什么问题？

可能导致Executor僵死问题，海量数据的shuffle和数据倾斜等都可能导致full gc。以shuffle为例，伴随
着大量的Shuffle写操作，JVM的新生代不断GC，Eden Space写满了就往Survivor Space写，同时超过
一定大小的数据会直接写到老生代，当新生代写满了之后，也会把老的数据搞到老生代，如果老生代空
间不足了，就触发FULL GC，还是空间不够，那就OOM错误了，此时线程被Blocked，导致整个
Executor处理数据的进程被卡住

### Spark累加器有哪些特点？

1. 累加器在全局唯一的，只增不减，记录全局集群的唯一状态
2. 在exe中修改它，在driver读取
3. executor级别共享的，广播变量是task级别的共享两个application不可以共享累加器，但是同一个
   app不同的job可以共享

### spark hashParitioner的弊端是什么？

HashPartitioner分区的原理很简单，对于给定的key，计算其hashCode，并除以分区的个数取余，如
果余数小于0，则用余数+分区的个数，最后返回的值就是这个key所属的分区ID；弊端是数据不均匀，
容易导致数据倾斜，极端情况下某几个分区会拥有rdd的所有数据。

### RangePartitioner分区的原理及特点？

**原理**
RangePartitioner分区则尽量保证每个分区中数据量的均匀，而且分区与分区之间是有序的，也就是说
一个分区中的元素肯定都是比另一个分区内的元素小或者大；但是分区内的元素是不能保证顺序的。简
单的说就是将一定范围内的数映射到某一个分区内。其原理是水塘抽样。

**特点**
RangePartioner尽量保证每个分区中数据量的均匀，而且分区与分区之间是有序的，一个分区中的元素
肯定都是比另一个分区内的元素小或者大；但是分区内的元素是不能保证顺序的。简单的说就是将一定
范围内的数映射到某一个分区内。RangePartitioner作用：将一定范围内的数映射到某一个分区内，在
实现中，分界的算法尤为重要。算法对应的函数是rangeBounds。

### 如何理解Standalone模式下，Spark资源分配是粗粒度的？

spark默认情况下资源分配是粗粒度的，也就是说程序在提交时就分配好资源，后面执行的时候使用分
配好的资源，除非资源出现了故障才会重新分配。比如Spark shell启动，已提交，一注册，哪怕没有任
务，worker都会分配资源给executor。

### 窄依赖父RDD的partition和子RDD的parition是不是都是一对一的关系？

不一定，除了一对一的窄依赖，还包含一对固定个数的窄依赖（就是对父RDD的依赖的Partition的数量
不会随着RDD数量规模的改变而改变），比如join操作的每个partiion仅仅和已知的partition进行join，
这个join操作是窄依赖，依赖固定数量的父rdd，因为是确定的partition关系。

### Hadoop中，Mapreduce操作的mapper和reducer阶段相当于spark中的哪几个算子？

相当于spark中的map算子和reduceByKey算子，当然还是有点区别的,MR会自动进行排序的，spark要
看你用的是什么partitioner。

### Spark中的HashShufle的有哪些不足？

1. shuffle产生海量的小文件在磁盘上，此时会产生大量耗时的、低效的IO操作
2. 容易导致内存不够用，由于内存需要保存海量的文件操作句柄和临时缓存信息，如果数据处理规模
   比较大的话，容易出现OOM
3. 容易出现数据倾斜，导致OOM。

### spark.default.parallelism这个参数有什么意义，实际生产中如何设置？

1. 参数用于设置每个stage的默认task数量。这个参数极为重要，如果不设置可能会直接影响你的
   Spark作业性能
2. 很多人都不会设置这个参数，会使得集群非常低效，你的cpu，内存再多，如果task始终为1，那也
   是浪费，spark官网建议task个数为CPU的核数*executor的个数的2~3倍。

### spark.shuffle.memoryFraction参数的含义，以及优化经验？

1. spark.shuffle.memoryFraction是shuffle调优中重要参数，shuffle从上一个task拉去数据过来，要
   在Executor进行聚合操作，聚合操作时使用Executor内存的比例由该参数决定，默认是20%如果聚合时数据超过了该大小，那么就会spill到磁盘，极大降低性能
2. 如果Spark作业中的RDD持久化操作较少，shuffle操作较多时，建议降低持久化操作的内存占比，
   提高shuffle操作的内存占比比例，避免shuffle过程中数据过多时内存不够用，必须溢写到磁盘上，降
   低了性能。此外，如果发现作业由于频繁的gc导致运行缓慢，意味着task执行用户代码的内存不够用，那么同样建议调低这个参数的值

### Spark中standalone模式特点，有哪些优点和缺点？

**特点**

1. standalone是master/slave架构，集群由Master与Worker节点组成，程序通过与Master节点交互
   申请资源，Worker节点启动Executor运行
2. standalone调度模式使用FIFO调度方式
3. 无依赖任何其他资源管理系统，Master负责管理集群资源

**优点**

1. 部署简单
2. 不依赖其他资源管理系统

**缺点**

1. 默认每个应用程序会独占所有可用节点的资源，当然可以通过spark.cores.max来决定一个应用可以
   申请的CPU cores个数
2. 可能有单点故障，需要自己配置master HA

### FIFO调度模式的基本原理、优点和缺点？

**基本原理**

按照先后顺序决定资源的使用，资源优先满足最先来的job。第一个job优先获取所有可用的资源，接下
来第二个job再获取剩余资源。以此类推，如果第一个job没有占用所有的资源，那么第二个job还可以
继续获取剩余资源，这样多个job可以并行运行，如果第一个job很大，占用所有资源，则第二job就需
要等待，等到第一个job释放所有资源。

**优点和缺点**

1. 适合长作业，不适合短作业
2. 适合CPU繁忙型作业（计算时间长，相当于长作业），不利于IO繁忙型作业（计算时间短，相当于
   短作业）

### FAIR调度模式的优点和缺点？

所有的任务拥有大致相当的优先级来共享集群资源，spark多以轮训的方式为任务分配资源，不管长任
务还是端任务都可以获得资源，并且获得不错的响应时间，对于短任务，不会像FIFO那样等待较长时间
了，通过参数spark.scheduler.mode 为FAIR指定。

### RDD的数据结构是怎么样的？ 一个RDD对象，包含如下5个核心属性。

1. 一个分区列表，每个分区里是RDD的部分数据（或称数据块）
2. 一个依赖列表，存储依赖的其他RDD
3. 一个名为compute的计算函数，用于计算RDD各分区的值
4. 分区器（可选），用于键/值类型的RDD，比如某个RDD是按散列来分区
5. 计算各分区时优先的位置列表（可选），比如从HDFS上的文件生成RDD时，RDD分区的位置优先选择数据所在的节点，这样可以避免数据移动带来的开销

### 说说你对Hadoop生态的认识。

hadoop生态主要分为三大类型

1. 分布式系统：HDFS，hbase
2. 分布式计算引擎：Spark，MapReduce
3. 周边工具：如zookeeper，pig，hive，oozie，sqoop，ranger，kafka等

### Spark streaming以及基本工作原理？

Spark streaming是spark core API的一种扩展，可以用于进行大规模、高吞吐量、容错的实时数据流的处理。

它支持从多种数据源读取数据，比如Kafka、Flume、Twitter和TCP Socket，并且能够使用算子比如map、reduce、join和window等来处理数据，处理后的数据可以保存到文件系统、数据库等存储中。

Spark streaming内部的基本工作原理是：接受实时输入数据流，然后将数据拆分成batch，比如每收集一秒的数据封装成一个batch，然后将每个batch交给spark的计算引擎进行处理，最后会生产处一个结果数据流，其中的数据也是一个一个的batch组成的。

###  DStream以及基本工作原理？

- DStream是spark streaming提供的一种高级抽象，代表了一个持续不断的数据流。
- DStream可以通过输入数据源来创建，比如Kafka、flume等，也可以通过其他DStream的高阶函数来创建，比如map、reduce、join和window等。
- DStream内部其实不断产生RDD，每个RDD包含了一个时间段的数据。
- Spark streaming一定是有一个输入的DStream接收数据，按照时间划分成一个一个的batch，并转化为一个RDD，RDD的数据是分散在各个子节点的partition中。

### Spark SQL 执行的流程？

这个问题如果深挖还挺复杂的，这里简单介绍下总体流程：

1. parser：基于 antlr 框架对 sql 解析，生成**抽象语法树**。
2. 变量替换：通过正则表达式找出符合规则的字符串，替换成系统缓存环境的变量，SQLConf 中的 spark.sql.variable.substitute ，默认是可用的；参考SparkSqlParser
3. parser：将 antlr 的 tree 转成 spark catalyst 的 LogicPlan，也就是 未解析的逻辑计划；详细参考 AstBuild , ParseDriver
4. analyzer：通过分析器，结合 catalog，把 logical plan 和实际的数据绑定起来，将 未解析的逻辑计划 生成 逻辑计划；详细参考QureyExecution
5. 缓存替换：通过 CacheManager，替换有相同结果的 logical plan（逻辑计划）
6. logical plan 优化，基于规则的优化；优化规则参考 Optimizer，优化执行器 RuleExecutor
7. 生成 spark plan，也就是物理计划；参考 QueryPlanner 和 SparkStrategies
8. spark plan 准备阶段
9. 构造 RDD 执行，涉及 spark 的 wholeStageCodegenExec 机制，基于janino 框架生成 java 代码并编译

### Spark SQL 到是如何将数据写到Hive 表的？

方式一：是利用 Spark RDD 的 API 将数据写入 hdfs 形成 hdfs 文件，之后再将 hdfs 文件和 hive 表做加载映射。

方式二：利用 Spark SQL 将获取的数据 RDD 转换成 DataFrame，再将DataFrame 写成缓存表，最后利用 Spark SQL 直接插入 hive 表中。而对于利用 Spark SQL 写 hive 表官方有两种常见的 API，**第一种是利用**
**JavaBean 做映射，第二种是利用 StructType 创建 Schema 做映射。**