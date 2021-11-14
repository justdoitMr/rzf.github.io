## Flink专题
<!-- TOC -->

- [Flink专题](#flink专题)
    - [41、要实现Exactly-Once,需具备什么条件？](#41要实现exactly-once需具备什么条件)
    - [42、什么是两阶段提交协议？](#42什么是两阶段提交协议)
    - [43、Flink 如何保证 Exactly-Once 语义？](#43flink-如何保证-exactly-once-语义)
    - [44、对Flink 端到端 严格一次Exactly-Once 语义做个总结](#44对flink-端到端-严格一次exactly-once-语义做个总结)
    - [45、Flink广播机制了解吗？](#45flink广播机制了解吗)
    - [46、Flink反压了解吗？](#46flink反压了解吗)
    - [47、Flink反压的影响有哪些？](#47flink反压的影响有哪些)
    - [48、Flink反压如何解决？](#48flink反压如何解决)
    - [49、Flink支持的数据类型有哪些？](#49flink支持的数据类型有哪些)
    - [50、Flink如何进行序列和反序列化的？](#50flink如何进行序列和反序列化的)
    - [51、为什么Flink使用自主内存而不用JVM内存管理？](#51为什么flink使用自主内存而不用jvm内存管理)
    - [52、那Flink自主内存是如何管理对象的？](#52那flink自主内存是如何管理对象的)
    - [53、Flink内存模型介绍一下？](#53flink内存模型介绍一下)
    - [54、Flink如何进行资源管理的？](#54flink如何进行资源管理的)
  - [1、集群架构剖析](#1集群架构剖析)
  - [2、Slot与资源管理](#2slot与资源管理)
  - [3、应用执行](#3应用执行)
  - [03、Flink 源码篇](#03flink-源码篇)
    - [55、FLink作业提交流程应该了解吧？](#55flink作业提交流程应该了解吧)
    - [56、FLink作业提交分为几种方式？](#56flink作业提交分为几种方式)
    - [57、FLink JobGraph是在什么时候生成的？](#57flink-jobgraph是在什么时候生成的)
    - [58、那在jobGraph提交集群之前都经历哪些过程？](#58那在jobgraph提交集群之前都经历哪些过程)
    - [59、看你提到PipeExecutor，它有哪些实现类？](#59看你提到pipeexecutor它有哪些实现类)
    - [60、Local提交模式有啥特点，怎么实现的？](#60local提交模式有啥特点怎么实现的)
    - [61、远程提交模式都有哪些？](#61远程提交模式都有哪些)
    - [62、Standalone模式简单介绍一下？](#62standalone模式简单介绍一下)
    - [63、yarn集群提交方式介绍一下？](#63yarn集群提交方式介绍一下)
    - [64、yarn - session模式特点？](#64yarn---session模式特点)
    - [65、yarn - perJob模式特点？](#65yarn---perjob模式特点)
    - [66、yarn - application模式特点？](#66yarn---application模式特点)
    - [67、yarn - session 提交流程详细介绍一下？](#67yarn---session-提交流程详细介绍一下)
    - [68、yarn - perjob 提交流程详细介绍一下？](#68yarn---perjob-提交流程详细介绍一下)
    - [69、流图、作业图、执行图三者区别？](#69流图作业图执行图三者区别)
    - [70、流图介绍一下？](#70流图介绍一下)
    - [71、作业图介绍一下？](#71作业图介绍一下)
    - [72、执行图介绍一下？](#72执行图介绍一下)
    - [73、Flink调度器的概念介绍一下？](#73flink调度器的概念介绍一下)
    - [74、Flink调度行为包含几种？](#74flink调度行为包含几种)
    - [75、Flink调度模式包含几种？](#75flink调度模式包含几种)
    - [76、Flink调度策略包含几种？](#76flink调度策略包含几种)
    - [80、Flink的任务槽是什么意思？](#80flink的任务槽是什么意思)
    - [81、Flink 槽共享又是什么意思？](#81flink-槽共享又是什么意思)
  - [04、Flink SQL篇](#04flink-sql篇)
    - [82、Flink SQL有没有使用过？](#82flink-sql有没有使用过)
    - [83、Flink被称作流批一体，那从哪个版本开始，真正实现流批一体的？](#83flink被称作流批一体那从哪个版本开始真正实现流批一体的)
    - [84、Flink SQL 使用哪种解析器？](#84flink-sql-使用哪种解析器)
    - [85、Calcite主要功能包含哪些？](#85calcite主要功能包含哪些)
    - [86、Flink SQL 处理流程说一下？](#86flink-sql-处理流程说一下)
    - [87、Flink SQL包含哪些优化规则？](#87flink-sql包含哪些优化规则)
    - [88、Flink SQL中涉及到哪些operation？](#88flink-sql中涉及到哪些operation)
    - [89、Flink Hive有没有使用过？](#89flink-hive有没有使用过)
    - [90、Flink与Hive集成时都做了哪些操作？](#90flink与hive集成时都做了哪些操作)
    - [91、HiveCatalog类包含哪些方法？](#91hivecatalog类包含哪些方法)
    - [92、Flink SQL1.11新增了实时数仓功能，介绍一下？](#92flink-sql111新增了实时数仓功能介绍一下)
    - [93、Flink -Hive实时写数据介绍下？](#93flink--hive实时写数据介绍下)
    - [94、Flink -Hive实时读数据介绍下？](#94flink--hive实时读数据介绍下)
    - [95、Flink -Hive实时写数据时，如何保证已经写入分区的数据何时才能对下游可见呢？](#95flink--hive实时写数据时如何保证已经写入分区的数据何时才能对下游可见呢)
    - [96、源码中分区提交的PartitionCommitTrigger介绍一下？](#96源码中分区提交的partitioncommittrigger介绍一下)
    - [97、PartitionTimeCommitTigger 是如何知道该提交哪些分区的呢？（源码分析）](#97partitiontimecommittigger-是如何知道该提交哪些分区的呢源码分析)
    - [98、如何保证已经写入分区的数据对下游可见的标志问题（源码分析）](#98如何保证已经写入分区的数据对下游可见的标志问题源码分析)
    - [99、Flink SQL CEP有没有接触过？](#99flink-sql-cep有没有接触过)
    - [100、Flink SQL CEP了解的参数介绍一下？](#100flink-sql-cep了解的参数介绍一下)
    - [101、编写一个CEP SQL案例，如银行卡盗刷](#101编写一个cep-sql案例如银行卡盗刷)
    - [102、Flink CDC了解吗？什么是 Flink SQL CDC Connectors？](#102flink-cdc了解吗什么是-flink-sql-cdc-connectors)
    - [103、Flink CDC原理介绍一下](#103flink-cdc原理介绍一下)
    - [104、通过CDC设计一种Flink SQL 采集+计算+传输(ETL)一体化的实时数仓](#104通过cdc设计一种flink-sql-采集计算传输etl一体化的实时数仓)
    - [105、Flink SQL CDC如何实现一致性保障（源码分析）](#105flink-sql-cdc如何实现一致性保障源码分析)
    - [106、Flink SQL GateWay了解吗？](#106flink-sql-gateway了解吗)
    - [107、Flink SQL GateWay创建会话讲解一下？](#107flink-sql-gateway创建会话讲解一下)
    - [108、Flink SQL GateWay如何处理并发请求？多个提交怎么处理？](#108flink-sql-gateway如何处理并发请求多个提交怎么处理)
    - [109、如何维护多个SQL之间的关联性？](#109如何维护多个sql之间的关联性)
    - [110、sql字符串如何提交到集群成为代码？](#110sql字符串如何提交到集群成为代码)

<!-- /TOC -->



#### 41、要实现Exactly-Once,需具备什么条件？

流系统要实现Exactly-Once，需要保证上游 Source 层、中间计算层和下游 Sink 层三部分同时满足**端到端严格一次处理，如下图：**

![1636786126215](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/144847-531017.png)

**Source端：**数据从上游进入Flink，必须保证消息严格一次消费。同时Source 端必须满足可重放（replay）。否则 Flink 计算层收到消息后未计算，却发生 failure 而重启，消息就会丢失。

**Flink计算层：**利用 Checkpoint 机制，把状态数据定期持久化存储下来，Flink程序一旦发生故障的时候，可以选择状态点恢复，避免数据的丢失、重复。

**Sink端**:Flink将处理完的数据发送到Sink端时，通过 **两阶段提交协议 ，**即 TwoPhaseCommitSinkFunction 函数。该 SinkFunction 提取并封装了两阶段提交协议中的公共逻辑，保证Flink 发送Sink端时实现严格一次处理语义。   **同时：**Sink端必须支持**事务机制**，能够进行数据**回滚**或者满足**幂等性。**

**回滚机制：**即当作业失败后，能够将部分写入的结果回滚到之前写入的状态。

**幂等性：**就是一个相同的操作，无论重复多少次，造成的结果和只操作一次相等。即当作业失败后，写入部分结果，但是当重新写入全部结果时，不会带来负面结果，重复写入不会带来错误结果。

#### 42、什么是两阶段提交协议？

**两阶段提交协议（Two -Phase Commit，2PC）是解决分布式事务问题最常用的方法，它可以保证在分布式事务中，**要么所有参与进程都提交事务，要么都取消**，即实现ACID中的 A（原子性）。**

两阶段提交协议中 有两个重要角色**，协调者（Coordinator）**和 **参与者（Participant）**,其中协调者只有一个，起到分布式事务的协调管理作用，参与者有多个。

两阶段提交阶段分为两个阶段：**投票阶段（Voting）**和 **提交阶段（Commit）。**

**投票阶段：**

（1）协调者向所有参与者**发送 prepare 请求**和事务内容，询问是否可以准备事务提交，等待参与者的相应。

（2）参与者执行事务中包含的操作，并记录 undo 日志（用于回滚）和 redo 日志（用于重放），但不真正提交。

（3）参与者向协调者返回事务操作的执行结果，执行成功返回yes，失败返回no。

**提交阶段：**

分为成功与失败两种情况。

若所有参与者都返回 yes，说明事务可以提交：

- 协调者向所有参与者发送 commit 请求。
- 参与者收到 commit 请求后，将事务真正地提交上去，并释放占用的事务资源，并向协调者返回 ack 。
- 协调者收到所有参与者的 ack 消息，事务成功完成，如下图：

![1636786355834](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/145238-801142.png)

若有参与者返回 no 或者超时未返回，说明事务中断，需要回滚：

- 协调者向所有参与者发送rollback请求。
- 参与者收到rollback请求后，根据undo日志回滚到事务执行前的状态，释放占用的事务资源，并向协调者返回ack。
- 协调者收到所有参与者的ack消息，事务回滚完成。

![1636786396061](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/145318-916970.png)

#### 43、Flink 如何保证 Exactly-Once 语义？

Flink通过两阶段提交协议来保证Exactly-Once语义。

**对于Source端：**Source端严格一次处理比较简单，因为数据要进入Flink 中，所以Flink 只需要保存消费数据的偏移量 （offset）即可。如果Source端为 kafka，Flink 将 Kafka Consumer 作为 Source，可以将偏移量保存下来，如果后续任务出现了故障，恢复的时候可以由连接器重置偏移量，重新消费数据，保证一致性。

**对于 Sink 端：**Sink 端是最复杂的，因为数据是落地到其他系统上的，数据一旦离开 Flink 之后，Flink 就监控不到这些数据了，所以严格一次处理语义必须也要应用于 Flink 写入数据的外部系统，**故这些外部系统必须提供一种手段允许提交或回滚这些写入操作**，同时还要保证与 Flink Checkpoint 能够协调使用（**Kafka 0.11 版本已经实现精确一次处理语义**）。

我们以 **Kafka - Flink -Kafka** 为例 说明如何保证Exactly-Once语义。

![1636786778053](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/145939-695951.png)

如上图所示：Flink作业包含以下算子。

（1）一个Source算子，从Kafka中读取数据（即KafkaConsumer）

（2）一个窗口算子，基于时间窗口化的聚合运算（即window+window函数）

（3）一个Sink算子，将结果写会到Kafka（即kafkaProducer）

Flink使用两阶段提交协议 **预提交（Pre-commit）**阶段和 **提交（Commit）阶段保证端到端严格一次。**

**（1）预提交阶段**

**1、当Checkpoint 启动时，进入预提交阶段**，JobManager 向Source Task 注入检查点分界线（CheckpointBarrier）,Source Task 将 CheckpointBarrier 插入数据流，向下游广播开启本次快照，如下图所示：

![1636786863780](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/150104-39956.png)

**2、Source 端：***Flink Data Source 负责保存 KafkaTopic 的 offset偏移量**，当 Checkpoint 成功时 Flink 负责提交这些写入，否则就终止取消掉它们，当 Checkpoint 完成位移保存，它会将 checkpoint barrier（检查点分界线） 传给下一个 Operator，然后每个算子会对当前的状态做个快照，保存到**状态后端（State Backend）。

**对于 Source 任务而言，就会把当前的 offset 作为状态保存起来。下次从 Checkpoint 恢复时，Source 任务可以重新提交偏移量，从上次保存的位置开始重新消费数据，如下图所示：**

![1636786926759](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/150220-795312.png)

预处理阶段：checkpoint barrier传递 及 offset 保存

**3、Slink 端：**从 Source 端开始，每个内部的 transformation 任务遇到 checkpoint barrier（检查点分界线）时，都会把状态存到 Checkpoint 里。数据处理完毕到 Sink 端时，Sink 任务首先把数据写入外部 Kafka，**这些数据都属于预提交的事务（还不能被消费）**，**此时的 Pre-commit 预提交阶段下Data Sink 在保存状态到状态后端的同时还必须预提交它的外部事务，**如下图所示：

![1636786988260](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/150310-459870.png)

预处理阶段：预提交到外部系统

**（2）提交阶段**

**4、当所有算子任务的快照完成**（所有创建的快照都被视为是 Checkpoint 的一部分），**也就是这次的 Checkpoint 完成时**，**JobManager 会向所有任务发通知，确认这次 Checkpoint 完成，此时 Pre-commit 预提交阶段才算完成**。才正式到两阶段提交协议的**第二个阶段：**commit 阶段。该阶段中 JobManager 会为应用中每个 Operator 发起 Checkpoint 已完成的回调逻辑。

本例中的 Data Source 和窗口操作无外部状态，因此在该阶段，这两个 Opeartor 无需执行任何逻辑，**但是 Data Sink 是有外部状态的，此时我们必须提交外部事务**，当 Sink 任务收到确认通知，就会正式提交之前的事务，Kafka 中未确认的数据就改为“已确认”，数据就真正可以被消费了，如下图所示：

![1636787117421](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/150519-76449.png)

提交阶段：数据精准被消费

> 注：Flink 由 JobManager 协调各个 TaskManager 进行 Checkpoint 存储，Checkpoint 保存在 StateBackend（状态后端） 中，默认 StateBackend 是内存级的，也可以改为文件级的进行持久化保存。

#### 44、对Flink 端到端 严格一次Exactly-Once 语义做个总结

![1636787214188](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/094713-401934.png)

#### 45、Flink广播机制了解吗？

如下图所示：

![1636787262956](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/150744-847457.png)

（1）从图中可以理解 **广播** 就是一个**公共的共享变量，**广播变量是发给**TaskManager的内存中，**所以广播变量不应该太大，将一个数据集广播后，不同的**Task**都可以在节点上获取到，每个节点只存一份。 如果不使用广播，每一个Task都会拷贝一份数据集，造成内存资源浪费  。

#### 46、Flink反压了解吗？

反压（backpressure）是实时计算应用开发中，特别是流式计算中，十分常见的问题。**反压**意味着数据管道中某个节点成为瓶颈，**下游处理速率** **跟不上** **上游发送数据的速率**，而需要对上游进行限速。由于实时计算应用通常使用消息队列来进行生产端和消费端的解耦，消费端数据源是 pull-based 的，所以反压通常是从某个节点传导至数据源并降低数据源（比如 Kafka consumer）的摄入速率。

简单来说就是**下游处理速率** **跟不上** **上游发送数据的速率**，下游来不及消费，导致队列被占满后，上游的生产会被阻塞，最终导致数据源的摄入被阻塞。

#### 47、Flink反压的影响有哪些？

反压会影响到两项指标: **checkpoint 时长**和 **state 大小**

（1）前者是因为 checkpoint barrier 是不会越过普通数据的，**数据处理被阻塞**也会**导致** checkpoint barrier 流经**整个数据管道的时长变长**，因而 checkpoint 总体时间（End to End Duration）变长。

（2）后者是因为为保证 EOS（Exactly-Once-Semantics，准确一次），对于有两个以上输入管道的 Operator，checkpoint barrier 需要对齐（Alignment），接受到较快的输入管道的 barrier 后，它后面数据会被缓存起来但不处理，直到较慢的输入管道的 barrier 也到达，这些被缓存的数据会被放到state 里面，导致 checkpoint 变大。

**这两个影响对于生产环境的作业来说是十分危险的**，因为 checkpoint 是保证数据一致性的关键，checkpoint 时间变长有可能导致 checkpoint 超时失败，而 state 大小同样可能拖慢 checkpoint 甚至导致 OOM （使用 Heap-based StateBackend）或者物理内存使用超出容器资源（使用 RocksDBStateBackend）的稳定性问题。

#### 48、Flink反压如何解决？

Flink社区提出了 FLIP-76: Unaligned Checkpoints[4] 来解耦反压和 checkpoint。

（1）定位反压节点

要解决反压首先要做的是定位到造成反压的节点，这主要有两种办法:

1. 通过 Flink Web UI 自带的反压监控面板；
2. 通过 Flink Task Metrics。

**（1）反压监控面板**

Flink Web UI 的反压监控提供了 SubTask 级别的反压监控，原理是通过周期性对 Task 线程的栈信息采样，得到线程被阻塞在请求 Buffer（意味着被下游队列阻塞）的频率来判断该节点是否处于反压状态。默认配置下，这个频率在 0.1 以下则为 OK，0.1 至 0.5 为 LOW，而超过 0.5 则为 HIGH。

![1636787557625](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/151239-298269.png)

**（2）Task Metrics**

Flink 提供的 Task Metrics 是更好的反压监控手段

如果一个 Subtask 的发送端 Buffer 占用率很高，则表明它被下游反压限速了；

如果一个 Subtask 的接受端 Buffer 占用很高，则表明它将反压传导至上游。

#### 49、Flink支持的数据类型有哪些？

Flink支持的数据类型如下图所示：

![图片](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/151312-999578.png)

从图中可以看到 Flink 类型可以分为基础类型（Basic）、数组（Arrays）、复合类型（Composite）、辅助类型（Auxiliary）、泛型和其它类型（Generic）。Flink 支持任意的 Java 或是 Scala 类型。 

#### 50、Flink如何进行序列和反序列化的？

所谓序列化和反序列化的含义：

**序列化：**就是将一个内存对象转换成二进制串，形成网络传输或者持久化的数据流。

**反序列化：**将二进制串转换为内存对。

**TypeInformation 是 Flink 类型系统的核心类** 

**在Flink中，当数据需要进行序列化时，会使用TypeInformation的**生成序列化器接口调用一个 createSerialize() 方法，创建出TypeSerializer，TypeSerializer提供了序列化和反序列化能力。如下图所示：Flink 的序列化过程:

![1636787880390](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/151803-512600.png)

**对于大多数数据类型** **Flink 可以自动生成对应的序列化器***，能非常高效地对数据集进行序列化和反序列化 ，如下图：

![1636787916777](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/151839-161760.png)

**比如，BasicTypeInfo、WritableTypeIno ，但针对 GenericTypeInfo 类型，Flink 会使用 Kyro 进行序列化和反序列化。其中，Tuple、Pojo 和 CaseClass 类型是复合类型，它们可能嵌套一个或者多个数据类型。在这种情况下，它们的序列化器同样是复合的。它们会将内嵌类型的序列化委托给对应类型的序列化器。**

**通过一个案例介绍Flink序列化和反序列化：**

![1636787973149](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/151935-647474.png)

**如上图所示，当创建一个Tuple 3 对象时，包含三个层面，一是 int 类型，一是 double 类型，还有一个是 Person。Person对象包含两个字段，一是 int 型的 ID，另一个是 String 类型的 name，**

**（1）在序列化操作时，会委托相应具体序列化的序列化器进行相应的序列化操作。从图中可以看到 Tuple 3 会把 int 类型通过 IntSerializer 进行序列化操作，此时 int 只需要占用四个字节。**

**（2）Person 类会被当成一个 Pojo 对象来进行处理，PojoSerializer 序列化器会把一些属性信息使用一个字节存储起来。同样，其字段则采取相对应的序列化器进行相应序列化，在序列化完的结果中，可以看到所有的数据都是由 MemorySegment 去支持。** 

**MemorySegment 具有什么作用呢？**

**MemorySegment 在 Flink 中会将对象序列化到预分配的内存块上，它代表 1 个固定长度的内存，默认大小为 32 kb。MemorySegment 代表 Flink 中的一个最小的内存分配单元，相当于是 Java 的一个 byte 数组。每条记录都会以序列化的形式存储在一个或多个 MemorySegment 中。**

#### 51、为什么Flink使用自主内存而不用JVM内存管理？

因为在内存中存储大量的数据 （包括缓存和高效处理）时，JVM会面临很多问题，包括如下：

JVM 内存管理的不足： 

**1）Java 对象存储密度低。**Java 的对象在内存中存储包含 3 个主要部分：对象头、实例 数据、对齐填充部分。例如，一个只包含 boolean 属性的对象占 16byte：对象头占 8byte， boolean 属性占 1byte，为了对齐达到 8 的倍数额外占 7byte。而实际上只需要一个 bit（1/8 字节）就够了。

**2）Full GC 会极大地影响性能。**尤其是为了处理更大数据而开了很大内存空间的 JVM 来说，GC 会达到秒级甚至分钟级。

**3）OOM 问题影响稳定性。**OutOfMemoryError 是分布式计算框架经常会遇到的问题， 当JVM中所有对象大小超过分配给JVM的内存大小时，就会发生OutOfMemoryError错误， 导致 JVM 崩溃，分布式框架的健壮性和性能都会受到影响。

**4）缓存未命中问题。**CPU 进行计算的时候，是从 CPU 缓存中获取数据。现代体系的 CPU 会有多级缓存，而加载的时候是以 Cache Line 为单位加载。如果能够将对象连续存储， 这样就会大大降低 Cache Miss。使得 CPU 集中处理业务，而不是空转。

#### 52、那Flink自主内存是如何管理对象的？

Flink 并不是将大量对象存在堆内存上，而是将对象都序列化到一个预分配的内存块上， 这个内存块叫做 **MemorySegment**，它代表了一段固定长度的内存（默认大小为 32KB），也 是 **Flink 中最小的内存分配单元**，并且提供了非常高效的读写方法，很多运算可以直接操作 二进制数据，不需要反序列化即可执行。每条记录都会以序列化的形式存储在一个或多个 MemorySegment 中。如果需要处理的数据多于可以保存在内存中的数据，Flink 的运算符会 将部分数据溢出到磁盘

#### 53、Flink内存模型介绍一下？

Flink总体内存类图如下：

![图片](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/152344-726643.png)

主要包含**JobManager内存模型**和 **TaskManager内存模型**

**JobManager内存模型**

![1636788273015](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/152434-190395.png)

在 1.10 中，Flink 统一了 TM 端的内存管理和配置，相应的在 1.11 中，Flink 进一步 对 JM 端的内存配置进行了修改，使它的选项和配置方式与 TM 端的配置方式保持一致。

![1636788359730](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/152602-251279.png)

**TaskManager内存模型**

**Flink 1.10** 对 TaskManager 的内存模型和 Flink 应用程序的配置选项进行了**重大更改**， 让用户能够更加严格地控制其内存开销。

![1636788416502](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/152657-230534.png)

![1636788448121](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/13/152729-222409.png)

**JVM Heap：JVM 堆上内存** 

**1、Framework Heap Memory：**Flink 框架本身使用的内存，即 TaskManager 本身所 占用的堆上内存，不计入 Slot 的资源中。 

配置参数：taskmanager.memory.framework.heap.size=128MB,默认 128MB

**2、Task Heap Memory：**Task 执行用户代码时所使用的堆上内存。

配置参数：taskmanager.memory.task.heap.size

**Off-Heap Mempry：JVM 堆外内存** 

**1、DirectMemory：JVM 直接内存** 

1）Framework Off-Heap Memory：Flink框架本身所使用的内存，即TaskManager 本身所占用的对外内存，不计入 Slot 资源。

配置参数：taskmanager.memory.framework.off-heap.size=128MB，默认 128MB 

2）Task Off-Heap Memory：Task 执行用户代码所使用的对外内存。

 配置参数：taskmanager.memory.task.off-heap.size=0,默认 0 

3）Network Memory：网络数据交换所使用的堆外内存大小，如网络数据交换 缓冲区 

**2、Managed Memory：Flink 管理的堆外内存**，

用于排序、哈希表、缓存中间结果及 RocksDB State Backend 的本地内存。

**JVM specific memory：JVM 本身使用的内存** 

**1、JVM metaspace：JVM 元空间** 

**2、JVM over-head 执行开销：**JVM 执行时自身所需要的内容，包括线程堆栈、IO、 编译缓存等所使用的内存。

配置参数：taskmanager.memory.jvm-overhead.min=192mb

taskmanager.memory.jvm-overhead.max=1gb 

taskmanager.memory.jvm-overhead.fraction=0.1 

**总体内存** 

**1、总进程内存：**Flink Java 应用程序（包括用户代码）和 JVM 运行整个进程所消 耗的总内存。

总进程内存 = Flink 使用内存 + JVM 元空间 + JVM 执行开销 

配置项：taskmanager.memory.process.size: 1728m

**2、Flink 总内存：**仅 Flink Java 应用程序消耗的内存，包括用户代码，但不包括 JVM 为其运行而分配的内存。

Flink 使用内存：框架堆内外 + task 堆内外 + network + manage

#### 54、Flink如何进行资源管理的？

Flink在资源管理上可以分为两层：**集群资源**和**自身资源**。集群资源支持主流的资源管理系统，如yarn、mesos、k8s等，也支持独立启动的standalone集群。自身资源涉及到每个子task的资源使用，由Flink自身维护。


### 1、集群架构剖析

Flink的运行主要由 客户端、一个JobManager（后文简称JM）和 一个以上的TaskManager（简称TM或Worker）组成。

![1636853083770](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/092444-130287.png)

**客户端**

客户端主要用于提交任务到集群，在Session或Per Job模式中，客户端程序还要负责**解析用户代码，生成JobGraph**；在Application模式中，直接提交用户jar和执行参数即可。客户端一般支持两种模式：detached模式，客户端提交后自动退出。attached模式，客户端提交后阻塞等待任务执行完毕再退出。 

**JobManager**

JM负责决定应用何时调度task，在task执行结束或失败时如何处理，协调检查点、故障恢复。该进程主要由下面几个部分组成：

1. **ResourceManager**，负责资源的申请和释放、管理slot（Flink集群中最细粒度的资源管理单元）。Flink实现了多种RM的实现方案以适配多种资源管理框架，如yarn、mesos、k8s或standalone。在standalone模式下，RM只能分配slot，而不能启动新的TM。注意：这里所说的RM跟Yarn的RM不是一个东西，这里的RM是JM中的一个独立的服务。

2. **Dispatcher**，提供Flink提交任务的rest接口，为每个提交的任务启动新的JobMaster，为所有的任务提供web ui，查询任务执行状态。

3. **JobMaster**，负责管理执行单个JobGraph，多个任务可以同时在一个集群中启动，每个都有自己的JobMaster。注意这里的JobMaster和JobManager的区别。

**TaskManager**

TM也叫做worker，用于执行数据流图中的任务，缓存并交换数据。集群至少有一个TM，TM中最小的资源管理单元是Slot，每个Slot可以执行一个Task，因此TM中slot的数量就代表同时可以执行任务的数量。

### 2、Slot与资源管理

每个TM是一个独立的JVM进程，内部基于独立的线程执行一个或多个任务。TM为了控制每个任务的执行资源，使用task slot来进行管理。每个task slot代表TM中的一部分固定的资源，比如一个TM有3个slot，每个slot将会得到TM的1/3内存资源。不同任务之间不会进行资源的抢占，注意GPU目前没有进行隔离，目前slot只能划分内存资源。

比如下面的数据流图，在扩展成并行流图后，同一的task可能分拆成多个任务并行在集群中执行。操作链可以把多个不同的任务进行合并，从而支持在一个线程中先后执行多个任务，无需频繁释放申请线程。同时操作链还可以统一缓存数据，增加数据处理吞吐量，降低处理延迟。

在Flink中，想要不同子任务合并需要满足几个条件：下游节点的入边是1（保证不存在数据的shuffle）；子任务的上下游不为空；连接策略总是ALWAYS；分区类型为ForwardPartitioner；并行度一致；当前Flink开启Chain特性。

 ![1636853401596](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/093004-621723.png)

在集群中的执行图可能如下：

![1636853467102](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/093107-955410.png)

Flink也支持slot的共享，即把不同任务根据任务的依赖关系分配到同一个Slot中。这样带来几个好处：方便统计当前任务所需的最大资源配置（某个子任务的最大并行度）；避免Slot的过多申请与释放，提升Slot的使用效率。

![1636853521080](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/093203-103631.png)

通过Slot共享，就有可能某个Slot中包含完整的任务执行链路。

### 3、应用执行

一个Flink应用就是用户编写的main函数，其中可能包含一个或多个Flink的任务。这些任务可以在本地执行，也可以在远程集群启动，集群既可以长期运行，也支持独立启动。下面是目前支持的任务提交方案：

**Session集群**

**生命周期**：集群事先创建并长期运行，客户端提交任务时与该集群连接。即使所有任务都执行完毕，集群仍会保持运行，除非手动停止。因此集群的生命周期与任务无关。

**资源隔离**：TM的slot由RM申请，当上面的任务执行完毕会自动进行释放。由于多个任务会共享相同的集群，因此任务间会存在竞争，比如网络带宽等。如果某个TM挂掉，上面的所有任务都会失败。

**其他方面**：拥有提前创建的集群，可以避免每次使用的时候过多考虑集群问题。比较适合那些执行时间很短，对启动时间有比较高的要求的场景，比如交互式查询分析。

**Per Job集群**

**生命周期**：为每个提交的任务单独创建一个集群，客户端在提交任务时，直接与ClusterManager沟通申请创建JM并在内部运行提交的任务。TM则根据任务运行需要的资源延迟申请。一旦任务执行完毕，集群将会被回收。

**资源隔离**：任务如果出现致命问题，仅会影响自己的任务。

**其他方面**：由于RM需要申请和等待资源，因此启动时间会稍长，适合单个比较大、长时间运行、需要保证长期的稳定性、不在乎启动时间的任务。

**Application集群**

**生命周期**：与Per Job类似，只是main()方法运行在集群中。任务的提交程序很简单，不需要启动或连接集群，而是直接把应用程序打包到资源管理系统中并启动对应的EntryPoint，在EntryPoint中调用用户程序的main()方法，解析生成JobGraph，然后启动运行。集群的生命周期与应用相同。

**资源隔离**：RM和Dispatcher是应用级别。 

### 03、Flink 源码篇

![1636853640470](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/093409-454346.png)

#### 55、FLink作业提交流程应该了解吧？

**Flink的提交流程：**

1. 在Flink Client中，通过反射启动jar中的main函数，生成Flink StreamGraph和JobGraph，将JobGraph提交给Flink集群

2. Flink集群收到JobGraph（JobManager收到）后，将JobGraph翻译成ExecutionGraph,然后开始调度，启动成功之后开始消费数据。

总结来说：Flink核心执行流程，对用户API的调用可以转为 StreamGraph -->JobGraph -- >  ExecutionGraph。

#### 56、FLink作业提交分为几种方式？

**Flink的作业提交分为两种方式**

1. **Local 方式：**即本地提交模式，直接在IDEA运行代码。

2. **远程提交方式：**分为Standalone方式、yarn方式、K8s方式

3. **Yarn 方式分为三种提交模式：**Yarn-perJob模式、Yarn-Sessionmo模式、Yarn-Application模式

#### 57、FLink JobGraph是在什么时候生成的？

StreamGraph、JobGraph全部是在Flink Client 客户端生成的，即提交集群之前生成，原理图如下：

![1636853897720](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/093820-148596.png)

#### 58、那在jobGraph提交集群之前都经历哪些过程？

（1）用户通过启动Flink集群，使用命令行提交作业，运行 flink run -c  WordCount  xxx.jar

（2）运行命令行后，会通过run脚本调用CliFrontend入口，CliFrontend会触发用户提交的jar文件中的main方法，然后交给PipelineExecuteor # execute方法，最终根据提交的模式选择触发一个具体的PipelineExecutor执行。

（3）根据具体的PipelineExecutor执行，将对用户的代码进行编译生成streamGraph，经过优化后生成jobgraph。

**具体流程图如下：**

![1636853995896](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/093957-465470.png)

#### 59、看你提到PipeExecutor，它有哪些实现类？

（1）PipeExecutor 在Flink中被叫做 流水线执行器，它是一个接口，是Flink Client生成JobGraph 之后，将作业提交给集群的重要环节，前面说过，作业提交到集群有好几种方式，最常用的是yarn方式，yarn方式包含3种提交模式，主要使用 session模式，perjob模式。Application模式 jaobGraph是在集群中生成。

所以PipeExecutor 的实现类如下图所示：（在代码中按CTRL+H就会出来）

![1636854036415](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/094038-100910.png)

除了上述框的两种模式外，在IDEA环境中运行Flink MiniCluster 进行调试时，使用LocalExecutor。

#### 60、Local提交模式有啥特点，怎么实现的？

（1）Local是在本地IDEA环境中运行的提交方式。不上集群。主要用于调试，原理图如下：

![1636854066161](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/094107-275316.png)

1. Flink程序由JobClient进行提交 

2. JobClient将作业提交给JobManager 

3. JobManager负责协调资源分配和作业执行。资源分配完成后，任务将提交给相应的TaskManager

4. TaskManager启动一个线程开始执行，TaskManager会向JobManager报告状态更改，如开始执 行，正在进行或者已完成。 

5. 作业执行完成后，结果将发送回客户端。

**源码分析：**通过Flink1.12.2源码进行分析的

**（1）创建获取对应的StreamExecutionEnvironment对象**：LocalStreamEnvironment

调用StreamExecutionEnvironment对象的execute方法

![1636854134587](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/094217-49070.png)

**（2）获取streamGraph**

![1636854227674](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/094348-259879.png)

**（3）执行具体的PipeLineExecutor - >得到localExecutorFactory**

![1636854251416](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/094412-997450.png)

**(4) 获取JobGraph**

根据localExecutorFactory的实现类LocalExecutor生成JobGraph

![1636854275135](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/094437-268640.png)

上面这部分全部是在Flink Client生成的，由于是使用Local模式提交。所有接下来将创建MiniCluster集群，由miniCluster.submitJob指定要提交的jobGraph

**（5）实例化MiniCluster集群**

![1636854301649](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/094502-314382.png)

**（6）返回JobClient 客户端**

在上面执行miniCluster.submitJob 将JobGraph提交到本地集群后，会返回一个JobClient客户端，该JobClient包含了应用的一些详细信息，包括JobID,应用的状态等等。最后返回到代码执行的上一层，对应类为StreamExecutionEnvironment。

![1636854329035](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/094530-365924.png)

**以上就是Local模式的源码执行过程。**

#### 61、远程提交模式都有哪些？

**远程提交方式：**分为Standalone方式、yarn方式、K8s方式

**Standalone：**包含session模式

**Yarn 方式分为三种提交模式：**Yarn-perJob模式、Yarn-Sessionmo模式、Yarn-Application模式。

**K8s方式：**包含 session模式

#### 62、Standalone模式简单介绍一下？

Standalone 模式为Flink集群的单机版提交方式，只使用一个节点进行提交，常用Session模式。

作业提交原理图如下：

![1636854591534](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/094953-851712.png)

提交命令如下：

```java
bin/flink run org.apache.flink.WordCount xxx.jar
```

1. client客户端提交任务给JobManager
2. JobManager负责申请任务运行所需要的资源并管理任务和资源，
3. JobManager分发任务给TaskManager执行
4. TaskManager定期向JobManager汇报状态

#### 63、yarn集群提交方式介绍一下？

通过yarn集群提交分为3种提交方式：分别为session模式、perjob模式、application模式

#### 64、yarn - session模式特点？

提交命令如下：

```java
./bin/flink run -t yarn-session \-Dyarn.application.id=application_XXXX_YY  xxx.jar
```

**Yarn-Session模式：**所有**作业共享集群资源**，隔离性差，JM负载瓶颈，main方法在客户端执行。适合执行时间短，频繁执行的短任务，集群中的所有作业 **只有一个JobManager**，另外，**Job被随机分配给TaskManager**

**特点：**

**Session-Cluster模式需要先启动集群**，**然后再提交作业**，接着会向yarn申请一块空间后，资源永远保持不变。如果资源满了，下一个作业就无法提交，只能等到 yarn中的其中一个作业执行完成后，释放了资源，下个作业才会正常提交。所有作业共享Dispatcher和ResourceManager；共享资源；适合规模小执行时间短的 作业。

**原理图如下：**

![1636854858621](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/095420-499419.png)

#### 65、yarn - perJob模式特点？

提交命令：

```java
./bin/flink run -t yarn-per-job --detached  xxx.jar
```

**Yarn-Per-Job模式：**每个作业单独启动集群，隔离性好，JM负载均衡，main方法在客户端执行。在per-job模式下，每个Job都有一个JobManager，每个TaskManager只有单个Job。

**特点：**

**一个任务会对应一个Job**，**每提交一个作业**会根据自身的情况，**都会单独向yarn申请资源**，直到作业执行完成，一个作业的失败与否并不会影响下一个作业的正常提交和运行。独享Dispatcher 和 ResourceManager，按需接受资源申请；**适合规模大长时间运行的作业。**

**原理图如下：**

![1636854928394](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/095531-690085.png)

#### 66、yarn - application模式特点？

**提交命令如下：**

```java
./bin/flink run-application -t yarn-application xxx.jar
```

**Yarn-Application模式：**每个作业单独启动集群，隔离性好，JM负载均衡，**main方法在JobManager上执行**。

**特点：**

在yarn-per-job 和 yarn-session模式下，客户端都需要执行以下三步，即： 

1、获取作业所需的依赖项； 

2、通过执行环境分析并取得逻辑计划，即StreamGraph→JobGraph；

3、将依赖项和JobGraph上传到集群中。 

![1636854998023](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/095639-719831.png)

只有在这些都完成之后，才会通过env.execute()方法 触发 Flink运行时真正地开始执行作业。**如果所有用户都在同一个客户端上提交作业**，**较大的依赖会消耗更多的带宽**，而较复杂的作业逻辑翻译成JobGraph也需要吃掉更多的CPU和内存，**客户端的资源反而会成为瓶颈**。

为了解决它，社区在传统部署模式的基础上实现了 Application模式。原本需要客户端做的三件事被转移到了JobManager里，也就是说main()方法在集群中执行(入口点位于 ApplicationClusterEntryPoint )，客 户端只需要负责发起部署请求了

**原理图如下：**

![1636855052223](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/095733-198922.png)

**综上所述，Flink社区比较推荐使用 yarn-perjob  或者 yarn-application模式进行提交应用。**

#### 67、yarn - session 提交流程详细介绍一下？

**提交流程图如下：**

![1636855114711](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/095835-995738.png)

**1、启动集群**

**（1）Flink Client向Yarn ResourceManager提交任务信息。**

​       1）Flink Client将应用配置（Flink-conf.yaml、logback.xml、log4j.properties）和相关文件（Flink Jar、配置类文件、用户Jar文件、JobGraph对象等）上传至分布式存储HDFS中。

​       2）Flink Client向Yarn ResourceManager提交任务信息

**（2）Yarn 启动 Flink集群，做2步操作：**

​        1) **通过Yarn Client 向Yarn  ResourceManager提交Flink创建集群的申请**，Yarn ResourceManager 分配Container 资源，并通知对应的NodeManager上启动一个ApplicationMaster(每提交一个flink job 就会启动一个applicationMaster)，ApplicationMaster会包含当前要启动的 JobManager和 Flink自己内部要使用的ResourceManager。

​        2）在JobManager 进程中运行**YarnSessionClusterEntryPoint** 作为集群启动的入口。初始化Dispatcher，Flink自己内部要使用的ResourceManager，启动相关RPC服务，等待Flink Client 通过Rest接口提交JobGraph。

**2、作业提交**

**（3）Flink Client 通过Rest 向Dispatcher 提交编译好的JobGraph。**Dispatcher 是 Rest 接口，不负责实际的调度、指定工作。

**（4）Dispatcher 收到 JobGraph 后，为作业创建一个JobMaster，将工作交给JobMaster，**JobMaster负责作业调度，管理作业和Task的生命周期**，构建ExecutionGraph（**JobGraph的并行化版本，调度层最核心的数据结构**）**

**以上两步执行完后，作业进入调度执行阶段。**

**3、作业调度执行**

**（5）JobMaster向ResourceManager申请资源，开始调度ExecutionGraph。**

（6）ResourceManager将资源请求加入等待队列，通过心跳向YarnResourceManager申请新的Container来启动TaskManager进程。

（7）YarnResourceManager启动，然后从HDFS加载Jar文件等所需相关资源，在容器中启动TaskManager，TaskManager启动TaskExecutor

（8）TaskManager启动后,向ResourceManager 注册，并把自己的Slot资源情况汇报给ResourceManager。

（9）ResourceManager从等待队列取出Slot请求，向TaskManager确认资源可用情况，并告知TaskManager将Slot分配给哪个JobMaster。

（10）TaskManager向JobMaster回复自己的一个Slot属于你这个任务，JobMaser会将Slot缓存到SlotPool。

（11）JobMaster调度Task到TaskMnager的Slot上执行。

#### 68、yarn - perjob 提交流程详细介绍一下？

**提交命令如下：**

```java
./bin/flink run -t yarn-per-job --detached  xxx.jar
```

**提交流程图如下所示：**

![1636855291878](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/100133-533544.png)

**1、启动集群**

**（1）Flink Client向Yarn ResourceManager提交任务信息。**

​       1）Flink Client将应用配置（Flink-conf.yaml、logback.xml、log4j.properties）和相关文件（Flink Jar、配置类文件、用户Jar文件、JobGraph对象等）上传至分布式存储HDFS中。

​       2）Flink Client向Yarn ResourceManager提交任务信息

**（2）Yarn 启动 Flink集群，做2步操作：**

​        1) **通过Yarn Client 向Yarn  ResourceManager提交Flink创建集群的申请**，Yarn ResourceManager 分配Container 资源，并通知对应的NodeManager上启动一个ApplicationMaster(每提交一个Flink job 就会启动一个ApplicationMaster)，ApplicationMaster会包含当前要启动的 JobManager和 Flink自己内部要使用的ResourceManager。

​        2）在JobManager 进程中运行**YarnJobClusterEntryPoint** 作为集群启动的入口。初始化Dispatcher，Flink自己内部要使用的ResourceManager，启动相关RPC服务，等待Flink Client 通过Rest接口提交JobGraph。

**2、作业提交**

**（3）ApplicationMaster启动Dispatcher，Dispatcher启动ResourceManager和JobMaster****(该步和Session不同，Jabmaster是由Dispatcher拉起，而不是Client传过来的)。**JobMaster负责作业调度，管理作业和Task的生命周期**，构建ExecutionGraph（**JobGraph的并行化版本，调度层最核心的数据结构**）**

**以上两步执行完后，作业进入调度执行阶段。**

**3、作业调度执行**

**（4）JobMaster向ResourceManager申请Slot资源，开始调度ExecutionGraph。**

（5）ResourceManager将资源请求加入等待队列，通过心跳向YarnResourceManager申请新的Container来启动TaskManager进程。

（6）YarnResourceManager启动，然后从HDFS加载Jar文件等所需相关资源，在容器中启动TaskManager。

（7）TaskManager在内部启动TaskExecutor。

（8）TaskManager启动后,向ResourceManager 注册，并把自己的Slot资源情况汇报给ResourceManager。

（9）ResourceManager从等待队列取出Slot请求，向TaskManager确认资源可用情况，并告知TaskManager将Slot分配给哪个JobMaster。

（10）TaskManager向JobMaster回复自己的一个Slot属于你这个任务，JobMaser会将Slot缓存到SlotPool。

（11）JobMaster调度Task到TaskMnager的Slot上执行。

#### 69、流图、作业图、执行图三者区别？

**Flink内部Graph总览图，由于现在Flink 实行流批一体代码，Batch API基本废弃，就不过多介绍** 

**在Flink DataStramAPI 中，Graph内部转换图如下：**

![1636855345801](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/100227-920844.png)

**以WordCount为例，流图、作业图、执行图、物理执行图之间的Task调度如下：**

![1636855378413](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/100259-597975.png)

对于Flink 流计算应用，运行用户代码时，首先调用DataStream API ，将用户代码转换为 **Transformation**，然后经过：**StreamGraph**->**JobGraph**->**ExecutionGraph** 3层转换（这些都是Flink内置的数据结构），最后经过Flink调度执行，在Flink 集群中启动计算任务，形成一个**物理执行图**。

#### 70、流图介绍一下？

**（1）流图  StreamGraph**

![1636855627796](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/100709-306479.png)

流图StreamGraph 核心对象包括两个：**StreamNode 点 和** **StreamEdge 边**

​    **1）StreamNode 点**

**StreamNode 点** ，从 Transformation转换而来，可以简单理解为 **StreamNode** 表示一个算子，存在实体和虚拟，可以有多个输入和输出，实体**StreamNode** 最终变成物理算子，虚拟的附着在**StreamEdge 边** 上。

​    **2）StreamEdge边**

**StreamEdge 是 StreamGraph 的边，**用来连接两个StreamNode 点，一个StreamEdge可以有多个出边、入边等信息。

#### 71、作业图介绍一下？

**（2）作业图  JobGraph**

JobGraph是由StreamGraph优化而来，是通过**OperationChain** 机制将算子合并起来，在执行时，调度在同一个Task线程上，避免数据的跨线程，跨网络传递。

![1636855713453](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/100834-850796.png)

作业图JobGraph 核心对象包括三个：

**JobVertex 点** **、** **JobEdge 边**、IntermediateDataSet 中间数据集

**1）JobVertex 点** 

经过算子融合优化后符合条件的多个StreamNode 可能会融合在一起生成一个 JobVertex，即一个JobVertex 包含一个或多个算子， **JobVertex 的输入是 JobEdge.** **输出是** **IntermediateDataSet** 

**2）**JobEdge 边

**Job**Edge表示 JobGraph 中的一 个数据流转通道， 其上游数据源是 **IntermediateDataSet** ，下游消费者是 **JobVertex** 。

**Job**Edge中的数据分发模式会直接影响执行时 Task 之间的数据连接关系是**点对点连接**还是**全连接**。

**3）**IntermediateDataSet 中间数据集

中间数据集 **IntermediateDataSet**  是一种逻辑结构.用来表示 **JobVertex** 的输出，即该 JobVertex 中包含的算子会产生的数据集。不同的执行模式下，其对应的结果分区类型不同，决 定了在执行时刻数据交换的模式。

#### 72、执行图介绍一下？

**（3）执行图  ExecutionGraph**

**ExecutionGraph**是调度Flink 作业执行的核心数据结构，包含了作业中所有并行执行的Task信息、Task之间的关联关系、数据流转关系。

StreamGraph 和JobGraph都在Flink Client生成，然后交给Flink集群。JobGraph到ExecutionGraph在JobMaster中 完成，转换过程中重要变化如下：

**1）加入了并行度的概念，成为真正可调度的图结构。**

**2）生成了6个核心对象。**

![1636855791179](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/100951-387153.png)

**执行图ExecutionGraph 核心对象包括6个：**

ExecutionJobVertex、ExecutionVertex、IntermediateResult、IntermediateResultPartition、ExecutionEdge、Execution。

**1）ExecutionJobVertex**

该对象和 JobGraph 中的 JobVertex 一 一对应。该对象还包含一组 ExecutionVertex， 数量 与该 JobVertex 中所包含的StreamNode 的并行度一致，假设 StreamNode 的并行度为5 ，那么**ExecutionJobVertex**中也会包含 5个**ExecutionVertex**。

**ExecutionJobVertex**用来将一个JobVertex 封装成 **ExecutionJobVertex**，并依次创建 ExecutionVertex、Execution、IntermediateResult 和  IntermediateResultPartition，用于丰富**ExecutionGraph。**

**2）ExecutionVertex**

**ExecutionJobVertex**会对作业进行并行化处理，构造可以并行执行的实例，每一个并行执行的实例就是 **ExecutionVertex**。

**3）IntermediateResult**

**IntermediateResult** 又叫作中间结果集，该对象是个逻辑概念 表示 **ExecutionJobVertex**输出，和 JobGrap 中的IntermediateDalaSet 一 一对应，同样 一个**ExecutionJobVertex**  可以有多个中间结果，取决于当前 JobVertex 有几个出边(JobEdge)。

**4）IntermediateResultPartition**

**IntermediateResultPartition** 又叫作中间结果分区。表示1个 **ExecutionVertex**输出结果，与 Execution Edge 相关联。

**5）ExecutionEdge**

表示**ExecutionVertex** 的输入，连按到上游产生的**IntermediateResultPartition** 。1个Execution对应唯一的1个**IntermediateResultPartition** 和1个**ExecutionVertex**。1个**ExecutionVertex** 可以有多个**ExecutionEdge。**

**6）**Execution

**ExecutionVertex** 相当于每个 Task 的模板，在真正执行的时候，会将**ExecutionVertex中的信息包装为1个**Execution，执行一个ExecutionVertex的一次尝试。

JobManager 和 TaskManager 之间关于Task 的部署和Task执行状态的更新都是通过ExecutionAttemptID来识别标识的。

**接下来问问作业调度的问题**

#### 73、Flink调度器的概念介绍一下？

**调度器**是Flink作业执行的核心组件，管理作业执行的所有相关过程，包括JobGraph到ExecutionGraph的转换、作业生命周期管理（作业的发布、取消、停止）、作业的Task生命周期管理（Task的发布、取消、停止）、资源申请与释放、作业和Task的Faillover等。

**（1）DefaultScheduler**

Flink 目前默认的调度器。是Flink新的调度设计，使用SchedulerStrategy来实现调度。

**（2）LegacySchedular**

过去的调度器，实现了原来的Execution调度逻辑。

#### 74、Flink调度行为包含几种？

**调度行为包含四种：**

SchedulerStrategy接口定义了调度行为，其中包含4种行为：

![1636855871200](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/101112-251859.png)

（1）startScheduling:调度入口，触发调度器的调度行为

（2）restartTasks:重启执行失败的Task,一般是Task执行异常导致的。

（3）onExecutionStateChange：当Execution状态发生改变时。

（4）onPartitionConsumable：当IntermediateResultPartition中的数据可以消费时。

#### 75、Flink调度模式包含几种？

**调度模式包含3种：Eager模式、分阶段模式（Lazy_From_Source）、分阶段Slot重用模式（Lazy_From_Sources_With_Batch_Slot_Request）。**

**1）Eager 调度**

　　**适用于流计算**。一次性申请需要的所有资源，如果资源不足，则作业启动失败。

**2）分阶段调度**

　　LAZY_FROM_SOURCES **适用于批处理**。从 SourceTask 开始分阶段调度，申请资源的时候，一次性申请本阶段所需要的所有资源。上游 Task 执行完毕后开始调度执行下游的 Task，

读取上游的数据，执行本阶段的计算任务，执行完毕之后，调度后一个阶段的 Task，依次进行调度，直到作业完成。

**3）分阶段 Slot 重用调度**

　　LAZY_FROM_SOURCES_WITH_BATCH_SLOT_REQUEST **适用于批处理**。与分阶段调度基本一样，区别在于该模式下使用批处理资源申请模式，可以在资源不足的情况下执行作

业，但是需要确保在本阶段的作业执行中没有 Shuffle 行为。

　　目前视线中的 Eager 模式和 LAZY_FROM_SOURCES 模式的资源申请逻辑一样，LAZY_FROM_SOURCES_WITH_BATCH_SLOT_REQUEST 是单独的资源申请逻辑。

#### 76、Flink调度策略包含几种？

**调度策略包含3种：**

![1636855933158](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/101214-519110.png)

调度策略全部实现于调度器SchedulingStrategy，有三种实现：

**1） EagerSchedulingStrategy：**适用于流计算，同时调度所有的 task

**2） LazyFromSourcesSchedulingStrategy：**适用于批处理，当输入数据准备好时（上游处理完）进行 vertices 调度。

**3） PipelinedRegionSchedulingStrategy：**以流水线的局部为粒度进行调度

　　PipelinedRegionSchedulingStrategy 是 1.11 加入的，从 1.12 开始，将以 pipelined region为单位进行调度。

pipelined region 是一组流水线连接的任务。这意味着，对于包含多个 region的流作业，在开始部署任务之前，它不再等待所有任务获取 slot。取而代之的是，一旦任何region 获得了足够的任务 slot 就可以部署它。对于批处理作业，将不会为任务分配 slot，也不会单独部署任务。取而代之的是，一旦某个 region 获得了足够的 slot，则该任务将与所有其他任务一起部署在同一区域中。

> 77、Flink作业生命周期包含哪些状态？

在Flink集群中，**JobMaster** 负责**作业**的生命周期管理，具体的管理行为在调度器和ExecutionGraph中实现。

作业的完整生命周期状态变换如下图所示：

![1636855959359](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/101241-880546.png)

（1）作业首先处于创建状态（created），然后切换到运行状态（running），并且在完成所有工作后，它将切换到完成状态（finished）。

（2）在失败的情况下，作业首先切换到失败状态（failing），取消所有正在运行任务。

如果所有节点都已达到最终状态，并且作业不可重新启动，则状态将转换为失败（failed）。（3）如果作业可以重新启动，那么它将进入重新启动状态（restarting）。一旦完成重新启动，它将变成创建状态（created）。

（4）在用户取消作业的情况下，将进入取消状态（cancelling），会取消所有当前正在运行的任务。一旦所有运行的任务已经达到最终状态，该作业将转换到已取消状态（canceled）。

**完成状态（finished）**，取消状态（canceled）**和**失败状态（failed）**表示一个全局的终结状态，并且触发清理工作，**而暂停状态（suspended）仅处于本地终止状态**。意味着作业的执行在相应的 JobManager 上终止，但集群的另一个 JobManager 可以从持久的HA存储中恢复这个作业并重新启动。因此，处于暂停状态的作业将不会被完全清理。

> 78、Task的作业生命周期包含哪些状态？

**TaskManager** 负责**Task** 的生命周期管理，并将状态的变化通知到JobMaster,在ExecutionGraph中跟踪Execution的状态变化，一个Execution对于一个Task。

**Task的生命周期如下：共8种状态。**

![1636855988815](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/101310-19479.png)

**在执行 ExecutionGraph 期间，每个并行任务经过多个阶段，从创建（created）到完成（finished）或失败（failed） ，下图说明了它们之间的状态和可能的转换。任务可以执行多次（例如故障恢复）。每个 Execution 跟踪一个 ExecutionVertex 的执行，每个 ExecutionVertex 都有一个当前 Execution（current execution）和一个前驱 Execution（prior execution）。**

> 79、Flink的任务调度流程讲解一下？

任务调度流程图如下：

![1636856013600](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/101334-396786.png)

1. 当Flink执行executor会自动根据程序代码生成**DAG数据流图** ，即 **Jobgraph；**

2. **ActorSystem**创建**Actor**将数据流图发送给JobManager中的Actor；

3. **JobManager**会不断接收**TaskManager**的心跳消息，从而可以获取到有效的TaskManager；

4. JobManager通过调度器在TaskManager中调度执行Task（在Flink中，最小的调度单元就是task，对应就是一个线程） ；

5. 在程序运行过程中，task与task之间是可以进行数据传输的 。

**• Job Client**

- 主要职责是提交任务, 提交后可以结束进程, 也可以等待结果返回 ；

- Job Client 不是 Flink 程序执行的内部部分，但它是任务执行的起点；

- Job Client 负责接受用户的程序代码，然后创建数据流，将数据流提交给 JobManager 以便进一步执行。执行完成后，Job Client 将结果返回给用户。

**• JobManager** 

- 主要职责是调度工作并协调任务做检查点；
- 集群中至少要有一个 master，master 负责调度 task，协调checkpoints 和 容错；
- 高可用设置的话可以有多个 master，但要保证一个是 leader, 其他是stand by；
- Job Manager 包含 **Actor System、Scheduler、CheckPoint**三个重要的组件 ；
- JobManager从客户端接收到任务以后, 首先生成优化过的执行计划, 再调度到TaskManager中执行。

**• TaskManager**

- 主要职责是从JobManager处接收任务, 并部署和启动任务, 接收上游的数 据并处理 
- Task Manager 是在 JVM 中的一个或多个线程中执行任务的工作节点。 
- TaskManager在创建之初就设置好了Slot, 每个Slot可以执行一个任务。

#### 80、Flink的任务槽是什么意思？

![1636856115902](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/101517-147923.png)

每个TaskManager是一个JVM的进程, 可以在不同的线程中执行一个或多个子任务。为了控制一个worker能接收多少个task。worker通过task slot来进行控制（一个worker 至少有一个task slot）。

**1、任务槽**

每个task slot表示TaskManager拥有资源的一个固定大小的子集。 

一般来说：我们分配槽的个数都是和CPU的核数相等，比如8核，那么就分配8个槽。

Flink将进程的内存划分到多个slot中。

图中有2个TaskManager，每个TaskManager有3个slot，每个slot占有1/3的内存。

内存被划分到不同的slot之后可以获得如下好处：

​    • **TaskManager**最多能同时并发执行的任务是可以控制的，那就是3个，因为不能超过slot的数量。 **任务槽的作用就是分离任务的托管内存，不会发生cpu隔离。**

​     • slot有独占的内存空间，这样在一个TaskManager中可以运行多个不同的作业，作业之间不受影响。

**总结：task slot的个数代表TaskManager可以并行执行的task数。**

#### 81、Flink 槽共享又是什么意思？

**2、槽共享**

**默认情况下，Flink允许子任务共享插槽**，即使它们是不同任务的子任务，只要它们来自同一个作业。结果是一个槽可以保存作业的整个管道。允许插槽共享有两个主要好处：

- 只需计算Job中最高并行度（parallelism）的task slot。只要这个满足，其他的job也都能满足。 
- 资源分配更加公平。如果有比较空闲的slot可以将更多的任务分配给它。图中若没有任务槽共享，负载不高的Source/Map等subtask将会占据许多资源，而负载较高的窗口subtask则会缺乏资源。 
- 有了任务槽共享，可以将基本并行度（base parallelism）从2提升到6。提高了分槽资源的利用率。同时它还可以保障TaskManager给subtask的分配的slot方案更加公平。

![1636856151070](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/101552-772955.png)


### 04、Flink SQL篇

Flink SQL也是面试的重点考察点，不仅需要你掌握扎实的SQL编程，同时还需要理解SQL提交的核心原理，以及Flink SQL中涉及的一些重点知识，例如CEP、CDC、SQL GateWay、SQL-Hive等，思维导图如下：

![1636856490919](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102131-910161.png)

#### 82、Flink SQL有没有使用过？

用过，在Flink中，一共有**四种**级别的**抽象**，而Flink SQL作为最上层，是Flink API的一等公民

![1636856523406](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102205-460742.png)

在标准SQL中，SQL语句包含四种类型

DML(Data Manipulation Language):数据操作语言，用来定义数据库记录（数据）。

DCL (Data Control Language):数据控制语言，用来定义访问权限和安全级别。

DQL (Data Query Language):数据查询语言，用来查询记录（数据）。

DDL(Data Definition Language):数据定义语言，用来定义数据库对象（库，表，列等）。

**Flink SQL包含 DML 数据操作语言、 DDL 数据语言， DQL 数据查询语言**，**不包含DCL语言。**

#### 83、Flink被称作流批一体，那从哪个版本开始，真正实现流批一体的？

从1.9.0版本开始，引入了阿里巴巴的 Blink ，对 FIink TabIe & SQL 模块做了重大的重构，保留了 Flink Planner 的同时，引入了 Blink PIanner，没引入以前，Flink 没考虑流批作业统一，针对流批作业，底层实现两套代码，引入后，基于流批一体理念，重新设计算子，以流为核心，流作业和批作业最终都会被转为transformation。

#### 84、Flink SQL 使用哪种解析器？

Flink SQL使用 Apache Calcite作为解析器和优化器。

**Calcite** 一种动态数据管理框架，它具备很多典型数据库管理系统的功能 如SQL 解析、 SQL 校验、 SQL 查询优化、 SQL 生成以及数据连接查询等，但是又省略了一些关键的功能，如 Calcite并**不存储**相关的**元数据**和**基本数据**，不完全包含相关处理数据的算法等。

#### 85、Calcite主要功能包含哪些？

Calcite 主要包含以下五个部分：

**(1) SQL 解析 (Parser)**

Calcite SQL 解析是**通过 JavaCC** 实现的，使用 JavaCC 编写 SQL 语法描述文件，将 SQL 解析成未经校验的 AST 语法树。 

**(2) SQL 校验 （Validato）**

校验分两部分 

1）无状态的校验  即验证 SQL 语句是否符合规范。

2）有状态的校验 即通过与元数据结合验证 SQL 中的 Schema、Field、 Function 是否存 在，输入输出类型是否匹配等。 

**(3) SQL 查询优化** 

对上个步骤的输出( RelNode ，逻辑计划树)进行优化，得到优化后的物理执行计划 优化有两种：**基于规则的优化** 和 **基于代价的优化**，后面会详细介绍。

**(4) SQL 生成**  

将物理执行计划生成为在特定平台/引擎的可执行程序，如生成符合 MySQL 或 Oracle 等不同平台规则的 SQL 查询语句等。 

**(5) 数据连接与执行** 

通过各个执行平台执行查询，得到输出结果。 

在Flink 或者其他使用 Calcite 的大数据引擎中，一般到 SQL 查询优化即结束，由各个平台结合 Calcite SQL 代码生成 和 平台实现的代码生成，将优化后的物理执行计划组合成可执行的代码，然后在内存中编译执行。

#### 86、Flink SQL 处理流程说一下？

下面举个例子，详细描述一下Flink Sql的处理流程，如下图所示：

![1636856581338](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102302-57181.png)

我们写一张source表，来源为kafka，当执行create table log_kafka之后 Flink SQL将做如下操作：

![1636856613343](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102334-470392.png)

（1）首先，FlinkSQL 底层使用的是 **apache Calcite** 引擎来处理SQL语句，Calcite会使用javaCC做SQL解析，javaCC根据Calcite中定义的 **Parser.jj** 文件，生成一系列的java代码，生成的java代码会**把SQL转换成AST抽象语法树（即SQLNode类型）**。

（2）生成的 SqlNode 抽象语法树，他是一个**未经验证的抽象语法树**，这时，SQL Validator 会获取 Flink Catalog中的元数据信息来验证 sql 语法，元数据信息检查包括表名，字段名，函数名，数据类型等检查。然后**生成一个校验后的SqlNode**。

（3）到达这步后，只是将 SQL 解析到 java 数据结构的固定节点上，并没有给出相关节点之间的关联关系以及每个节点的类型信息。所以，还 **需要将 SqlNode 转换为逻辑计划，也就是LogicalPl**an，在转换过程中，会使用 **SqlToOperationConverter** 类，来将SqlNode转换为Operation,Operation会根据SQL语法来执行创建表或者删除表等操作，同时**FlinkPlannerImpl.rel()方法会将SQLNode转换成RelNode树，并返回RelRoot**。

（4）第4步将**执行 Optimize 操作**，按照预定义的优化规则 RelOptRule 优化逻辑计划。

Calcite中的优化器RelOptPlanner有两种，一是**基于规则优化（RBO）的HepPlanner**，二是**基于代价优化（CBO）的VolcanoPlanner**。然后得到优化后的RelNode,  再基于Flink里面的rules**将优化后的逻辑计划转换成物理计划**。

（5）第5步 **执行 execute 操作**，会通过代码生成 transformation,然后递归遍历各节点，将DataStreamRelNode 转换成DataStream, 在这期间，会依次递归调用DataStreamUnion、DataStreamCalc、DataStreamScan类中重写的 translateToPlan方法。递归调用各节点的translateToPlan，实际是利用CodeGen元编成Flink的各种算子，相当于直接利用Flink的DataSet或者DataStream开发程序。

（6）最后进一步**编译**成可执行的 JobGraph 提交运行。

#### 87、Flink SQL包含哪些优化规则？

如下图为执行流程图

![1636856644137](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102406-730684.png)

**总结就是：**

先解析，然后验证，将SqlNode转化为Operation来创建表，然后调用rel方法将sqlNode变成 逻辑计划 （RelNodeTree）紧接着对逻辑计划进行优化；

优化之前 会根据Calcite中的优化器中的基于规则优化的HepPlanner**针对四种规则进行预处理**，处理完之后得到Logic RelNode，紧接着**使用代价优化的VolcanoPlanner使用**  Logical_Opt_Rules（逻辑计划优化）找到最优的执行Planner,并转换为FlinkLogical RelNode。

最后运用 Flink包含的优化规则，如DataStream_Opt_Rules：流式计算优化，DataStream_Deco_Rules：装饰流式计算优化   将优化后的逻辑计划转换为物理计划。

**优化规则包含如下：**

Table_subquery_rules  子查询优化

Expand_plan_rules：扩展计划优化

Post_expand_clean_up_rules：扩展计划优化

Datastream_norm_rules：正常化流处理                        

Logical_Opt_Rules：逻辑计划优化          

DataStream_Opt_Rules：流式计算优化         

DataStream_Deco_Rules：装饰流式计算优化

#### 88、Flink SQL中涉及到哪些operation？

**先介绍一下什么是Operation**

在Flink SQL中吗，涉及的DDL，DML，DQL操作都是Operation，在 Flink内部表示，Operation可以和SqlNode对应起来。

**Operation执行在优化前**，执行的函数为executeQperation,如下图所示，为执行的所有Operation。

![1636856679270](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102440-554664.png)

#### 89、Flink Hive有没有使用过？

Flink社区在Flink1.11版本进行了重大改变，如下图所示：

![1636856701686](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102502-135215.png)

#### 90、Flink与Hive集成时都做了哪些操作？

如下所示为Flink与HIve进行连接时的执行图：

![1636856723843](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102525-68993.png)

（1）Flink1.1新引入了Hive方言，所以在Flink SQL中可以编写HIve语法，即Hive Dialect。

（2）编写HIve SQL后，FlinkSQL Planner 会将SQL进行解析，验证，转换成逻辑计划，物理计划，最终变成Jobgraph。

（3）**HiveCatalog作为Flink和Hive的表元素持久化介质，会将不同会话的Flink元数据存储到Hive Metastore中**。用户利用HiveCatalog可以将hive表或者 Kafka表存储到Hive Metastore中。

BlinkPlanner 是在Flink1.9版本新引入的机制，Blink 的查询处理器则实现流批作业接口的统一，**底层的 API 都是Transformation**。真正实现 流 &批 的统一处理，替代原FlinkPlanner将流&批区分处理的方式。在1.11版本后 已经默认为Blink Planner。

#### 91、HiveCatalog类包含哪些方法？

重点方法如下：

![1636856750482](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102551-600723.png)

HiveCatalog主要是**持久化元数据**，所以 一般的创建类型都包含，如 database,Table,View，Function,Partition,还有is_Generic字段判断等。

#### 92、Flink SQL1.11新增了实时数仓功能，介绍一下？

Flink1.11 版本新增的一大功能是实时数仓，可以实时的将kafka中的数据插入Hive中，传统的实时数仓基于 Kafka+ Flinkstreaming，定义全流程的流计算作业，有着秒级甚至毫秒的实时性，**但实时数仓的一个问题是历史数据只有 3-15天，无法在其上做 Ad-hoc的查询。**

针对这个特点，Flink1.11 版本将 **FlieSystemStreaming Sink** 重新修改，增加了**分区提交**和**滚动策略机制**，让HiveStreaming  sink 重新使用文件系统流接收器。

Flink 1.11 的 Table/SQL API 中，**FileSystemConnector 是靠增强版 StreamingFileSink组件实现**，在源码中名为 StreamingFileWriter。***** **只有在Checkpoint 成功时，StreamingFileSink写入的文件才会由 Pending状态变成 Finished状态，从而能够安全地被下游读取。所以，我们一定要打开 Checkpointing，并设定合理的间隔。**

#### 93、Flink -Hive实时写数据介绍下？

**StreamingWrite**，从kafka 中实时拿到数据，使用分区提交将数据从Kafka写入Hive表中，并运行批处理查询以读取该数据。

Flink -SQL 写法

**Source源**

![1636856784769](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102625-242079.png)

**Sink目的地**

![1636856808207](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102649-40833.png)

**Insert 插入**

![1636856824649](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102705-665159.png)

Flink-table写法：

**Source源**

![1636856847429](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102728-707378.png)

**Sink目的地**

![1636856871980](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102753-264989.png)

**Insert 插入**

![1636856888096](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102808-930650.png)

#### 94、Flink -Hive实时读数据介绍下？

**如下图所示：**

![1636856910232](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102831-174526.png)

Flink 源码中在对Hive进行读取操作时，会经历以下几个步骤：

1、Flink都是基于calcite先解析sql，确定表来源于hive，如果是Hive表，将会**在HiveCatalog中创建HiveTableFactory**

2、HiveTableFactory 会基于配置文件**创建 HiveTableSource**，然后HiveTableSource在真正执行时，会调用getDataStream方法，通过getDataStream方法来确定查询匹配的分区信息，然后创建表对应的InputFormat，然后确定并行度，根据并行度确定slot 分发HiveMapredSplitReader任务。

3、在TaskManager端的slot中，Split会确定读取的内容，基于Hive中定义的序列化工具，InputFormat执行读取反序列化，得到value值。

4、最后循环执行reader.next 获取value，将其解析成Row。

#### 95、Flink -Hive实时写数据时，如何保证已经写入分区的数据何时才能对下游可见呢？

**如下图所示：**

![1636856934691](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102855-593449.png)

首先可以看一下，在实时的将数据存储到Hive数仓中，FileSystemConnector 为了与 Flink-Hive集成的大环境适配，**最大的改变就是分区提交**，可以看一下左下图，官方文档给出的，分区可以采取日期+ 小时的策略，或者时分秒的策略。

**那如何保证已经写入分区的数据何时才能对下游可见呢？** 这就和 **触发机制** 有关， 触发机制包含process-time和 partition-time以及时延。

**partition-time** 指的是根据事件时间中提取的分区触发。当'watermark' > 'partition-time' + 'delay' ，选择partition-time的数据才能提交成功，

**process-time** 指根据系统处理时间触发，当加上时延后，要想让分区进行提交，当'currentprocessing time' > 'partition creation time' + 'delay'  选择 process-time的数据可以提交成功。

但选择process-time触发机制会有缺陷，就是当数据迟到或者程序失败重启时，数据不能按照事件时间被归入正确分区。所以 一般会选择 partition-time。

#### 96、源码中分区提交的PartitionCommitTrigger介绍一下？

**在源码中，PartitionCommitTrigger类图如下所示**

![1636856966742](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/102927-374152.png)

该类中维护了两对必要的信息：

1. pendingPartitions/pendingPartitionsState：等待提交的分区以及对应的状态
2. watermarks/watermarksState：watermarks（用 TreeMap 存储以保证有序）以及对应的状态。

#### 97、PartitionTimeCommitTigger 是如何知道该提交哪些分区的呢？（源码分析）

1、检查checkpoint ID 是否合法；

2、取出当前checkpoint ID 对应的水印，并调用 TreeMap的headMap() 和 clear() 方法删掉早于当前 checkpoint ID的水印数据（没用了）；

3、遍历等待提交的分区，调用之前定义的PartitionTimeExtractor。

（比如${year}-${month}-${day} ${hour}:00:00）抽取分区时间。

如果watermark>partition-time+delay，说明可以提交，并返回它们

#### 98、如何保证已经写入分区的数据对下游可见的标志问题（源码分析）

**在源码中，主要涉及**PartitionCommitPolicy类，如下图所示：

![1636857014876](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103015-736927.png)

#### 99、Flink SQL CEP有没有接触过？

**CEP的概念：**

- 复杂事件处理（Complex Event Processing），用于识别输入流中符合指定规则的事件，并按照指定方式输出。
- 起床—>洗漱—>吃饭—>上班一系列串联起来的事件流形成的模式
- 浏览商品—>加入购物车—>创建订单—>支付完成—>发货—>收货事件流形成的模式。

通过概念可以了解，CEP主要是识别输入流中用户指定的一些基本规则的事件，然后将这些事件再通过指定方式输出。

如下图所示： 我们指定“方块、圆”为基本规则的事件，在输入的原始流中，将这些事件作为一个结果流输出来。

![1636857081891](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103123-657989.png)

**CEP的使用场景：**

![1636857109942](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103151-518840.png)

像用户异常检测：我们指定异常操作事件为要输出的结果流；策略营销：指定符合要求的事件为结果流；运维监控：指定一定范围的指标为结果流；银行卡盗刷：指定同一时刻在两个地方被刷两次为异常结果流。

**Flink CEP SQL 语法 是通过SQL方式进行复杂事件处理，但是与** Flink SQL语法也不太相同，其中包含许多规则。

#### 100、Flink SQL CEP了解的参数介绍一下？

**CEP包含的参数如下：**

![1636857148245](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103229-610723.png)

**参数介绍**

p**输出模式（**每个找到的匹配项应该输出多少行）

- one  row  per  match

每次检测到完整的匹配后进行汇总输出

- all  rows per match (flink暂不支持)

检测到完整的匹配后会把匹配过程中每条具体记录进行输出

p**runningVS final语义**

- 在计算中使用那些匹配的事件

running匹配中和final匹配结束

- define语句中只可以使用running,measure两者都可以

- 输出结果区别

对于one row per match，输出没区别

对于all  rows  per match ，输出不同

![图片](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103325-949181.png)

**3、匹配后跳转模式介绍**

after  match（匹配后，从哪里开始重新匹配）

- bskip to next row

从匹配成功的事件序列中的第一个事件的下一个事件开始进行下一次匹配

- skip  past last row

从匹配成功的事件序列中的最后一个事件的下一个事件开始进行下一次匹配

- skip  to first pattern  Item  

从匹配成功的事件序列中第一个对应于patternItem的事件开始进行下一次匹配     

- skip  to last pattern  Item

从匹配成功的事件序列中最后一个对应于patternItem的事件开始进行下一次匹配

注意：

在使用skip to first/last patternItem容易出现循环匹配问题，需要慎重

针对上面的匹配后跳转模式分别介绍:

**（1）after  match   skip past  last  row 如下图**

![1636857264129](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103425-916303.png)

**（2）after  match   skip to  next  row 如下图**

![1636857288607](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103449-955883.png)

**（3）after  match   skip to  last patternItem 如下图**

![1636857311033](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103511-7948.png)

**（4）after  match   skip to  first  patternItem 如下图**

![1636857333868](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103534-295589.png)

#### 101、编写一个CEP SQL案例，如银行卡盗刷

通过Flink CEP SQL 写的关于金融场景 银行卡盗刷案例。

案例介绍：在金融场景中，有时会出现银行卡盗刷现象，犯罪分子利用互联网等技术，在间隔10分钟或者更短时间内，使一张银行卡在不同的两个地方出现多次刷卡记录，这从常规操作来说，在间隔时间很多的情况下，用户是无法同时在两个城市进行刷卡交易的，所以出现这种问题，就需要后台做出触发报警机制。

要求：当相同的cardId在十分钟内，从两个不同的Location发生刷卡现象，触发报警机制，以便检测信用卡盗刷现象。

![1636857374327](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103616-361824.png)

（1）编写cep sql时，包含许多技巧，首先我们编写最基础的查询语句,从一张表中查询需要的字段。

```java
select starttime,endtime,cardId,event from dataStream
```

（2）match_recognize();

该字段是CEP SQL 的前提条件，用于生成一个追加表，所有的 CEP SQL都是书写在这里面。

（3）分区，排序

由于是对同一ID，所以需要使用 partition by,还要根据时间进行排序 order by

（4）理解CEP  SQL核心的编写顺序

如上图标的顺序

![1636857434228](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103722-634765.png)

#### 102、Flink CDC了解吗？什么是 Flink SQL CDC Connectors？

**在 Flink 1.11 引入了 CDC 机制**，CDC 的全称是 **Change Data Capture**，用于捕捉数据库表的增删改查操作，是目前非常成熟的同步数据库变更方案。

Flink CDC Connectors 是 Apache Flink 的一组源连接器，是可以从 MySQL、PostgreSQL 数据直接读取全量数据和增量数据的 Source Connectors，开源地址：`https://github.com/ververica/flink-cdc-connectors。`

目前(1.13版本)支持的 Connectors 如下：      

![1636857474904](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103755-973244.png)

另外**支持解析 Kafka 中 debezium-json 和 canal-json 格式的 Change Log**，通过Flink 进行计算或者直接写入到其他外部数据存储系统(比如 Elasticsearch)，或者将 Changelog Json 格式的 Flink 数据写入到 Kafka:

![1636857489501](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103810-319387.png)

Flink CDC Connectors 和 Flink 之间的版本映射:

![1636857503952](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103824-211741.png)

#### 103、Flink CDC原理介绍一下

在最新CDC 调研报告中，**Debezium** 和 **Canal** 是目前最流行使用的 CDC 工具，这些 CDC 工具的核心原理是抽取数据库日志获取变更。在经过一系列调研后，目前Debezium (支持全量、增量同步，同时支持 MySQL、PostgreSQL、Oracle 等数据库)，使用较为广泛。

Flink SQL CDC 内置了 **Debezium** 引擎，利用其抽取日志获取变更的能力，将 changelog 转换为 Flink SQL 认识的 RowData 数据。（以下右侧是 Debezium 的数据格式，左侧是 Flink 的 **RowData 数据格式**）。

![1636857524440](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103845-662251.png)

RowData 代表了一行的数据，在 RowData 上面会有一个元数据的信息 **RowKind**，RowKind 里面包括了插入(+I)、更新前(-U)、更新后(+U)、删除(-D)，这样和数据库里面的 binlog 概念十分类似。通过 Debezium 采集的数据，包含了旧数据(before)和新数据行(after)以及原数据信息(source)，op 的 u 表示是 update 更新操作标识符（op 字段的值 c，u，d，r 分别对应 create，update，delete，reade），ts_ms 表示同步的时间戳。

#### 104、通过CDC设计一种Flink SQL 采集+计算+传输(ETL)一体化的实时数仓

设计图如下：

![1636857550032](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/103911-530055.png)

**通过 Flink CDC connectors 替换 Debezium+Kafka 的数据采集模块**，**实现** **Flink SQL 采集+计算+传输(ETL)一体化**，以Mysql为Source源，Flink CDC中间件为插件，ES或者Kafka，或者其他为Sink，这样设计的优点如下：

- 开箱即用，简单易上手
- 减少维护的组件，简化实时链路，减轻部署成本
- 减小端到端延迟
- Flink 自身支持 Exactly Once 的读取和计算
- 数据不落地，减少存储成本
- 支持全量和增量流式读取
- binlog 采集位点可回溯

#### 105、Flink SQL CDC如何实现一致性保障（源码分析）

Flink SQL CDC 用于获取数据库变更日志的 Source 函数是 **DebeziumSourceFunction**，且最终返回的类型是 **RowData**，该函数实现了 CheckpointedFunction，即通过 Checkpoint 机制来保证发生 failure 时不会丢数，实现 exactly once 语义，这部分在函数的注释中有明确的解释。

```java
/** * The {@link DebeziumSourceFunction} is a streaming data source that pulls captured change data * from databases into Flink. * 通过Checkpoint机制来保证发生failure时不会丢数，实现exactly once语义 * <p>The source function participates in checkpointing and guarantees that no data is lost * during a failure, and that the computation processes elements "exactly once". * 注意：这个Source Function不能同时运行多个实例 * <p>Note: currently, the source function can't run in multiple parallel instances. * * <p>Please refer to Debezium's documentation for the available configuration properties: * https://debezium.io/documentation/reference/1.2/development/engine.html#engine-properties</p> */
@PublicEvolvingpublic class DebeziumSourceFunction<T> extends RichSourceFunction<T> implementsCheckpointedFunction,ResultTypeQueryable<T> {}
```

**为实现 CheckpointedFunction，需要实现以下两个方法：**

```java
public interface CheckpointedFunction {
  //做快照，把内存中的数据保存在checkpoint状态中void snapshotState(FunctionSnapshotContext var1) throws Exception;
//程序异常恢复后从checkpoint状态中恢复数据void initializeState(FunctionInitializationContext var1) throws Exception;}
```

**接下来我们看看 DebeziumSourceFunction 中都记录了哪些状态。**

```java
/** Accessor for state in the operator state backend.     offsetState中记录了读取的binlog文件和位移信息等，对应Debezium中的*/
private transient ListState<byte[]> offsetState;
/** * State to store the history records, i.e. schema changes. * historyRecordsState记录了schema的变化等信息 * @see FlinkDatabaseHistory*/
private transient ListState<String> historyRecordsState;
```

我们发现在 Flink SQL CDC 是一个相对简易的场景，没有中间算子，是通过 Checkpoint 持久化 binglog 消费位移和 schema 变化信息的快照，来实现 Exactly Once。

#### 106、Flink SQL GateWay了解吗？

**Flink SQL GateWay的概念：**

FlinkSql Gateway是Flink集群的“任务网关”，支持以restapi 的形式提交查询、插入、删除等任务，如下图所示：

![1636857658426](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/104059-894712.png)

总体架构如下图所示：

![1636857674505](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/104115-127885.png)

#### 107、Flink SQL GateWay创建会话讲解一下？

**创建会话流程图如下：**

![1636857694946](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/14/104136-965836.png)

（1）传入参数包含name名称、planner执行引擎（Blink或原生的flink）、executetype（streaming或者batch）、properties（配置参数，如并发度等）；

（2）在SessionMnager中，会根据这些参数创建对应的SessionContext；

  SessionContext sessionContext= new SessionContext(sessionName, sessionId, sessionEnv, defaultContext);

（3）将创建Session放入Map集合中，最后返回对应的SessionId，方便后续使用。

   sessions.put(sessionId,session); return sessionId;

#### 108、Flink SQL GateWay如何处理并发请求？多个提交怎么处理？ 

sql gateway内部维护SessionManager，里面通过Map维护了各个Session，每个Session的任务执行是独立的。同一个Session通过ExecuteContext内部的tEnv按顺序提交。

#### 109、如何维护多个SQL之间的关联性？ 

 在每个Session中单独维护了tEnv，同一个session中的操作其实是在一个env中执行的。因此只要是同一个session中的任务，内部使用的tEnv就是同一个。这样就可以实现在Asession中，先创建一个view，然后执行一个select，最后执行一个insert。

#### 110、sql字符串如何提交到集群成为代码？

**Session**中维护了tenv，sql会通过tenv编译生成**pipeline**（即DAG图），在batch模式下是Plan执行计划；在stream模式下是**StreamGraph**。然后Session内部会创建一个ProgramDeployer代码发布器，根据Flink中配置的target创建不同的excutor。最后调用executor.execute方法提交Pipeline和config执行。