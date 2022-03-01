
<!-- TOC -->

- [Spark](#spark)
- [一、Spark 基础](#一spark-基础)
  - [1. Spark 发展史](#1-spark-发展史)
  - [2. Spark 为什么会流行](#2-spark-为什么会流行)
  - [3. Spark VS Hadoop](#3-spark-vs-hadoop)
  - [3. Spark 特点](#3-spark-特点)
  - [4. Spark 运行模式](#4-spark-运行模式)
  - [Spark 为什么比 MapReduce 快？](#spark-为什么比-mapreduce-快)
- [二、Spark Core](#二spark-core)
  - [1. RDD 详解](#1-rdd-详解)
    - [1) 为什么要有 RDD?](#1-为什么要有-rdd)
    - [2) RDD 是什么?](#2-rdd-是什么)
    - [3) RDD 主要属性](#3-rdd-主要属性)
  - [2. RDD-API](#2-rdd-api)
    - [1) RDD 的创建方式](#1-rdd-的创建方式)
    - [2) RDD 的算子分类](#2-rdd-的算子分类)
    - [3) Transformation 转换算子](#3-transformation-转换算子)
    - [4) Action 动作算子](#4-action-动作算子)
    - [4) RDD 算子练习](#4-rdd-算子练习)
    - [你知道 reduceByKey 和 groupByKey 有啥区别吗？](#你知道-reducebykey-和-groupbykey-有啥区别吗)
    - [你知道  reduceByKey、foldByKey、aggregateByKey、combineByKey 区别吗？](#你知道--reducebykeyfoldbykeyaggregatebykeycombinebykey-区别吗)
    - [你刚才提到了 DAG,能说一下什么是 DAG？](#你刚才提到了-dag能说一下什么是-dag)
    - [Spark 广播变量和累加器介绍一下？](#spark-广播变量和累加器介绍一下)
    - [广播变量和累加器的区别是啥？](#广播变量和累加器的区别是啥)
    - [如何使用 Spark 实现 TopN 的获取（描述思路）](#如何使用-spark-实现-topn-的获取描述思路)
  - [3. RDD 的持久化/缓存](#3-rdd-的持久化缓存)
    - [持久化/缓存 API 详解](#持久化缓存-api-详解)
  - [4. RDD 容错机制 Checkpoint](#4-rdd-容错机制-checkpoint)
  - [5. RDD 依赖关系](#5-rdd-依赖关系)
    - [1) 宽窄依赖](#1-宽窄依赖)
    - [2) 为什么要设计宽窄依赖](#2-为什么要设计宽窄依赖)
  - [6. DAG 的生成和划分 Stage](#6-dag-的生成和划分-stage)
    - [1) DAG 介绍](#1-dag-介绍)
    - [2) DAG 划分 Stage](#2-dag-划分-stage)
  - [7. RDD 累加器和广播变量](#7-rdd-累加器和广播变量)
    - [1) 累加器](#1-累加器)
    - [2) 广播变量](#2-广播变量)
- [三、Spark SQL](#三spark-sql)
  - [1. 数据分析方式](#1-数据分析方式)
    - [1) 命令式](#1-命令式)
    - [2) SQL](#2-sql)
    - [3) 总结](#3-总结)
  - [2. SparkSQL 前世今生](#2-sparksql-前世今生)
    - [1) 发展历史](#1-发展历史)
  - [3. Hive 和 SparkSQL](#3-hive-和-sparksql)
  - [4. 数据分类和 SparkSQL 适用场景](#4-数据分类和-sparksql-适用场景)
    - [1) 结构化数据](#1-结构化数据)
    - [2) 半结构化数据](#2-半结构化数据)
    - [3) 总结](#3-总结-1)
  - [5. Spark SQL 数据抽象](#5-spark-sql-数据抽象)
    - [1) DataFrame](#1-dataframe)
    - [2) DataSet](#2-dataset)
    - [3) RDD、DataFrame、DataSet 的区别](#3-rdddataframedataset-的区别)
    - [4) 总结](#4-总结)
  - [6. Spark SQL 应用](#6-spark-sql-应用)
    - [1) 创建 DataFrame/DataSet](#1-创建-dataframedataset)
    - [2) 两种查询风格：DSL 和 SQL](#2-两种查询风格dsl-和-sql)
    - [3) Spark SQL 完成 WordCount](#3-spark-sql-完成-wordcount)
    - [4) Spark SQL 多数据源交互](#4-spark-sql-多数据源交互)
- [四、Spark Streaming](#四spark-streaming)
  - [1. 整体流程](#1-整体流程)
  - [Spark Streaming 如何执行流式计算的?](#spark-streaming-如何执行流式计算的)
  - [使用 Spark Streaming 写一个 WordCount?](#使用-spark-streaming-写一个-wordcount)
  - [使用 Spark Streaming 常用算子有哪些?](#使用-spark-streaming-常用算子有哪些)
  - [使用 Spark Streaming 有状态装换的算子有哪些?](#使用-spark-streaming-有状态装换的算子有哪些)
  - [Spark Streaming 如何实现精确一次消费?](#spark-streaming-如何实现精确一次消费)
  - [Spark Streaming 背压机制了解吗？](#spark-streaming-背压机制了解吗)
  - [SparkStreaming 有哪几种方式消费 Kafka 中的数据，它们之间的区别是什么？](#sparkstreaming-有哪几种方式消费-kafka-中的数据它们之间的区别是什么)
  - [2. 数据抽象](#2-数据抽象)
  - [3. DStream 相关操作](#3-dstream-相关操作)
    - [1) Transformations](#1-transformations)
    - [2) Output/Action](#2-outputaction)
  - [4. Spark Streaming 完成实时需求](#4-spark-streaming-完成实时需求)
    - [1) WordCount](#1-wordcount)
    - [2) updateStateByKey](#2-updatestatebykey)
    - [3) reduceByKeyAndWindow](#3-reducebykeyandwindow)
- [五、Structured Streaming](#五structured-streaming)
  - [1. API](#1-api)
  - [2. 核心思想](#2-核心思想)
  - [3. 应用场景](#3-应用场景)
  - [4. Structured Streaming 实战](#4-structured-streaming-实战)
    - [1) 读取 Socket 数据](#1-读取-socket-数据)
    - [2) 读取目录下文本数据](#2-读取目录下文本数据)
    - [3) 计算操作](#3-计算操作)
    - [4) 输出](#4-输出)
- [六、Spark 的两种核心 Shuffle](#六spark-的两种核心-shuffle)
  - [Spark Shuffle](#spark-shuffle)
  - [一、Hash Shuffle 解析](#一hash-shuffle-解析)
    - [1. HashShuffleManager](#1-hashshufflemanager)
    - [2. 优化的 HashShuffleManager](#2-优化的-hashshufflemanager)
      - [基于 Hash 的 Shuffle 机制的优缺点](#基于-hash-的-shuffle-机制的优缺点)
  - [二、SortShuffle 解析](#二sortshuffle-解析)
    - [1. 普通运行机制](#1-普通运行机制)
    - [2. bypass 运行机制](#2-bypass-运行机制)
    - [3. Tungsten Sort Shuffle 运行机制](#3-tungsten-sort-shuffle-运行机制)
      - [基于 Sort 的 Shuffle 机制的优缺点](#基于-sort-的-shuffle-机制的优缺点)
- [七、Spark 底层执行原理](#七spark-底层执行原理)
  - [Spark 运行流程](#spark-运行流程)
    - [1. 从代码角度看 DAG 图的构建](#1-从代码角度看-dag-图的构建)
    - [2. 将 DAG 划分为 Stage 核心算法](#2-将-dag-划分为-stage-核心算法)
    - [3. 将 DAG 划分为 Stage 剖析](#3-将-dag-划分为-stage-剖析)
    - [4. 提交 Stages](#4-提交-stages)
    - [5. 监控 Job、Task、Executor](#5-监控-jobtaskexecutor)
    - [6. 获取任务执行结果](#6-获取任务执行结果)
    - [7. 任务调度总体诠释](#7-任务调度总体诠释)
  - [Spark 运行架构特点](#spark-运行架构特点)
    - [1. Executor 进程专属](#1-executor-进程专属)
    - [2. 支持多种资源管理器](#2-支持多种资源管理器)
    - [3. Job 提交就近原则](#3-job-提交就近原则)
    - [4. 移动程序而非移动数据的原则执行](#4-移动程序而非移动数据的原则执行)
- [八、Spark 数据倾斜](#八spark-数据倾斜)
  - [1. 预聚合原始数据](#1-预聚合原始数据)
  - [2. 预处理导致倾斜的key](#2-预处理导致倾斜的key)
  - [3. 提高reduce并行度](#3-提高reduce并行度)
  - [4. 使用map join](#4-使用map-join)
- [九、Spark性能优化](#九spark性能优化)
- [Spark调优之RDD算子调优](#spark调优之rdd算子调优)
  - [1. RDD复用](#1-rdd复用)
  - [2. 尽早filter](#2-尽早filter)
  - [3. 读取大量小文件-用wholeTextFiles](#3-读取大量小文件-用wholetextfiles)
  - [4. mapPartition和foreachPartition](#4-mappartition和foreachpartition)
  - [5. filter+coalesce/repartition(减少分区)](#5-filtercoalescerepartition减少分区)
  - [6. 并行度设置](#6-并行度设置)
  - [7. repartition/coalesce调节并行度](#7-repartitioncoalesce调节并行度)
  - [8. reduceByKey本地预聚合](#8-reducebykey本地预聚合)
  - [9. 使用持久化+checkpoint](#9-使用持久化checkpoint)
  - [10. 使用广播变量](#10-使用广播变量)
  - [11. 使用Kryo序列化](#11-使用kryo序列化)
- [Spark调优之Shuffle调优](#spark调优之shuffle调优)
  - [1. map和reduce端缓冲区大小](#1-map和reduce端缓冲区大小)
  - [2. reduce端重试次数和等待时间间隔](#2-reduce端重试次数和等待时间间隔)
  - [3. bypass机制开启阈值](#3-bypass机制开启阈值)
- [十、故障排除](#十故障排除)
  - [1. 避免OOM-out of memory](#1-避免oom-out-of-memory)
  - [2. 避免GC导致的shuffle文件拉取失败](#2-避免gc导致的shuffle文件拉取失败)
  - [3. YARN-CLIENT模式导致的网卡流量激增问题](#3-yarn-client模式导致的网卡流量激增问题)
  - [4. YARN-CLUSTER模式的JVM栈内存溢出无法执行问题](#4-yarn-cluster模式的jvm栈内存溢出无法执行问题)
  - [5. 避免SparkSQL JVM栈内存溢出](#5-避免sparksql-jvm栈内存溢出)
- [十一、Spark大厂面试真题](#十一spark大厂面试真题)
    - [1. 通常来说，Spark与MapReduce相比，Spark运行效率更高。请说明效率更高来源于Spark内置的哪些机制？](#1-通常来说spark与mapreduce相比spark运行效率更高请说明效率更高来源于spark内置的哪些机制)
    - [2. hadoop和spark使用场景？](#2-hadoop和spark使用场景)
    - [3. spark如何保证宕机迅速恢复?](#3-spark如何保证宕机迅速恢复)
    - [4. hadoop和spark的相同点和不同点？](#4-hadoop和spark的相同点和不同点)
    - [5. RDD持久化原理？](#5-rdd持久化原理)
    - [6. checkpoint检查点机制？](#6-checkpoint检查点机制)
    - [7. checkpoint和持久化机制的区别？](#7-checkpoint和持久化机制的区别)
    - [8. RDD机制理解吗？](#8-rdd机制理解吗)
    - [9. Spark streaming以及基本工作原理？](#9-spark-streaming以及基本工作原理)
    - [10. DStream以及基本工作原理？](#10-dstream以及基本工作原理)
    - [11. spark有哪些组件？](#11-spark有哪些组件)
    - [12. spark工作机制？](#12-spark工作机制)
    - [13. 说下宽依赖和窄依赖](#13-说下宽依赖和窄依赖)
    - [14. Spark主备切换机制原理知道吗？](#14-spark主备切换机制原理知道吗)
    - [15. spark解决了hadoop的哪些问题？](#15-spark解决了hadoop的哪些问题)
    - [16. 数据倾斜的产生和解决办法？](#16-数据倾斜的产生和解决办法)
    - [17. 你用sparksql处理的时候， 处理过程中用的dataframe还是直接写的sql？为什么？](#17-你用sparksql处理的时候-处理过程中用的dataframe还是直接写的sql为什么)
    - [18. RDD中reduceBykey与groupByKey哪个性能好，为什么](#18-rdd中reducebykey与groupbykey哪个性能好为什么)
    - [19. Spark master HA主从切换过程不会影响到集群已有作业的运行，为什么](#19-spark-master-ha主从切换过程不会影响到集群已有作业的运行为什么)
    - [20. spark master使用zookeeper进行ha，有哪些源数据保存到Zookeeper里面](#20-spark-master使用zookeeper进行ha有哪些源数据保存到zookeeper里面)

<!-- /TOC -->

## Spark

![1639224368992](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/200610-323256.png)

## 一、Spark 基础

### 1. Spark 发展史

大数据、人工智能( Artificial Intelligence )像当年的石油、电力一样， 正以前所未有的广度和深度影响所有的行业， 现在及未来公司的核心壁垒是数据， 核心竞争力来自基于大数据的人工智能的竞争。

Spark 是当今大数据领域最活跃、最热门、最高效的大数据通用计算平台之一。

**2009 年诞生于美国加州大学伯克利分校 AMP 实验室**；
2010 年通过 BSD 许可协议开源发布；
2013 年捐赠给 Apache 软件基金会并切换开源协议到切换许可协议至 Apache2.0；
**2014 年 2 月，Spark 成为 Apache 的顶级项目**；
2014 年 11 月, Spark 的母公司 Databricks 团队使用 Spark 刷新数据排序世界记录。

Spark 成功构建起了一体化、多元化的大数据处理体系。在任何规模的数据计算中， Spark 在性能和扩展性上都更具优势。

1. Hadoop 之父 Doug Cutting 指出：Use of MapReduce engine for Big Data projects will decline, replaced by Apache Spark (大数据项目的 MapReduce 引擎的使用将下降，由 Apache Spark 取代)。
2. Hadoop 商业发行版本的市场领导者 Cloudera 、HortonWorks 、MapR 纷纷转投 Spark,并把 Spark 作为大数据解决方案的首选和核心计算引擎。

2014 年的 Benchmark 测试中， Spark 秒杀 Hadoop ，在使用十分之一计算资源的情况下，相同数据的排序上， Spark 比 MapReduce 快 3 倍！在没有官方 PB 排序对比的情况下，首次将 Spark 推到了 IPB 数据(十万亿条记录) 的排序，在使用 190 个节点的情况下，工作负载在 4 小时内完成， 同样远超雅虎之前使用 3800 台主机耗时 16 个小时的记录。

在 FullStack 理想的指引下，**Spark 中的 Spark SQL 、SparkStreaming 、MLLib 、GraphX 、R 五大子框架和库之间可以无缝地共享数据和操作**， 这不仅打造了 Spark 在当今大数据计算领域其他计算框架都无可匹敌的优势， 而且使得 Spark 正在加速成为大数据处理中心首选通用计算平台。

### 2. Spark 为什么会流行

- 原因 1：**优秀的数据模型和丰富计算抽象**

Spark 产生之前，已经有 MapReduce 这类非常成熟的计算系统存在了，并提供了高层次的 API(map/reduce)，把计算运行在集群中并提供容错能力，从而实现分布式计算。

虽然 MapReduce 提供了对数据访问和计算的抽象，但是对于数据的复用就是简单的将中间数据写到一个稳定的文件系统中(例如 **HDFS**)，所以会产生数据的复制备份，磁盘的 I/O 以及数据的序列化，所以在遇到需要在多个计算之间复用中间结果的操作时效率就会非常的低。而这类操作是非常常见的，例如**迭代式计算，交互式数据挖掘，图计算**等。

认识到这个问题后，学术界的 AMPLab 提出了一个新的模型，叫做 RDD。**RDD 是一个可以容错且并行的数据结构(其实可以理解成分布式的集合，操作起来和操作本地集合一样简单)，它可以让用户显式的将中间结果数据集保存在内存中，并且通过控制数据集的分区来达到数据存放处理最优化**.同时 RDD 也提供了丰富的 API (map、reduce、filter、foreach、redeceByKey...)来操作数据集。后来 RDD 被 AMPLab 在一个叫做 Spark 的框架中提供并开源。

简而言之，Spark 借鉴了 MapReduce 思想发展而来，保留了其分布式并行计算的优点并改进了其明显的缺陷。让中间数据存储在内存中提高了运行速度、并提供丰富的操作数据的 API 提高了开发速度。

- 原因 2：**完善的生态圈-fullstack**

![1639224487155](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/200808-707044.png)

目前，Spark 已经发展成为一个包含多个子项目的集合，其中包含 SparkSQL、Spark Streaming、GraphX、MLlib 等子项目。

**Spark Core**：实现了 Spark 的基本功能，包含 RDD、任务调度、内存管理、错误恢复、与存储系统交互等模块。

**Spark SQL**：Spark 用来操作结构化数据的程序包。通过 Spark SQL，我们可以使用 SQL 操作数据。

**Spark Streaming**：Spark 提供的对实时数据进行流式计算的组件。提供了用来操作数据流的 API。

**Spark MLlib**：提供常见的机器学习(ML)功能的程序库。包括分类、回归、聚类、协同过滤等，还提供了模型评估、数据导入等额外的支持功能。

**GraphX(图计算)**：Spark 中用于图计算的 API，性能良好，拥有丰富的功能和运算符，能在海量数据上自如地运行复杂的图算法。

**集群管理器**：Spark 设计为可以高效地在一个计算节点到数千个计算节点之间伸缩计算。

**Structured Streaming**：处理结构化流,统一了离线和实时的 API。

### 3. Spark VS Hadoop

|              |                     Hadoop                     |                      Spark                      |
| :----------: | :--------------------------------------------: | :---------------------------------------------: |
|     类型     |      分布式基础平台, 包含计算, 存储, 调度      |                 分布式计算工具                  |
|     场景     |             大规模数据集上的批处理             |          迭代计算, 交互式计算, 流计算           |
|     价格     |               对机器要求低, 便宜               |             对内存有要求, 相对较贵              |
|   编程范式   |     Map+Reduce, API 较为底层, 算法适应性差     | RDD 组成 DAG 有向无环图, API 较为顶层, 方便使用 |
| 数据存储结构 | MapReduce 中间计算结果存在 HDFS 磁盘上, 延迟大 |       RDD 中间运算结果存在内存中 , 延迟小       |
|   运行方式   |        Task 以进程方式维护, 任务启动慢         |         Task 以线程方式维护, 任务启动快         |

> 注意：
> 尽管 Spark 相对于 Hadoop 而言具有较大优势，但 Spark 并不能完全替代 Hadoop，Spark 主要用于替代 Hadoop 中的 MapReduce 计算模型。存储依然可以使用 HDFS，但是中间结果可以存放在内存中；调度可以使用 Spark 内置的，也可以使用更成熟的调度系统 YARN 等。
>
> 实际上，Spark 已经很好地融入了 Hadoop 生态圈，并成为其中的重要一员，它可以借助于 YARN 实现资源调度管理，借助于 HDFS 实现分布式存储。
>
> 此外，Hadoop 可以使用廉价的、异构的机器来做分布式存储与计算，但是，Spark 对硬件的要求稍高一些，对内存与 CPU 有一定的要求。

### 3. Spark 特点

- 快

与 Hadoop 的 MapReduce 相比，**Spark 基于内存的运算要快 100 倍以上，基于硬盘的运算也要快 10 倍以上**。Spark 实现了高效的 DAG 执行引擎，可以通过基于内存来高效处理数据流。

- 易用

**Spark 支持 Java、Python、R 和 Scala 的 API，还支持超过 80 种高级算法，使用户可以快速构建不同的应用**。而且 Spark 支持交互式的 Python 和 Scala 的 shell，可以非常方便地在这些 shell 中使用 Spark 集群来验证解决问题的方法。

- 通用

Spark 提供了统一的解决方案。**Spark 可以用于批处理、交互式查询(Spark SQL)、实时流处理(Spark Streaming)、机器学习(Spark MLlib)和图计算(GraphX)**。这些不同类型的处理都可以在同一个应用中无缝使用。Spark 统一的解决方案非常具有吸引力，毕竟任何公司都想用统一的平台去处理遇到的问题，减少开发和维护的人力成本和部署平台的物力成本。

- 兼容性

**Spark 可以非常方便地与其他的开源产品进行融合**。比如，Spark 可以使用 Hadoop 的 YARN 和 Apache Mesos 作为它的资源管理和调度器，并且可以处理所有 Hadoop 支持的数据，包括 HDFS、HBase 和 Cassandra 等。这对于已经部署 Hadoop 集群的用户特别重要，因为不需要做任何数据迁移就可以使用 Spark 的强大处理能力。

Spark 也可以不依赖于第三方的资源管理和调度器，它实现了 Standalone 作为其内置的资源管理和调度框架，这样进一步降低了 Spark 的使用门槛，使得所有人都可以非常容易地部署和使用 Spark。此外，Spark 还提供了在 EC2 上部署 Standalone 的 Spark 集群的工具。

### 4. Spark 运行模式

1. local 本地模式(单机)--学习测试使用

   分为 local 单线程和 local-cluster 多线程。

2. standalone 独立集群模式--学习测试使用

   典型的 Mater/slave 模式。

3. standalone-HA 高可用模式--生产环境使用

   基于 standalone 模式，使用 zk 搭建高可用，避免 Master 是有单点故障的。

4. **on yarn 集群模式--生产环境使用**

   运行在 yarn 集群之上，由 yarn 负责资源管理，Spark 负责任务调度和计算。

   好处：计算资源按需伸缩，集群利用率高，共享底层存储，避免数据跨集群迁移。

5. on mesos 集群模式--国内使用较少

   运行在 mesos 资源管理器框架之上，由 mesos 负责资源管理，Spark 负责任务调度和计算。

6. on cloud 集群模式--中小公司未来会更多的使用云服务

   比如 AWS 的 EC2，使用这个模式能很方便的访问 Amazon 的 S3。

### Spark 为什么比 MapReduce 快？

1. Spark 是基于**内存计算**，MapReduce 是基于**磁盘运算**，所以速度快
2. Spark 拥有高效的**调度算法**，是基于 DAG,形成一系列的有向无环图
3. Spark 是通过 RDD 算子来运算的，它拥有两种操作，一种转换操作，一种动作操作，可以将先运算的结果存储在内存中，随后在计算出来.
4. Spark 还拥有**容错机制 Linage**。

## 二、Spark Core

### 1. RDD 详解

#### 1) 为什么要有 RDD?

在许多迭代式算法(比如机器学习、图算法等)和交互式数据挖掘中，不同计算阶段之间会**重用中间结果**，即一个阶段的输出结果会作为下一个阶段的输入。但是，之前的 MapReduce 框架采用**非循环式的数据流模型**，把中间结果写入到 HDFS 中，带来了大量的数据复制、磁盘 IO 和序列化开销。且这些框架只能支持一些特定的计算模式(map/reduce)，并没有提供一种通用的数据抽象。

AMP 实验室发表的一篇关于 RDD 的论文:《Resilient Distributed Datasets: A Fault-Tolerant Abstraction for In-Memory Cluster Computing》就是为了解决这些问题的。

RDD 提供了一个抽象的数据模型，让我们不必担心底层数据的分布式特性，只需将具体的应用逻辑表达为一系列转换操作(函数)，不同 RDD 之间的转换操作之间还可以形成依赖关系，进而实现**管道化**，从而避免了中间结果的存储，大大降低了数据复制、磁盘 IO 和序列化开销，并且还提供了更多的 API(map/reduec/filter/groupBy...)。

#### 2) RDD 是什么?

RDD(Resilient Distributed Dataset)叫做**弹性分布式数据集**，是 Spark 中最基本的数据抽象，代表一个**不可变、可分区、里面的元素可并行计算的集合**。单词拆解：

- Resilient ：它是弹性的，RDD 里面的中的数据可以保存在**内存中或者磁盘**里面；
- Distributed ：它里面的元素是分布式存储的，可以用于分布式计算；
- Dataset: 它是一个集合，可以存放很多元素。

#### 3) RDD 主要属性

进入 RDD 的源码中看下：

![1639224647235](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/201048-278416.png)

在源码中可以看到有对 RDD 介绍的注释，我们来翻译下：

1. A list of partitions（分区列表） ：一组分片(Partition)/一个分区(Partition)列表，即数据集的基本组成单位。对于 RDD 来说，每个分片都会被一个计算任务处理，**分片数决定并行度**。用户可以在创建 RDD 时指定 RDD 的分片个数，如果没有指定，那么就会采用默认值。
2. A function for computing each split(计算函数) ：一个函数会被作用在每一个分区。Spark 中 RDD 的计算是以分片为单位的，compute 函数会被作用到每个分区上。
3. A list of dependencies on other RDDs （依赖关系）：一个 RDD 会依赖于其他多个 RDD。RDD 的每次转换都会生成一个新的 RDD，所以 RDD 之间就会形成类似于流水线一样的前后依赖关系。在部分分区数据丢失时，Spark 可以通过这个依赖关系重新计算丢失的分区数据，而不是对 RDD 的所有分区进行重新计算。(Spark 的容错机制)
4. Optionally, a Partitioner for key-value RDDs (e.g. to say that the RDD is hash-partitioned)（分区函数）：可选项，对于 KV 类型的 RDD 会有一个 Partitioner，即 RDD 的分区函数，默认为 HashPartitioner。
5. Optionally, a list of preferred locations to compute each split on (e.g. block locations for an HDFS file)（最佳位置）：可选项,一个列表，存储存取每个 Partition 的优先位置(preferred location)。对于一个 HDFS 文件来说，这个列表保存的就是每个 Partition 所在的块的位置。按照"**移动数据不如移动计算**"的理念，Spark 在进行任务调度的时候，会尽可能选择那些存有数据的 worker 节点来进行任务计算。

**总结**

RDD 是一个数据集的表示，不仅表示了数据集，还表示了这个数据集从哪来，如何计算，主要属性包括：

1. 分区列表
2. 计算函数
3. 依赖关系
4. 分区函数(默认是 hash)
5. 最佳位置

> 分区列表、分区函数、最佳位置，这三个属性其实说的就是数据集在哪，在哪计算更合适，如何分区；
>
> 计算函数、依赖关系，这两个属性其实说的是数据集怎么来的。

### 2. RDD-API

#### 1) RDD 的创建方式

1. 由外部存储系统的数据集创建，包括本地的文件系统，还有所有 Hadoop 支持的数据集，比如 HDFS、Cassandra、HBase 等：
   `val rdd1 = sc.textFile("hdfs://node1:8020/wordcount/input/words.txt")`
2. 通过已有的 RDD 经过算子转换生成新的 RDD：
   `val rdd2=rdd1.flatMap(_.split(" "))`
3. 由一个已经存在的 Scala 集合创建：
   `val rdd3 = sc.parallelize(Array(1,2,3,4,5,6,7,8))`或者
   `val rdd4 = sc.makeRDD(List(1,2,3,4,5,6,7,8))`

makeRDD 方法底层调用了 parallelize 方法：

![1639395046262](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193047-916008.png)

#### 2) RDD 的算子分类

RDD 的算子分为两类:

1. Transformation转换操作:**返回一个新的 RDD**
2. Action动作操作:**返回值不是 RDD(无返回值或返回其他的)**，会触发一个新的job进行执行。

> ❣️ 注意:
>
> 1、RDD 不实际存储真正要计算的数据，而是记录了数据的位置在哪里，数据的转换关系(调用了什么方法，传入什么函数)。
>
> 2、RDD 中的所有转换都是**惰性求值/延迟执行**的，也就是说并不会直接计算。只有当发生一个要求返回结果给 Driver 的 Action 动作时，这些转换才会真正运行。
>
> 3、之所以使用惰性求值/延迟执行，是因为这样可以在 Action 时对 RDD 操作形成 DAG 有向无环图进行 Stage 的划分和并行优化，这种设计让 Spark 更加有效率地运行。

#### 3) Transformation 转换算子

|                       转换算子                       |                             含义                             |
| :--------------------------------------------------: | :----------------------------------------------------------: |
|                    **map**(func)                     | 返回一个新的 RDD，该 RDD 由每一个输入元素经过 func 函数转换后组成 |
|                   **filter**(func)                   | 返回一个新的 RDD，该 RDD 由经过 func 函数计算后返回值为 true 的输入元素组成 |
|                  **flatMap**(func)                   | 类似于 map，但是每一个输入元素可以被映射为 0 或多个输出元素(所以 func 应该返回一个序列，而不是单一元素) |
|               **mapPartitions**(func)                | 类似于 map，但独立地在 RDD 的每一个分片上运行，因此在类型为 T 的 RDD 上运行时，func 的函数类型必须是 Iterator[T] => Iterator[U] |
|           **mapPartitionsWithIndex**(func)           | 类似于 mapPartitions，但 func 带有一个整数参数表示分片的索引值，因此在类型为 T 的 RDD 上运行时，func 的函数类型必须是(Int, Interator[T]) => Iterator[U] |
|       sample(withReplacement, fraction, seed)        | 根据 fraction 指定的比例对数据进行采样，可以选择是否使用随机数进行替换，seed 用于指定随机数生成器种子 |
|               **union**(otherDataset)                |         对源 RDD 和参数 RDD 求并集后返回一个新的 RDD         |
|              intersection(otherDataset)              |         对源 RDD 和参数 RDD 求交集后返回一个新的 RDD         |
|              **distinct**([numTasks]))               |             对源 RDD 进行去重后返回一个新的 RDD              |
|              **groupByKey**([numTasks])              |   在一个(K,V)的 RDD 上调用，返回一个(K, Iterator[V])的 RDD   |
|          **reduceByKey**(func, [numTasks])           | 在一个(K,V)的 RDD 上调用，返回一个(K,V)的 RDD，使用指定的 reduce 函数，将相同 key 的值聚合到一起，与 groupByKey 类似，reduce 任务的个数可以通过第二个可选的参数来设置 |
| aggregateByKey(zeroValue)(seqOp, combOp, [numTasks]) | 对 PairRDD 中相同的 Key 值进行聚合操作，在聚合过程中同样使用了一个中立的初始值。和 aggregate 函数类似，aggregateByKey 返回值的类型不需要和 RDD 中 value 的类型一致 |
|        **sortByKey**([ascending], [numTasks])        | 在一个(K,V)的 RDD 上调用，K 必须实现 Ordered 接口，返回一个按照 key 进行排序的(K,V)的 RDD |
|         sortBy(func,[ascending], [numTasks])         |                与 sortByKey 类似，但是更灵活                 |
|          **join**(otherDataset, [numTasks])          | 在类型为(K,V)和(K,W)的 RDD 上调用，返回一个相同 key 对应的所有元素对在一起的(K,(V,W))的 RDD |
|          cogroup(otherDataset, [numTasks])           | 在类型为(K,V)和(K,W)的 RDD 上调用，返回一个(K,(Iterable,Iterable))类型的 RDD |
|               cartesian(otherDataset)                |                           笛卡尔积                           |
|               pipe(command, [envVars])               |                     对 rdd 进行管道操作                      |
|             **coalesce**(numPartitions)              | 减少 RDD 的分区数到指定值。在过滤大量数据之后，可以执行此操作 |
|            **repartition**(numPartitions)            |                       重新给 RDD 分区                        |

#### 4) Action 动作算子

|                动作算子                 |                             含义                             |
| :-------------------------------------: | :----------------------------------------------------------: |
|              reduce(func)               | 通过 func 函数聚集 RDD 中的所有元素，这个功能必须是可交换且可并联的 |
|                collect()                |        在驱动程序中，以数组的形式返回数据集的所有元素        |
|                 count()                 |                     返回 RDD 的元素个数                      |
|                 first()                 |            返回 RDD 的第一个元素(类似于 take(1))             |
|                 take(n)                 |           返回一个由数据集的前 n 个元素组成的数组            |
| takeSample(withReplacement,num, [seed]) | 返回一个数组，该数组由从数据集中随机采样的 num 个元素组成，可以选择是否用随机数替换不足的部分，seed 用于指定随机数生成器种子 |
|       takeOrdered(n, [ordering])        |           返回自然顺序或者自定义顺序的前 n 个元素            |
|        **saveAsTextFile**(path)         | 将数据集的元素以 textfile 的形式保存到 HDFS 文件系统或者其他支持的文件系统，对于每个元素，Spark 将会调用 toString 方法，将它装换为文件中的文本 |
|      **saveAsSequenceFile**(path)       | 将数据集中的元素以 Hadoop sequencefile 的格式保存到指定的目录下，可以使 HDFS 或者其他 Hadoop 支持的文件系统 |
|         saveAsObjectFile(path)          |    将数据集的元素，以 Java 序列化的方式保存到指定的目录下    |
|            **countByKey**()             | 针对(K,V)类型的 RDD，返回一个(K,Int)的 map，表示每一个 key 对应的元素个数 |
|              foreach(func)              |        在数据集的每一个元素上，运行函数 func 进行更新        |
|       **foreachPartition**(func)        |            在数据集的每一个分区上，运行函数 func             |

**统计操作：**

|      算子      |           含义            |
| :------------: | :-----------------------: |
|     count      |           个数            |
|      mean      |           均值            |
|      sum       |           求和            |
|      max       |          最大值           |
|      min       |          最小值           |
|    variance    |           方差            |
| sampleVariance |     从采样中计算方差      |
|     stdev      | 标准差:衡量数据的离散程度 |
|  sampleStdev   |       采样的标准差        |
|     stats      |       查看统计结果        |

#### 4) RDD 算子练习

- **需求**：

给定一个键值对 RDD：

```
val rdd = sc.parallelize(Array(("spark",2),("hadoop",6),("hadoop",4),("spark",6)))
```

key 表示图书名称，value 表示某天图书销量

请计算每个键对应的平均值，也就是计算每种图书的每天平均销量。

最终结果:("spark",4),("hadoop",5)。

- **答案 1**：

```
val rdd = sc.parallelize(Array(("spark",2),("hadoop",6),("hadoop",4),("spark",6)))
val rdd2 = rdd.groupByKey()
rdd2.collect
//Array[(String, Iterable[Int])] = Array((spark,CompactBuffer(2, 6)), (hadoop,CompactBuffer(6, 4)))
rdd2.mapValues(v=>v.sum/v.size).collect
Array[(String, Int)] = Array((spark,4), (hadoop,5))
```

- **答案 2**：

```
val rdd = sc.parallelize(Array(("spark",2),("hadoop",6),("hadoop",4),("spark",6)))
val rdd2 = rdd.groupByKey()
rdd2.collect
//Array[(String, Iterable[Int])] = Array((spark,CompactBuffer(2, 6)), (hadoop,CompactBuffer(6, 4)))

val rdd3 = rdd2.map(t=>(t._1,t._2.sum /t._2.size))
rdd3.collect
//Array[(String, Int)] = Array((spark,4), (hadoop,5))
```

#### 你知道 reduceByKey 和 groupByKey 有啥区别吗？

reduceByKey()会在 shuffle 之前对数据进行合并。有点类似于在 MapReduce 中的 combiner。这样做的好处在于，在转换操作时就已经对数据进行了一次聚合操作，从而减小数据传输。如下图所示：

![1639396592551](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/195638-331860.png)

groupByKey 算子操作发生在动作操作端，即 Shuffle 之后，所以势必会将所有的数据通过网络进行传输，造成不必要的浪费。同时如果数据量十分大，可能还会造成 OutOfMemoryError。如下图所示：

![1639396615691](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/195702-205195.png)

#### 你知道  reduceByKey、foldByKey、aggregateByKey、combineByKey 区别吗？

- reduceByKey **没有初始值** 分区内和分区间逻辑相同

- foldByKey **有初始值** 分区内和分区间逻辑相同

- aggregateByKey 有初始值 **分区内和分区间逻辑可以不同**

- combineByKey **初始值可以变化结构 分区内和分区间逻辑不同**

#### 你刚才提到了 DAG,能说一下什么是 DAG？

DAG(Directed Acyclic Graph 有向无环图)指的是数据转换执行的过程，有方向，无闭环(其实就是 RDD 执行的流程)；

![1639396679128](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/05/153157-562506.png)

原始的 RDD 通过一系列的转换操作就形成了 DAG 有向无环图，任务执行时，可以按照 DAG 的描述，执行真正的计算(数据被操作的一个过程)。

**DAG 的边界**

- 开始:通过 SparkContext 创建的 RDD；
- 结束:触发 Action，一旦触发 Action 就形成了一个完整的 DAG。

#### Spark 广播变量和累加器介绍一下？

在默认情况下，当 Spark 在集群的多个不同节点的多个任务上并行运行一个函数时，它会把函数中涉及到的每个变量，在每个任务上都生成一个副本。但是，有时候需要在多个任务之间共享变量，或者在任务(Task)和任务控制节点(Driver Program)之间共享变量。

为了满足这种需求，Spark 提供了两种类型的变量：

**累加器 accumulators**：累加器支持在所有不同节点之间进行累加计算(比如计数或者求和)。

**广播变量 broadcast variables**：广播变量用来把变量在所有节点的内存之间进行共享，**在每个机器上缓存一个只读的变量**，而不是为机器上的每个任务都生成一个副本。

#### 广播变量和累加器的区别是啥？

![1639396751616](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/195913-716556.png)

#### 如何使用 Spark 实现 TopN 的获取（描述思路）

**方法1**：

1. 按照 key 对数据进行聚合（groupByKey）；
2. 将 value 转换为数组，利用 scala 的 sortBy 或者 sortWith 进行排序（mapValues）数据量太大，会OOM。

**方法2：**

（1）取出所有的 key；

（2）对 key 进行迭代，每次取出一个 key 利用 spark 的排序算子进行排序。

**方法3：**

（1）自定义分区器，按照 key 进行分区，使不同的 key 进到不同的分区；

（2）对每个分区运用 spark 的排序算子进行排序。

### 3. RDD 的持久化/缓存

在实际开发中某些 RDD 的计算或转换可能会比较耗费时间，如果这些 RDD 后续还会频繁的被使用到，那么可以将这些 RDD 进行持久化/缓存，这样下次再使用到的时候就不用再重新计算了，提高了程序运行的效率。

```scala
val rdd1 = sc.textFile("hdfs://node01:8020/words.txt")
val rdd2 = rdd1.flatMap(x=>x.split(" ")).map((_,1)).reduceByKey(_+_)
rdd2.cache //缓存/持久化
rdd2.sortBy(_._2,false).collect//触发action,会去读取HDFS的文件,rdd2会真正执行持久化
rdd2.sortBy(_._2,false).collect//触发action,会去读缓存中的数据,执行速度会比之前快,因为rdd2已经持久化到内存中了
```

#### 持久化/缓存 API 详解

- ersist 方法和 cache 方法

RDD 通过 persist 或 cache 方法可以将前面的计算结果缓存，但是并不是这两个方法被调用时立即缓存，而是触发后面的 action 时，该 RDD 将会被缓存在计算节点的内存中，并供后面重用。
通过查看 RDD 的源码发现 cache 最终也是调用了 persist 无参方法(默认存储只存在内存中)：

![1639395151595](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193233-105196.png)

- 存储级别

默认的存储级别都是仅在内存存储一份，Spark 的存储级别还有好多种，存储级别在 object StorageLevel 中定义的。

|              持久化级别               |                             说明                             |
| :-----------------------------------: | :----------------------------------------------------------: |
|          **MORY_ONLY(默认)**          | 将 RDD 以非序列化的 Java 对象存储在 JVM 中。如果没有足够的内存存储 RDD，则某些分区将不会被缓存，每次需要时都会重新计算。这是默认级别 |
| **MORY_AND_DISK(开发中可以使用这个)** | 将 RDD 以非序列化的 Java 对象存储在 JVM 中。如果数据在内存中放不下，则溢写到磁盘上．需要时则会从磁盘上读取 |
|   MEMORY_ONLY_SER (Java and Scala)    | 将 RDD 以序列化的 Java 对象(每个分区一个字节数组)的方式存储．这通常比非序列化对象(deserialized objects)更具空间效率，特别是在使用快速序列化的情况下，但是这种方式读取数据会消耗更多的 CPU |
| MEMORY_AND_DISK_SER (Java and Scala)  | 与 MEMORY_ONLY_SER 类似，但如果数据在内存中放不下，则溢写到磁盘上，而不是每次需要重新计算它们 |
|               DISK_ONLY               |                   将 RDD 分区存储在磁盘上                    |
|  MEMORY_ONLY_2, MEMORY_AND_DISK_2 等  | 与上面的储存级别相同，只不过将持久化数据存为两份，备份每个分区存储在两个集群节点上 |
|           OFF_HEAP(实验中)            | 与 MEMORY_ONLY_SER 类似，但将数据存储在堆外内存中。(即不是直接存储在 JVM 内存中) |

**总结：**

1. RDD 持久化/缓存的目的是**为了提高后续操作的速度**
2. 缓存的级别有很多，默认只存在内存中,开发中使用 memory_and_disk
3. **只有执行 action 操作的时候才会真正将 RDD 数据进行持久化/缓存**
4. **实际开发中如果某一个 RDD 后续会被频繁的使用，可以将该 RDD 进行持久化/缓存**

### 4. RDD 容错机制 Checkpoint

- **持久化的局限：**

持久化/缓存可以把数据放在内存中，虽然是快速的，但是也是最不可靠的；也可以把数据放在磁盘上，也不是完全可靠的！例如磁盘会损坏等。

- **问题解决：**

**Checkpoint 的产生就是为了更加可靠的数据持久化**，在 Checkpoint 的时候一般把数据放在在 HDFS 上，这就天然的借助了 HDFS 天生的高容错、高可靠来实现数据最大程度上的安全，实现了 RDD 的容错和高可用。

用法：

```scala
SparkContext.setCheckpointDir("目录") //HDFS的目录

RDD.checkpoint
```

- **总结：**
- 开发中如何保证数据的安全性性及读取效率：可以对频繁使用且重要的数据，先做缓存/持久化，再做 checkpint 操作。
- 持久化和 Checkpoint 的区别：

1. 位置：Persist 和 Cache 只能保存在本地的磁盘和内存中(或者堆外内存--实验中) Checkpoint 可以保存数据到 HDFS 这类可靠的存储上。
2. 生命周期：Cache 和 Persist 的 RDD 会在程序结束后会被清除或者手动调用 unpersist 方法 Checkpoint 的 RDD 在程序结束后依然存在，不会被删除,也就是说checkpoint会保存依赖链的关系。

### 5. RDD 依赖关系

#### 1) 宽窄依赖

- 两种依赖关系类型：RDD 和它依赖的父 RDD 的关系有两种不同的类型，即**宽依赖**(wide dependency/shuffle dependency)**窄依赖**(narrow dependency)

![1639395175241](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193255-444468.png)

- 图解：

![1639395202823](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193323-818751.png)

宽窄依赖

- 如何区分宽窄依赖：

> 窄依赖:父 RDD 的一个分区只会被子 RDD 的一个分区依赖；
>
> 宽依赖:父 RDD 的一个分区会被子 RDD 的多个分区依赖(涉及到 shuffle)。

#### 2) 为什么要设计宽窄依赖

**对于窄依赖：**

窄依赖的多个分区可以并行计算；

窄依赖的一个分区的数据如果丢失只需要重新计算对应的分区的数据就可以了。

**对于宽依赖：**

划分 Stage(阶段)的依据:对于宽依赖,必须等到上一阶段计算完成才能计算下一阶段。

### 6. DAG 的生成和划分 Stage

#### 1) DAG 介绍

- DAG 是什么：

DAG(Directed Acyclic Graph 有向无环图)指的是数据转换执行的过程，有方向，无闭环(其实就是 RDD 执行的流程)；

原始的 RDD 通过一系列的转换操作就形成了 DAG 有向无环图，任务执行时，可以按照 DAG 的描述，执行真正的计算(数据被操作的一个过程)。

- DAG 的边界

开始:通过 SparkContext 创建的 RDD；

结束:触发 Action，一旦触发 Action 就形成了一个完整的 DAG,触发一个行动算子，就会生成一个job,然后就会生成一个DAG图。

#### 2) DAG 划分 Stage

![1639395229668](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193351-101743.png)

**一个 Spark 程序可以有多个 DAG(有几个 Action，就有几个 DAG，上图最后只有一个 Action（图中未表现）,那么就是一个 DAG)**。

一个 DAG 可以有多个 Stage(**根据宽依赖/shuffle 进行划分**)。

**同一个 Stage 可以有多个 Task 并行执行**(**task 数=分区数**，如上图，Stage1 中有三个分区 P1、P2、P3，对应的也有三个 Task)。

可以看到这个 DAG 中只 reduceByKey 操作是一个宽依赖，Spark 内核会以此为边界将其前后划分成不同的 Stage。

同时我们可以注意到，在图中 Stage1 中，**从 textFile 到 flatMap 到 map 都是窄依赖，这几步操作可以形成一个流水线操作，通过 flatMap 操作生成的 partition 可以不用等待整个 RDD 计算结束，而是继续进行 map 操作，这样大大提高了计算的效率**。

- 为什么要划分 Stage? --并行计算

一个复杂的业务逻辑如果有 shuffle，那么就意味着前面阶段产生结果后，才能执行下一个阶段，即下一个阶段的计算要依赖上一个阶段的数据。那么我们按照 shuffle 进行划分(也就是按照宽依赖就行划分)，就可以将一个 DAG 划分成多个 Stage/阶段，在同一个 Stage 中，会有多个算子操作，可以形成一个 pipeline 流水线，流水线内的多个平行的分区可以并行执行。

- 如何划分 DAG 的 stage？

对于窄依赖，partition 的转换处理在 stage 中完成计算，不划分(将窄依赖尽量放在在同一个 stage 中，可以实现流水线计算)。

对于宽依赖，由于有 shuffle 的存在，只能在父 RDD 处理完成后，才能开始接下来的计算，也就是说需要要划分 stage。

**总结：**

Spark 会根据 shuffle/宽依赖使用回溯算法来对 DAG 进行 Stage 划分，从后往前，遇到宽依赖就断开，遇到窄依赖就把当前的 RDD 加入到当前的 stage/阶段中

> 具体的划分算法请参见 AMP 实验室发表的论文：《Resilient Distributed Datasets: A Fault-Tolerant Abstraction for In-Memory Cluster Computing》
> `http://xueshu.baidu.com/usercenter/paper/show?paperid=b33564e60f0a7e7a1889a9da10963461&site=xueshu_se`

### 7. RDD 累加器和广播变量

在默认情况下，当 Spark 在集群的多个不同节点的多个任务上并行运行一个函数时，它会把函数中涉及到的每个变量，在每个任务上都生成一个副本。但是，有时候需要在多个任务之间共享变量，或者在任务(Task)和任务控制节点(Driver Program)之间共享变量。

为了满足这种需求，Spark 提供了两种类型的变量：

1. **累加器 accumulators**：累加器支持在所有不同节点之间进行累加计算(比如计数或者求和)。
2. **广播变量 broadcast variables**：广播变量用来把变量在所有节点的内存之间进行共享，在每个机器上缓存一个只读的变量，而不是为机器上的每个任务都生成一个副本。

#### 1) 累加器

**1. 不使用累加器**

```
var counter = 0
val data = Seq(1, 2, 3)
data.foreach(x => counter += x)
println("Counter value: "+ counter)
```

运行结果：

```
Counter value: 6
```

如果我们将 data 转换成 RDD，再来重新计算：

```
var counter = 0
val data = Seq(1, 2, 3)
var rdd = sc.parallelize(data)
rdd.foreach(x => counter += x)
println("Counter value: "+ counter)
```

运行结果：

```
Counter value: 0
```

**2. 使用累加器**

通常在向 Spark 传递函数时，比如使用 map() 函数或者用 filter() 传条件时，可以使用驱动器程序中定义的变量，但是集群中运行的每个任务都会得到这些变量的一份新的副本，更新这些副本的值也不会影响驱动器中的对应变量。这时使用累加器就可以实现我们想要的效果:

**val xx: Accumulator[Int] = sc.accumulator(0)**

**3. 代码示例**：

```scala
import org.apache.spark.rdd.RDD
import org.apache.spark.{Accumulator, SparkConf, SparkContext}

object AccumulatorTest {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf().setAppName("wc").setMaster("local[*]")
    val sc: SparkContext = new SparkContext(conf)
    sc.setLogLevel("WARN")

    //使用scala集合完成累加
    var counter1: Int = 0;
    var data = Seq(1,2,3)
    data.foreach(x => counter1 += x )
    println(counter1)//6

    println("+++++++++++++++++++++++++")

    //使用RDD进行累加
    var counter2: Int = 0;
    val dataRDD: RDD[Int] = sc.parallelize(data) //分布式集合的[1,2,3]
    dataRDD.foreach(x => counter2 += x)
    println(counter2)//0
    //注意：上面的RDD操作运行结果是0
    //因为foreach中的函数是传递给Worker中的Executor执行,用到了counter2变量
    //而counter2变量在Driver端定义的,在传递给Executor的时候,各个Executor都有了一份counter2
    //最后各个Executor将各自个x加到自己的counter2上面了,和Driver端的counter2没有关系

    //那这个问题得解决啊!不能因为使用了Spark连累加都做不了了啊!
    //如果解决?---使用累加器
    val counter3: Accumulator[Int] = sc.accumulator(0)
    dataRDD.foreach(x => counter3 += x)
    println(counter3)//6
  }
}
```

#### 2) 广播变量

**1. 不使用广播变量**

**2. 使用广播变量**

**3. 代码示例**：

关键词：**sc.broadcast()**

```scala
import org.apache.spark.broadcast.Broadcast
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object BroadcastVariablesTest {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf().setAppName("wc").setMaster("local[*]")
    val sc: SparkContext = new SparkContext(conf)
    sc.setLogLevel("WARN")

    //不使用广播变量
    val kvFruit: RDD[(Int, String)] = sc.parallelize(List((1,"apple"),(2,"orange"),(3,"banana"),(4,"grape")))
    val fruitMap: collection.Map[Int, String] =kvFruit.collectAsMap
    //scala.collection.Map[Int,String] = Map(2 -> orange, 4 -> grape, 1 -> apple, 3 -> banana)
    val fruitIds: RDD[Int] = sc.parallelize(List(2,4,1,3))
    //根据水果编号取水果名称
    val fruitNames: RDD[String] = fruitIds.map(x=>fruitMap(x))
    fruitNames.foreach(println)
    //注意:以上代码看似一点问题没有,但是考虑到数据量如果较大,且Task数较多,
    //那么会导致,被各个Task共用到的fruitMap会被多次传输
    //应该要减少fruitMap的传输,一台机器上一个,被该台机器中的Task共用即可
    //如何做到?---使用广播变量
    //注意:广播变量的值不能被修改,如需修改可以将数据存到外部数据源,如MySQL、Redis
    println("=====================")
    val BroadcastFruitMap: Broadcast[collection.Map[Int, String]] = sc.broadcast(fruitMap)
    val fruitNames2: RDD[String] = fruitIds.map(x=>BroadcastFruitMap.value(x))
    fruitNames2.foreach(println)

  }
}
```

## 三、Spark SQL

### 1. 数据分析方式

#### 1) 命令式

在前面的 RDD 部分, 非常明显可以感觉的到是命令式的, 主要特征是通过一个算子, 可以得到一个结果, 通过结果再进行后续计算。

```
sc.textFile("...")
  .flatMap(_.split(" "))
  .map((_, 1))
  .reduceByKey(_ + _)
  .collect()
```

1. 命令式的优点

- 操作粒度更细，能够控制数据的每一个处理环节；
- 操作更明确，步骤更清晰，容易维护；
- 支持半/非结构化数据的操作。

1. 命令式的缺点

- 需要一定的代码功底；
- 写起来比较麻烦。

#### 2) SQL

对于一些数据科学家/数据库管理员/DBA, 要求他们为了做一个非常简单的查询, 写一大堆代码, 明显是一件非常残忍的事情, 所以 SQL on Hadoop 是一个非常重要的方向。

```
SELECT
   name,
   age,
   school
FROM students
WHERE age > 10
```

1. SQL 的优点

表达非常清晰, 比如说这段 SQL 明显就是为了查询三个字段，条件是查询年龄大于 10 岁的。

1. SQL 的缺点

- 试想一下 3 层嵌套的 SQL 维护起来应该挺力不从心的吧；
- 试想一下如果使用 SQL 来实现机器学习算法也挺为难的吧。

#### 3) 总结

SQL 擅长数据分析和通过简单的语法表示查询，命令式操作适合过程式处理和算法性的处理。

在 Spark 出现之前，对于结构化数据的查询和处理， 一个工具一向只能支持 SQL 或者命令式，使用者被迫要使用多个工具来适应两种场景，并且多个工具配合起来比较费劲。

而 Spark 出现了以后，统一了两种数据处理范式是一种革新性的进步。

### 2. SparkSQL 前世今生

SQL 是数据分析领域一个非常重要的范式，所以 Spark 一直想要支持这种范式，而伴随着一些决策失误，这个过程其实还是非常曲折的。

#### 1) 发展历史

![1639395266238](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193427-320687.png)

- **Hive**

解决的问题:

Hive 实现了 SQL on Hadoop，使用 MapReduce 执行任务 简化了 MapReduce 任务。

新的问题:

Hive 的查询延迟比较高，原因是使用 MapReduce 做计算。

- **Shark**

解决的问题：

Shark 改写 Hive 的物理执行计划， 使用 Spark 代替 MapReduce 物理引擎 使用列式内存存储。以上两点使得 Shark 的查询效率很高。

新的问题：

Shark 执行计划的生成严重依赖 Hive，想要增加新的优化非常困难；

Hive 是进程级别的并行，Spark 是线程级别的并行，所以 Hive 中很多线程不安全的代码不适用于 Spark；

由于以上问题，Shark 维护了 Hive 的一个分支，并且无法合并进主线，难以为继；

在 2014 年 7 月 1 日的 Spark Summit 上，Databricks 宣布终止对 Shark 的开发，将重点放到 Spark SQL 上。

- **SparkSQL-DataFrame**

解决的问题：

Spark SQL 执行计划和优化交给优化器 Catalyst；

内建了一套简单的 SQL 解析器，可以不使用 HQL；

还引入和 DataFrame 这样的 DSL API，完全可以不依赖任何 Hive 的组件

新的问题：

对于初期版本的 SparkSQL，依然有挺多问题，例如只能支持 SQL 的使用，不能很好的兼容命令式，入口不够统一等。

- **SparkSQL-Dataset**

SparkSQL 在 1.6 时代，增加了一个新的 API，叫做 Dataset，Dataset 统一和结合了 SQL 的访问和命令式 API 的使用，这是一个划时代的进步。

在 Dataset 中可以轻易的做到使用 SQL 查询并且筛选数据，然后使用命令式 API 进行探索式分析。

### 3. Hive 和 SparkSQL

Hive 是将 SQL 转为 MapReduce。

SparkSQL 可以理解成是将 SQL 解析成：“RDD + 优化” 再执行。

![1639395287867](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193448-735836.png)

### 4. 数据分类和 SparkSQL 适用场景

#### 1) 结构化数据

一般指数据有固定的 **Schema**(约束)，例如在用户表中，name 字段是 String 型，那么每一条数据的 name 字段值都可以当作 String 来使用：

|  id  |   name   |            url            | alexa | country |
| :--: | :------: | :-----------------------: | :---: | :-----: |
|  1   |  Google  |  https://www.google.cm/   |   1   |   USA   |
|  2   |   淘宝   |  https://www.taobao.com/  |  13   |   CN    |
|  3   | 菜鸟教程 |  https://www.runoob.com/  | 4689  |   CN    |
|  4   |   微博   |     http://weibo.com/     |  20   |   CN    |
|  5   | Facebook | https://www.facebook.com/ |   3   |   USA   |

#### 2) 半结构化数据

般指的是数据没有固定的 Schema，但是数据本身是有结构的。

- 没有固定 Schema

指的是半结构化数据是没有固定的 Schema 的，可以理解为没有显式指定 Schema。

比如说一个用户信息的 JSON 文件，
第 1 条数据的 phone_num 有可能是数字，
第 2 条数据的 phone_num 虽说应该也是数字，但是如果指定为 String，也是可以的，
因为没有指定 Schema，没有显式的强制的约束。

- 有结构

虽说半结构化数据是没有显式指定 Schema 的，也没有约束，但是半结构化数据本身是有有隐式的结构的，也就是数据自身可以描述自身。

例如 JSON 文件，其中的某一条数据是有字段这个概念的，每个字段也有类型的概念，所以说 JSON 是可以描述自身的，也就是数据本身携带有元信息。

#### 3) 总结

- **数据分类总结**：

|              |             定义              |                       特点                        |               举例                |
| :----------: | :---------------------------: | :-----------------------------------------------: | :-------------------------------: |
|  结构化数据  |        有固定的 Schema        |                 有预定义的 Schema                 |         关系型数据库的表          |
| 半结构化数据 | 没有固定的 Schema，但是有结构 | 没有固定的 Schema，有结构信息，数据一般是自描述的 | 指一些有结构的文件格式，例如 JSON |
| 非结构化数据 |  没有固定 Schema，也没有结构  |            没有固定 Schema，也没有结构            |       指图片/音频之类的格式       |

- **Spark 处理什么样的数据**？

RDD 主要用于处理非结构化数据 、半结构化数据、结构化；

SparkSQL 主要用于处理结构化数据(较为规范的半结构化数据也可以处理)。

- **总结**：

SparkSQL 是一个既支持 SQL 又支持命令式数据处理的工具；

SparkSQL 的主要适用场景是处理结构化数据(较为规范的半结构化数据也可以处理)。

### 5. Spark SQL 数据抽象

#### 1) DataFrame

- **什么是 DataFrame**

DataFrame 的前身是 SchemaRDD，从 Spark 1.3.0 开始 SchemaRDD 更名为 DataFrame。并不再直接继承自 RDD，而是自己实现了 RDD 的绝大多数功能。

DataFrame 是一种以 RDD 为基础的分布式数据集，类似于传统数据库的二维表格，带有 Schema 元信息(可以理解为数据库的列名和类型)。

- **总结**:

**DataFrame 就是一个分布式的表**；

**DataFrame = RDD - 泛型 + SQL 的操作 + 优化**。

#### 2) DataSet

- **DataSet**：

DataSet 是在 Spark1.6 中添加的新的接口。

与 RDD 相比，保存了更多的描述信息，概念上等同于关系型数据库中的二维表。

与 DataFrame 相比，保存了类型信息，是强类型的，提供了编译时类型检查。

调用 Dataset 的方法先会生成逻辑计划，然后被 spark 的优化器进行优化，最终生成物理计划，然后提交到集群中运行！

DataSet 包含了 DataFrame 的功能。

**Spark2.0 中两者统一，DataFrame 表示为 DataSet[Row]，即 DataSet 的子集。**

**DataFrame 其实就是 Dateset[Row]**：

![1639395309145](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193510-164009.png)

#### 3) RDD、DataFrame、DataSet 的区别

1. **结构图解**：

![1639395324557](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193525-302816.png)

- RDD[Person]：

  以 Person 为类型参数，但不了解 其内部结构。

- DataFrame：

  提供了详细的结构信息 schema 列的名称和类型。这样看起来就像一张表了。

- DataSet[Person]

  不光有 schema 信息，还有类型信息。

1. **数据图解**：

- 假设 RDD 中的两行数据长这样：

  ```
   RDD[Person]：
  ```

  ![1639395356414](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193557-945543.png)

- 那么 DataFrame 中的数据长这样：

  DataFrame = RDD[Person] - 泛型 + Schema + SQL 操作 + 优化：

![1639395342410](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193543-356595.png)

- 那么 Dataset 中的数据长这样：

  Dataset[Person] = DataFrame + 泛型：

![1639395369497](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193610-334304.png)

- Dataset 也可能长这样:Dataset[Row]：

  即 DataFrame = DataSet[Row]：

![1639395384325](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193625-282855.png)

#### 4) 总结

**DataFrame = RDD - 泛型 + Schema + SQL + 优化**

**DataSet = DataFrame + 泛型**

**DataSet = RDD + Schema + SQL + 优化**

### 6. Spark SQL 应用

- 在 spark2.0 版本之前

  SQLContext 是创建 DataFrame 和执行 SQL 的入口。

  HiveContext 通过 hive sql 语句操作 hive 表数据，兼容 hive 操作，hiveContext 继承自 SQLContext。

- 在 spark2.0 之后

  这些都统一于 SparkSession，SparkSession 封装了 SqlContext 及 HiveContext；

  实现了 SQLContext 及 HiveContext 所有功能；

  通过 SparkSession 还可以获取到 SparkConetxt。

#### 1) 创建 DataFrame/DataSet

- **读取文本文件**：

1. 在本地创建一个文件，有 id、name、age 三列，用空格分隔，然后上传到 hdfs 上。

vim /root/person.txt

```
1 zhangsan 20
2 lisi 29
3 wangwu 25
4 zhaoliu 30
5 tianqi 35
6 kobe 40
```

1. 打开 spark-shell

```
spark/bin/spark-shell
```

创建 RDD

```
val lineRDD= sc.textFile("hdfs://node1:8020/person.txt").map(_.split(" ")) //RDD[Array[String]]
```

1. 定义 case class(相当于表的 schema)

```
case class Person(id:Int, name:String, age:Int)
```

1. 将 RDD 和 case class 关联 val personRDD = lineRDD.map(x => Person(x(0).toInt, x(1), x(2).toInt)) //RDD[Person]
2. 将 RDD 转换成 DataFrame

```
val personDF = personRDD.toDF //DataFrame
```

1. 查看数据和 schema

personDF.show

```
+---+--------+---+
| id|    name|age|
+---+--------+---+
|  1|zhangsan| 20|
|  2|    lisi| 29|
|  3|  wangwu| 25|
|  4| zhaoliu| 30|
|  5|  tianqi| 35|
|  6|    kobe| 40|
+---+--------+---+
```

personDF.printSchema

1. 注册表

```
personDF.createOrReplaceTempView("t_person")
```

1. 执行 SQL

```
spark.sql("select id,name from t_person where id > 3").show
```

1. 也可以通过 SparkSession 构建 DataFrame

```
val dataFrame=spark.read.text("hdfs://node1:8020/person.txt")
dataFrame.show //注意：直接读取的文本文件没有完整schema信息
dataFrame.printSchema
```

- **读取 json 文件**:

```
val jsonDF= spark.read.json("file:///resources/people.json")
```

接下来就可以使用 DataFrame 的函数操作

```
jsonDF.show
```

> 注意：直接读取 json 文件有 schema 信息，因为 json 文件本身含有 Schema 信息，SparkSQL 可以自动解析。

- **读取 parquet 文件**：

```
val parquetDF=spark.read.parquet("file:///resources/users.parquet")
```

接下来就可以使用 DataFrame 的函数操作

```
parquetDF.show
```

> 注意：直接读取 parquet 文件有 schema 信息，因为 parquet 文件中保存了列的信息。

#### 2) 两种查询风格：DSL 和 SQL

- 准备工作：

先读取文件并转换为 DataFrame 或 DataSet：

```
val lineRDD= sc.textFile("hdfs://node1:8020/person.txt").map(_.split(" "))
case class Person(id:Int, name:String, age:Int)
val personRDD = lineRDD.map(x => Person(x(0).toInt, x(1), x(2).toInt))
val personDF = personRDD.toDF
personDF.show
//val personDS = personRDD.toDS
//personDS.show
```

- **DSL 风格**:

SparkSQL 提供了一个领域特定语言(DSL)以方便操作结构化数据

1. 查看 name 字段的数据

```
personDF.select(personDF.col("name")).show
personDF.select(personDF("name")).show
personDF.select(col("name")).show
personDF.select("name").show
```

1. 查看 name 和 age 字段数据

```
personDF.select("name", "age").show
```

1. 查询所有的 name 和 age，并将 age+1

```
personDF.select(personDF.col("name"), personDF.col("age") + 1).show
personDF.select(personDF("name"), personDF("age") + 1).show
personDF.select(col("name"), col("age") + 1).show
personDF.select("name","age").show
//personDF.select("name", "age"+1).show
personDF.select($"name",$"age",$"age"+1).show
```

1. 过滤 age 大于等于 25 的，使用 filter 方法过滤

```
personDF.filter(col("age") >= 25).show
personDF.filter($"age" >25).show
```

1. 统计年龄大于 30 的人数

```
personDF.filter(col("age")>30).count()
personDF.filter($"age" >30).count()
```

1. 按年龄进行分组并统计相同年龄的人数

```
personDF.groupBy("age").count().show
```

- **SQL 风格**:

DataFrame 的一个强大之处就是我们可以将它看作是一个关系型数据表，然后可以通过在程序中使用 spark.sql() 来执行 SQL 查询，结果将作为一个 DataFrame 返回。

如果想使用 SQL 风格的语法，需要将 DataFrame 注册成表,采用如下的方式：

```
personDF.createOrReplaceTempView("t_person")
spark.sql("select * from t_person").show
```

1. 显示表的描述信息

```
spark.sql("desc t_person").show
```

1. 查询年龄最大的前两名

```
spark.sql("select * from t_person order by age desc limit 2").show
```

1. 查询年龄大于 30 的人的信息

```
spark.sql("select * from t_person where age > 30 ").show
```

1. 使用 SQL 风格完成 DSL 中的需求

```
spark.sql("select name, age + 1 from t_person").show
spark.sql("select name, age from t_person where age > 25").show
spark.sql("select count(age) from t_person where age > 30").show
spark.sql("select age, count(age) from t_person group by age").show
```

- **总结**：

1. **DataFrame 和 DataSet 都可以通过 RDD 来进行创建**；
2. **也可以通过读取普通文本创建--注意:直接读取没有完整的约束,需要通过 RDD+Schema**；
3. **通过 josn/parquet 会有完整的约束**；
4. **不管是 DataFrame 还是 DataSet 都可以注册成表，之后就可以使用 SQL 进行查询了! 也可以使用 DSL**!

#### 3) Spark SQL 完成 WordCount

- **SQL 风格**：

```
import org.apache.spark.SparkContext
import org.apache.spark.sql.{DataFrame, Dataset, SparkSession}


object WordCount {
  def main(args: Array[String]): Unit = {
    //1.创建SparkSession
    val spark: SparkSession = SparkSession.builder().master("local[*]").appName("SparkSQL").getOrCreate()
    val sc: SparkContext = spark.sparkContext
    sc.setLogLevel("WARN")
    //2.读取文件
    val fileDF: DataFrame = spark.read.text("D:\\data\\words.txt")
    val fileDS: Dataset[String] = spark.read.textFile("D:\\data\\words.txt")
    //fileDF.show()
    //fileDS.show()
    //3.对每一行按照空格进行切分并压平
    //fileDF.flatMap(_.split(" ")) //注意:错误,因为DF没有泛型,不知道_是String
    import spark.implicits._
    val wordDS: Dataset[String] = fileDS.flatMap(_.split(" "))//注意:正确,因为DS有泛型,知道_是String
    //wordDS.show()
    /*
    +-----+
    |value|
    +-----+
    |hello|
    |   me|
    |hello|
    |  you|
      ...
     */
    //4.对上面的数据进行WordCount
    wordDS.createOrReplaceTempView("t_word")
    val sql =
      """
        |select value ,count(value) as count
        |from t_word
        |group by value
        |order by count desc
      """.stripMargin
    spark.sql(sql).show()

    sc.stop()
    spark.stop()
  }
}
```

- **DSL 风格**：

```
import org.apache.spark.SparkContext
import org.apache.spark.sql.{DataFrame, Dataset, SparkSession}


object WordCount2 {
  def main(args: Array[String]): Unit = {
    //1.创建SparkSession
    val spark: SparkSession = SparkSession.builder().master("local[*]").appName("SparkSQL").getOrCreate()
    val sc: SparkContext = spark.sparkContext
    sc.setLogLevel("WARN")
    //2.读取文件
    val fileDF: DataFrame = spark.read.text("D:\\data\\words.txt")
    val fileDS: Dataset[String] = spark.read.textFile("D:\\data\\words.txt")
    //fileDF.show()
    //fileDS.show()
    //3.对每一行按照空格进行切分并压平
    //fileDF.flatMap(_.split(" ")) //注意:错误,因为DF没有泛型,不知道_是String
    import spark.implicits._
    val wordDS: Dataset[String] = fileDS.flatMap(_.split(" "))//注意:正确,因为DS有泛型,知道_是String
    //wordDS.show()
    /*
    +-----+
    |value|
    +-----+
    |hello|
    |   me|
    |hello|
    |  you|
      ...
     */
    //4.对上面的数据进行WordCount
    wordDS.groupBy("value").count().orderBy($"count".desc).show()

    sc.stop()
    spark.stop()
  }
}
```

#### 4) Spark SQL 多数据源交互

- **读数据**：

读取 json 文件：

```
spark.read.json("D:\\data\\output\\json").show()
```

读取 csv 文件：

```
spark.read.csv("D:\\data\\output\\csv").toDF("id","name","age").show()
```

读取 parquet 文件：

```
spark.read.parquet("D:\\data\\output\\parquet").show()
```

读取 mysql 表：

```
val prop = new Properties()
    prop.setProperty("user","root")
    prop.setProperty("password","root")
spark.read.jdbc(
"jdbc:mysql://localhost:3306/bigdata?characterEncoding=UTF-8","person",prop).show()
```

- **写数据**：

写入 json 文件：

```
personDF.write.json("D:\\data\\output\\json")
```

写入 csv 文件：

```
personDF.write.csv("D:\\data\\output\\csv")
```

写入 parquet 文件：

```
personDF.write.parquet("D:\\data\\output\\parquet")
```

写入 mysql 表：

```
val prop = new Properties()
    prop.setProperty("user","root")
    prop.setProperty("password","root")
personDF.write.mode(SaveMode.Overwrite).jdbc(
"jdbc:mysql://localhost:3306/bigdata?characterEncoding=UTF-8","person",prop)
```

## 四、Spark Streaming

Spark Streaming 是一个基于 Spark Core 之上的**实时计算框架**，可以从很多数据源消费数据并对数据进行实时的处理，具有高吞吐量和容错能力强等特点。

![1639395424974](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193706-331858.png)

**Spark Streaming 的特点**：

1. 易用

可以像编写离线批处理一样去编写流式程序，支持 java/scala/python 语言。

1. 容错

SparkStreaming 在没有额外代码和配置的情况下可以恢复丢失的工作。

1. 易整合到 Spark 体系

流式处理与批处理和交互式查询相结合。

### 1. 整体流程

Spark Streaming 中，会有一个接收器组件 Receiver，作为一个长期运行的 task 跑在一个 Executor 上。Receiver 接收外部的数据流形成 input DStream。

DStream 会被按照时间间隔划分成一批一批的 RDD，当批处理间隔缩短到秒级时，便可以用于处理实时数据流。时间间隔的大小可以由参数指定，一般设在 500 毫秒到几秒之间。

对 DStream 进行操作就是对 RDD 进行操作，计算处理的结果可以传给外部系统。

Spark Streaming 的工作流程像下面的图所示一样，接受到实时数据后，给数据分批次，然后传给 Spark Engine 处理最后生成该批次的结果。

### spark Streaming是如何实现的，整体流程？

**整体流程**

Spark Streaming 中，会有一个接收器组件 Receiver，作为一个长期运行的 task 跑在一个 Executor 上。Receiver 接收外部的数据流形成 input DStream。

DStream 会被按照时间间隔划分成一批一批的 RDD，当批处理间隔缩短到秒级时，便可以用于处理实时数据流。时间间隔的大小可以由参数指定，一般设在 500 毫秒到几秒之间。

对 DStream 进行操作就是对 RDD 进行操作，计算处理的结果可以传给外部系统。

Spark Streaming 的工作流程像下面的图所示一样，接受到实时数据后，给数据分批次，然后传给 Spark Engine 处理最后生成该批次的结果。

> Spark Streaming在内部处理的机制原理是：先接受实时流的数据，并根据一定的时间间隔拆分成一批批的数据，这些批数据在Spark内核对应一个RDD实例，因此，流数据的DStream可以看成一组RDDs，然后通过调用Spark核心的作业处理这些批数据，最终得到处理后的一批批结果数据。 

### 如何从kafka中获取数据（spark streaming获取数据有哪两种方式，有什么特点）

#### 基于receiver模式

使用 kafka 高层次 Comsumer API 来实现的。receiver 从 kafka 获取的数据都是存储在 executor 内存的，然后 spark streaming 启动的 job 会去处理这些数据。用这种方式来与 Kafka 集成，配置中设置了     `enable.auto.commit`为 true，表明自己不需要维护 offset，而是由 Kafka 自己来维护（在 Kafka 0.10 后，默认的 offset 存储位置改为了 Kafka，实际上就是 Kafka 的一个 topic），Kafka 消费者会周期性地（默认为 5s）去修改偏移量。

这种方式接收的数据都保存在 Receiver 中，一旦出现意外，数据就有可能丢失，要想避免丢失的情况，就必须采用 WAL（Write Ahead Log，预写日志）机制，在数据写入内存前先进行持久化。    

现在我们来试想一种情况，数据从 Kafka 取出后，进行了 WAL，在这个时候，Driver 与 Executor 因为某种原因宕机，这时最新偏移量还没来得及提交，那么在 Driver 恢复后，会从记录的偏移量继续消费数据并处理 WAL 的数据，这样一来，被 WAL 持久化的数据就会被重复计算一次。因此，开启了 WAL 后，这样的容错机制最多只能实现“至少一次”的消息送达语义。而且开启 WAL 后，增加了 I/O 开销，降低了 Spark Streaming 的吞吐量，还会产生冗余存储。    

> 也就是基于receiver模式可能会重复消费数据。

#### Direct模式

spark 1.3 引入的，从而能够确保更加健壮的机制。这种接收数据的方式会**周期性地查询 kafka**，来获取每个 topic+partition 最新的 offset，从而定义每个 batch 的 offset 的范围。

当处理数据的 job 启动时，就会使用 kafka 简单 Comsumer API 来获取 kafka 指定 offset 范围的数据。利用 Spark 本身的 DAG 容错机制，使所有计算失败的数据均可溯源，从而实现了“恰好一次”的消息送达语义。    

#### 两种方式的对比

- 基于 receiver 的方式，是使用 Kafka 的高阶 API 来在 ZooKeeper 中保存消费过的 offset 的。这是消费 Kafka 数据的传统方式。这种方式配合着 WAL 机制可以保证数据零丢失的高可靠性，但是却无法保证数据被处理一次且仅一次，可能会处理两次。因为 Spark 和 ZooKeeper 之间可能是不同步的。
- 基于 direct 的方式，使用 kafka 的简单 api，Spark Streaming 自己就负责追踪消费的 offset，并保存在 checkpoint 中。Spark 自己一定是同步的，因此可以保证数据是消费一次且仅消费一次。

> 最重要一点就是，基于receiver方式，是由kafka来维护每一个partition的offset位置的。
>
> 而基于direct模式，是由spark自己维护offset的位置。
>
> 在生产中，基本上是使用direct模式的。

### Spark Streaming 如何执行流式计算的?

Spark Streaming 中的流式计算其实并不是真正的流计算，而是微批计算。Spark Streaming 的 RDD 实际是一组小批次的 RDD 集合，是微批（Micro-Batch）的模型，以批为核心。

Spark Streaming 在流计算实际上是分解成一段一段较小的批处理数据（Discretized Stream），其中批处理引擎使用 Spark Core，每一段数据都会被转换成弹性分布式数据集 RDD，然后 Spark Streaming 将对 DStream 的转换操作变为 Spark 对 RDD 的转换操作，并将转换的中间结果存入内存中，整个流式计算依据业务的需要可以对中间数据进行叠加。

### 使用 Spark Streaming 写一个 WordCount?

![1639396909006](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/200204-679298.png)

### 使用 Spark Streaming 常用算子有哪些?

Spark Streaming 算子和 Spark Core算子类似，也是分为 Transformations(转换操作)和 Action（动作操作）。

**Transformations 操作常用算子如下：**

map、flatMap、filter、union、reduceByKey、join、transform，这是算子都是无状态转换，当前批次的处理不需要使用之前批次的数据或中间结果。

**action 操作常用算子如下：**

print、saveAsTextFile、saveAsObjectFiles、saveAsHadoopFiles等。

### 使用 Spark Streaming 有状态装换的算子有哪些?

有状态装换的算子包含：

1. 基于追踪状态变化的转换（updateStateByKey）

updateStateByKey：将历史结果应用到当前批次。

1. 滑动窗口的转换 Window Operations

### Spark Streaming 如何实现精确一次消费?

**概念：**

- 精确一次消费（Exactly-once） 是指消息一定会被处理且只会被处理一次。不多不少就一次处理。
- 至少一次消费（at least once），主要是保证数据不会丢失，但有可能存在数据重复问题。
- 最多一次消费 （at most once），主要是保证数据不会重复，但有可能存在数据丢失问题。

如果同时解决了数据丢失和数据重复的问题，那么就实现了**精确一次消费**的语义了。

**解决方案：**

方案一：利用`关系型数据库的事务`进行处理

出现丢失或者重复的问题，核心就是偏移量的提交与数据的保存，不是原子性的。如果能做成要么数据保存和偏移量都成功，要么两个失败。那么就不会出现丢失或者重复了。

这样的话可以把存数据和偏移量放到一个事务里。这样就做到前面的成功，如果后面做失败了，就回滚前面那么就达成了原子性。

方案二：`手动提交偏移量+幂等性处理`

首先解决**数据丢失问题**，办法就是要等数据保存成功后再提交偏移量，所以就必须**手工来控制偏移量**的提交时机。

但是如果数据保存了，没等偏移量提交进程挂了，数据会被重复消费。怎么办？那就要把数据的保存做成幂等性保存。即同一批数据反复保存多次，数据不会翻倍，保存一次和保存一百次的效果是一样的。如果能做到这个，就达到了幂等性保存，就不用担心数据会重复了。

**难点**

话虽如此，在实际的开发中手动提交偏移量其实不难，难的是幂等性的保存，有的时候并不一定能保证。所以有的时候只能优先保证的数据不丢失。数据重复难以避免。即只保证了至少一次消费的语义。

### Spark Streaming 背压机制了解吗？

**问题：**

在默认情况下，Spark Streaming 通过 receivers (或者是 Direct 方式) 以生产者生产数据的速率接收数据。当 batch processing time > batch interval 的时候，也就是每个批次数据处理的时间要比 Spark Streaming 批处理间隔时间长；越来越多的数据被接收，但是数据的处理速度没有跟上，导致系统开始出现数据堆积，可能进一步导致 Executor 端出现 OOM 问题而出现失败的情况。

**解决办法：**

设置 spark.streaming.backpressure.enabled：ture；开启背压机制后 Spark Streaming 会根据延迟动态去 kafka 消费数据；

上限由 spark.streaming.kafka.maxRatePerPartition 参数控制，所以两个参数一般会一起使用。

### SparkStreaming 有哪几种方式消费 Kafka 中的数据，它们之间的区别是什么？

**一、基于 Receiver 的方式**

使用 Receiver 来获取数据。Receiver 是使用 Kafka 的高层次 Consumer API 来实现的。receiver 从 Kafka 中获取的数据都是存储在 Spark Executor 的内存中的（如果突然数据暴增，大量 batch 堆积，很容易出现内存溢出的问题），然后 Spark Streaming 启动的 job 会去处理那些数据。然而，在默认的配置下，这种方式可能会因为底层的失败而丢失数据。**如果要启用高可靠机制，让数据零丢失，就必须启用 Spark Streaming 的预写日志机制**（Write Ahead Log，WAL）。该机制会同步地将接收到的 Kafka 数据写入分布式文件系统（比如HDFS）上的预写日志中。所以，即使底层节点出现了失败，也可以使用预写日志中的数据进行恢复。

**二、基于 Direct 的方式**

这种新的不基于 Receiver 的直接方式，是在 Spark 1.3 中引入的，从而能够确保更加健壮的机制。替代掉使用 Receiver 来接收数据后，这种方式会周期性地查询 Kafka，来获得每个 topic+partition 的最新的 offset，从而定义每个 batch 的 offset 的范围。当处理数据的 job 启动时，就会使用 Kafka的简单 consumer api 来获取 Kafka 指定 offset 范围的数据。

**优点如下**：

- 简化并行读取：如果要读取多个 partition，不需要创建多个输入 DStream 然后对它们进行 union 操作。Spark 会创建跟 Kafka partition 一样多的 RDD partition，并且会并行从 Kafka 中读取数据。所以在 Kafka partition 和 RDD partition 之间，有一个一对一的映射关系。
- 高性能：如果要保证零数据丢失，在基于 receiver 的方式中，需要开启 WAL机制。这种方式其实效率低下，因为数据实际上被复制了两份，Kafka 自己本身就有高可靠的机制，会对数据复制一份，而这里又会复制一份到 WAL 中。而基于 direct 的方式，不依赖 Receiver，不需要开启 WAL 机制，只要 Kafka 中作了数据的复制，那么就可以通过 Kafka 的副本进行恢复。次且仅一次的事务机制。

**三、对比：**

- 基于 receiver 的方式，是使用 Kafka 的高阶 API 来在 ZooKeeper 中保存消费过的 offset 的。这是消费 Kafka 数据的传统方式。这种方式配合着 WAL 机制可以保证数据零丢失的高可靠性，但是却无法保证数据被处理一次且仅一次，可能会处理两次。因为 Spark 和 ZooKeeper 之间可能是不同步的。
- 基于 direct 的方式，使用 kafka 的简单 api，Spark Streaming 自己就负责追踪消费的 offset，并保存在 checkpoint 中。Spark 自己一定是同步的，因此可以保证数据是消费一次且仅消费一次。

**在实际生产环境中大都用 Direct 方式**

### 2. 数据抽象

Spark Streaming 的基础抽象是 DStream(Discretized Stream，离散化数据流，连续不断的数据流)，代表持续性的数据流和经过各种 Spark 算子操作后的结果数据流。

可以从以下多个角度深入理解 DStream：

1. DStream 本质上就是一系列时间上连续的 RDD

![1639395440127](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193721-303961.png)

1. 对 DStream 的数据的进行操作也是按照 RDD 为单位来进行的

![1639395454798](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193735-520754.png)

1. 容错性，底层 RDD 之间存在依赖关系，DStream 直接也有依赖关系，RDD 具有容错性，那么 DStream 也具有容错性
2. 准实时性/近实时性

Spark Streaming 将流式计算分解成多个 Spark Job，对于每一时间段数据的处理都会经过 Spark DAG 图分解以及 Spark 的任务集的调度过程。

对于目前版本的 Spark Streaming 而言，其最小的 Batch Size 的选取在 0.5~5 秒钟之间。

**所以 Spark Streaming 能够满足流式准实时计算场景，对实时性要求非常高的如高频实时交易场景则不太适合**。

- 总结

简单来说 DStream 就是对 RDD 的封装，你对 DStream 进行操作，就是对 RDD 进行操作。

对于 DataFrame/DataSet/DStream 来说本质上都可以理解成 RDD。

### 3. DStream 相关操作

DStream 上的操作与 RDD 的类似，分为以下两种：

1. Transformations(转换)
2. Output Operations(输出)/Action

#### 1) Transformations

以下是常见 Transformation---都是无状态转换：即每个批次的处理不依赖于之前批次的数据：

|        Transformation         |                             含义                             |
| :---------------------------: | :----------------------------------------------------------: |
|           map(func)           | 对 DStream 中的各个元素进行 func 函数操作，然后返回一个新的 DStream |
|         flatMap(func)         | 与 map 方法类似，只不过各个输入项可以被输出为零个或多个输出项 |
|         filter(func)          | 过滤出所有函数 func 返回值为 true 的 DStream 元素并返回一个新的 DStream |
|      union(otherStream)       | 将源 DStream 和输入参数为 otherDStream 的元素合并，并返回一个新的 DStream |
| reduceByKey(func, [numTasks]) | 利用 func 函数对源 DStream 中的 key 进行聚合操作，然后返回新的(K，V)对构成的 DStream |
| join(otherStream, [numTasks]) | 输入为(K,V)、(K,W)类型的 DStream，返回一个新的(K，(V，W)类型的 DStream |
|      **transform(func)**      | 通过 RDD-to-RDD 函数作用于 DStream 中的各个 RDD，可以是任意的 RDD 操作，从而返回一个新的 RDD |

除此之外还有一类特殊的 Transformations---有状态转换：当前批次的处理需要使用之前批次的数据或者中间结果。

有状态转换包括基于追踪状态变化的转换(updateStateByKey)和滑动窗口的转换：

1. **UpdateStateByKey(func)**
2. **Window Operations 窗口操作**

#### 2) Output/Action

Output Operations 可以将 DStream 的数据输出到外部的数据库或文件系统。

当某个 Output Operations 被调用时，spark streaming 程序才会开始真正的计算过程(与 RDD 的 Action 类似)。

|          Output Operation          |                             含义                             |
| :--------------------------------: | :----------------------------------------------------------: |
|              print()               |                         打印到控制台                         |
| saveAsTextFiles(prefix, [suffix])  | 保存流的内容为文本文件，文件名为"prefix-TIME_IN_MS[.suffix]" |
| saveAsObjectFiles(prefix,[suffix]) | 保存流的内容为 SequenceFile，文件名为 "prefix-TIME_IN_MS[.suffix]" |
| saveAsHadoopFiles(prefix,[suffix]) | 保存流的内容为 hadoop 文件，文件名为"prefix-TIME_IN_MS[.suffix]" |
|          foreachRDD(func)          |             对 Dstream 里面的每个 RDD 执行 func              |

### 4. Spark Streaming 完成实时需求

#### 1) WordCount

- 首先在 linux 服务器上安装 nc 工具

  nc 是 netcat 的简称，原本是用来设置路由器,我们可以利用它向某个端口发送数据 yum install -y nc

- 启动一个服务端并开放 9999 端口,等一下往这个端口发数据

  nc -lk 9999

- 发送数据

- 接收数据，代码示例：

```
import org.apache.spark.streaming.dstream.{DStream, ReceiverInputDStream}
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.streaming.{Seconds, StreamingContext}

object WordCount {
  def main(args: Array[String]): Unit = {
    //1.创建StreamingContext
    //spark.master should be set as local[n], n > 1
    val conf = new SparkConf().setAppName("wc").setMaster("local[*]")
    val sc = new SparkContext(conf)
    sc.setLogLevel("WARN")
    val ssc = new StreamingContext(sc,Seconds(5))//5表示5秒中对数据进行切分形成一个RDD
    //2.监听Socket接收数据
    //ReceiverInputDStream就是接收到的所有的数据组成的RDD,封装成了DStream,接下来对DStream进行操作就是对RDD进行操作
    val dataDStream: ReceiverInputDStream[String] = ssc.socketTextStream("node01",9999)
    //3.操作数据
    val wordDStream: DStream[String] = dataDStream.flatMap(_.split(" "))
    val wordAndOneDStream: DStream[(String, Int)] = wordDStream.map((_,1))
    val wordAndCount: DStream[(String, Int)] = wordAndOneDStream.reduceByKey(_+_)
    wordAndCount.print()
    ssc.start()//开启
    ssc.awaitTermination()//等待停止
  }
}
```

#### 2) updateStateByKey

- **问题**：

在上面的那个案例中存在这样一个问题：

每个批次的单词次数都被正确的统计出来，但是结果不能累加！

如果需要累加需要使用 updateStateByKey(func)来更新状态。

代码示例：

```
import org.apache.spark.streaming.dstream.{DStream, ReceiverInputDStream}
import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.spark.{SparkConf, SparkContext}


object WordCount2 {
  def main(args: Array[String]): Unit = {
    //1.创建StreamingContext
    //spark.master should be set as local[n], n > 1
    val conf = new SparkConf().setAppName("wc").setMaster("local[*]")
    val sc = new SparkContext(conf)
    sc.setLogLevel("WARN")
    val ssc = new StreamingContext(sc,Seconds(5))//5表示5秒中对数据进行切分形成一个RDD
    //requirement failed: ....Please set it by StreamingContext.checkpoint().
    //注意:我们在下面使用到了updateStateByKey对当前数据和历史数据进行累加
    //那么历史数据存在哪?我们需要给他设置一个checkpoint目录
    ssc.checkpoint("./wc")//开发中HDFS
    //2.监听Socket接收数据
    //ReceiverInputDStream就是接收到的所有的数据组成的RDD,封装成了DStream,接下来对DStream进行操作就是对RDD进行操作
    val dataDStream: ReceiverInputDStream[String] = ssc.socketTextStream("node01",9999)
    //3.操作数据
    val wordDStream: DStream[String] = dataDStream.flatMap(_.split(" "))
    val wordAndOneDStream: DStream[(String, Int)] = wordDStream.map((_,1))
    //val wordAndCount: DStream[(String, Int)] = wordAndOneDStream.reduceByKey(_+_)
    //====================使用updateStateByKey对当前数据和历史数据进行累加====================
    val wordAndCount: DStream[(String, Int)] =wordAndOneDStream.updateStateByKey(updateFunc)
    wordAndCount.print()
    ssc.start()//开启
    ssc.awaitTermination()//等待优雅停止
  }
  //currentValues:当前批次的value值,如:1,1,1 (以测试数据中的hadoop为例)
  //historyValue:之前累计的历史值,第一次没有值是0,第二次是3
  //目标是把当前数据+历史数据返回作为新的结果(下次的历史数据)
  def updateFunc(currentValues:Seq[Int], historyValue:Option[Int] ):Option[Int] ={
    val result: Int = currentValues.sum + historyValue.getOrElse(0)
    Some(result)
  }
}
```

#### 3) reduceByKeyAndWindow

使用上面的代码已经能够完成对所有历史数据的聚合，但是实际中可能会有一些需求,需要对指定时间范围的数据进行统计。

比如:

百度/微博的热搜排行榜 统计最近 24 小时的热搜词,每隔 5 分钟更新一次，所以面对这样的需求我们需要使用窗口操作 Window Operations。

**图解**：

我们先提出一个问题：统计经过某红绿灯的汽车数量之和？

假设在一个红绿灯处，我们每隔 15 秒统计一次通过此红绿灯的汽车数量，如下图：

![1639395481929](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193803-60858.png)

可以把汽车的经过看成一个流，无穷的流，不断有汽车经过此红绿灯，因此无法统计总共的汽车数量。但是，我们可以换一种思路，每隔 15 秒，我们都将与上一次的结果进行 sum 操作（滑动聚合, 但是这个结果似乎还是无法回答我们的问题，根本原因在于流是无界的，我们不能限制流，但可以在有一个有界的范围内处理无界的流数据。

![1639395496051](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193817-247948.png)

因此，我们需要换一个问题的提法：每分钟经过某红绿灯的汽车数量之和？

这个问题，就相当于一个定义了一个 Window（窗口），window 的界限是 1 分钟，且每分钟内的数据互不干扰，因此也可以称为翻滚（不重合）窗口，如下图：

第一分钟的数量为 8，第二分钟是 22，第三分钟是 27。。。这样，1 个小时内会有 60 个 window。

再考虑一种情况，每 30 秒统计一次过去 1 分钟的汽车数量之和：

![1639395511817](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193833-813633.png)

此时，window 出现了重合。这样，1 个小时内会有 120 个 window。

滑动窗口转换操作的计算过程如下图所示：

![1639395525180](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193846-575906.png)

我们可以事先设定一个滑动窗口的长度(也就是窗口的持续时间)，并且设定滑动窗口的时间间隔(每隔多长时间执行一次计算)，

比如设置滑动窗口的长度(也就是窗口的持续时间)为 24H,设置滑动窗口的时间间隔(每隔多长时间执行一次计算)为 1H

那么意思就是:每隔 1H 计算最近 24H 的数据

![1639395540117](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193901-164781.png)

**代码示例**：

```
import org.apache.spark.streaming.dstream.{DStream, ReceiverInputDStream}
import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.spark.{SparkConf, SparkContext}

object WordCount3 {
  def main(args: Array[String]): Unit = {
    //1.创建StreamingContext
    //spark.master should be set as local[n], n > 1
    val conf = new SparkConf().setAppName("wc").setMaster("local[*]")
    val sc = new SparkContext(conf)
    sc.setLogLevel("WARN")
    val ssc = new StreamingContext(sc,Seconds(5))//5表示5秒中对数据进行切分形成一个RDD
    //2.监听Socket接收数据
    //ReceiverInputDStream就是接收到的所有的数据组成的RDD,封装成了DStream,接下来对DStream进行操作就是对RDD进行操作
    val dataDStream: ReceiverInputDStream[String] = ssc.socketTextStream("node01",9999)
    //3.操作数据
    val wordDStream: DStream[String] = dataDStream.flatMap(_.split(" "))
    val wordAndOneDStream: DStream[(String, Int)] = wordDStream.map((_,1))
    //4.使用窗口函数进行WordCount计数
    //reduceFunc: (V, V) => V,集合函数
    //windowDuration: Duration,窗口长度/宽度
    //slideDuration: Duration,窗口滑动间隔
    //注意:windowDuration和slideDuration必须是batchDuration的倍数
    //windowDuration=slideDuration:数据不会丢失也不会重复计算==开发中会使用
    //windowDuration>slideDuration:数据会重复计算==开发中会使用
    //windowDuration<slideDuration:数据会丢失
    //下面的代码表示:
    //windowDuration=10
    //slideDuration=5
    //那么执行结果就是每隔5s计算最近10s的数据
    //比如开发中让你统计最近1小时的数据,每隔1分钟计算一次,那么参数该如何设置?
    //windowDuration=Minutes(60)
    //slideDuration=Minutes(1)
    val wordAndCount: DStream[(String, Int)] = wordAndOneDStream.reduceByKeyAndWindow((a:Int,b:Int)=>a+b,Seconds(10),Seconds(5))
    wordAndCount.print()
    ssc.start()//开启
    ssc.awaitTermination()//等待优雅停止
  }
}
```

## 五、Structured Streaming

在 2.0 之前，Spark Streaming 作为核心 API 的扩展，针对实时数据流，提供了一套可扩展、高吞吐、可容错的流式计算模型。Spark Streaming 会接收实时数据源的数据，并切分成很多小的 batches，然后被 Spark Engine 执行，产出同样由很多小的 batchs 组成的结果流。本质上，这是一种 micro-batch（微批处理）的方式处理，用批的思想去处理流数据.这种设计让**Spark Streaming 面对复杂的流式处理场景时捉襟见肘**。

spark streaming 这种构建在微批处理上的流计算引擎，比较突出的问题就是处理延时较高（无法优化到秒以下的数量级），以及无法支持基于 event_time 的时间窗口做聚合逻辑。

spark 在 2.0 版本中发布了新的流计算的 API，Structured Streaming/结构化流。

**Structured Streaming 是一个基于 Spark SQL 引擎的可扩展、容错的流处理引擎**。统一了流、批的编程模型，你可以使用静态数据批处理一样的方式来编写流式计算操作。并且支持基于 event_time 的时间窗口的处理逻辑。

随着数据不断地到达，Spark 引擎会以一种增量的方式来执行这些操作，并且持续更新结算结果。可以使用 Scala、Java、Python 或 R 中的 DataSet／DataFrame API 来表示流聚合、事件时间窗口、流到批连接等。此外，Structured Streaming 会通过 checkpoint 和预写日志等机制来实现 Exactly-Once 语义。

简单来说，对于开发人员来说，根本不用去考虑是流式计算，还是批处理，只要使用同样的方式来编写计算操作即可，Structured Streaming 提供了快速、可扩展、容错、端到端的一次性流处理，而用户无需考虑更多细节。

默认情况下，结构化流式查询使用微批处理引擎进行处理，该引擎将数据流作为一系列小批处理作业进行处理，从而实现端到端的延迟，最短可达 100 毫秒，并且完全可以保证一次容错。**自 Spark 2.3 以来，引入了一种新的低延迟处理模式，称为连续处理，它可以在至少一次保证的情况下实现低至 1 毫秒的端到端延迟。也就是类似于 Flink 那样的实时流，而不是小批量处理**。实际开发可以根据应用程序要求选择处理模式，但是连续处理在使用的时候仍然有很多限制，目前大部分情况还是应该采用小批量模式。

### 1. API

- **Spark Streaming 时代** -DStream-RDD

  Spark Streaming 采用的数据抽象是 DStream，而本质上就是时间上连续的 RDD，对数据流的操作就是针对 RDD 的操作。

![1639395560944](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193921-907666.png)

- **Structured Streaming 时代 - DataSet/DataFrame -RDD**

  Structured Streaming 是 Spark2.0 新增的可扩展和高容错性的实时计算框架，它构建于 Spark SQL 引擎，把流式计算也统一到 DataFrame/Dataset 里去了。

  Structured Streaming 相比于 Spark Streaming 的进步就类似于 Dataset 相比于 RDD 的进步。

![1639395576671](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193937-515265.png)

### 2. 核心思想

Structured Streaming 最核心的思想就是将实时到达的数据看作是一个不断追加的 unbound table 无界表，到达流的每个数据项(RDD)就像是表中的一个新行被附加到无边界的表中.这样用户就可以用静态结构化数据的批处理查询方式进行流计算，如可以使用 SQL 对到来的每一行数据进行实时查询处理。

### 3. 应用场景

Structured Streaming 将数据源映射为类似于关系数据库中的表，然后将经过计算得到的结果映射为另一张表，完全以结构化的方式去操作流式数据，**这种编程模型非常有利于处理分析结构化的实时数据**；

### 4. Structured Streaming 实战

#### 1) 读取 Socket 数据

```
import org.apache.spark.SparkContext
import org.apache.spark.sql.streaming.Trigger
import org.apache.spark.sql.{DataFrame, Dataset, Row, SparkSession}

object WordCount {
  def main(args: Array[String]): Unit = {
    //1.创建SparkSession,因为StructuredStreaming的数据模型也是DataFrame/DataSet
    val spark: SparkSession = SparkSession.builder().master("local[*]").appName("SparkSQL").getOrCreate()
    val sc: SparkContext = spark.sparkContext
    sc.setLogLevel("WARN")
    //2.接收数据
    val dataDF: DataFrame = spark.readStream
      .option("host", "node01")
      .option("port", 9999)
      .format("socket")
      .load()
    //3.处理数据
    import spark.implicits._
    val dataDS: Dataset[String] = dataDF.as[String]
    val wordDS: Dataset[String] = dataDS.flatMap(_.split(" "))
    val result: Dataset[Row] = wordDS.groupBy("value").count().sort($"count".desc)
    //result.show()
    //Queries with streaming sources must be executed with writeStream.start();
    result.writeStream
      .format("console")//往控制台写
      .outputMode("complete")//每次将所有的数据写出
      .trigger(Trigger.ProcessingTime(0))//触发时间间隔,0表示尽可能的快
      //.option("checkpointLocation","./ckp")//设置checkpoint目录,socket不支持数据恢复,所以第二次启动会报错,需要注掉
      .start()//开启
      .awaitTermination()//等待停止
  }
}
```

#### 2) 读取目录下文本数据

```
import org.apache.spark.SparkContext
import org.apache.spark.sql.streaming.Trigger
import org.apache.spark.sql.types.StructType
import org.apache.spark.sql.{DataFrame, Dataset, Row, SparkSession}
/**
  * {"name":"json","age":23,"hobby":"running"}
  * {"name":"charles","age":32,"hobby":"basketball"}
  * {"name":"tom","age":28,"hobby":"football"}
  * {"name":"lili","age":24,"hobby":"running"}
  * {"name":"bob","age":20,"hobby":"swimming"}
  * 统计年龄小于25岁的人群的爱好排行榜
  */
object WordCount2 {
  def main(args: Array[String]): Unit = {
    //1.创建SparkSession,因为StructuredStreaming的数据模型也是DataFrame/DataSet
    val spark: SparkSession = SparkSession.builder().master("local[*]").appName("SparkSQL").getOrCreate()
    val sc: SparkContext = spark.sparkContext
    sc.setLogLevel("WARN")
    val Schema: StructType = new StructType()
      .add("name","string")
      .add("age","integer")
      .add("hobby","string")
    //2.接收数据
    import spark.implicits._
    // Schema must be specified when creating a streaming source DataFrame.
    val dataDF: DataFrame = spark.readStream.schema(Schema).json("D:\\data\\spark\\data")
    //3.处理数据
    val result: Dataset[Row] = dataDF.filter($"age" < 25).groupBy("hobby").count().sort($"count".desc)
    //4.输出结果
    result.writeStream
      .format("console")
      .outputMode("complete")
      .trigger(Trigger.ProcessingTime(0))
      .start()
      .awaitTermination()
  }
}
```

#### 3) 计算操作

**获得到 Source 之后的基本数据处理方式和之前学习的 DataFrame、DataSet 一致，不再赘述**。

**官网示例代码**：

```
case class DeviceData(device: String, deviceType: String, signal: Double, time: DateTime)
val df: DataFrame = ... // streaming DataFrame with IOT device data with schema { device: string, deviceType: string, signal: double, time: string }
val ds: Dataset[DeviceData] = df.as[DeviceData]    // streaming Dataset with IOT device data
// Select the devices which have signal more than 10
df.select("device").where("signal > 10")      // using untyped APIs
ds.filter(_.signal > 10).map(_.device)         // using typed APIs
// Running count of the number of updates for each device type
df.groupBy("deviceType").count()                 // using untyped API
// Running average signal for each device type
import org.apache.spark.sql.expressions.scalalang.typed
ds.groupByKey(_.deviceType).agg(typed.avg(_.signal))    // using typed API
```

#### 4) 输出

计算结果可以选择输出到多种设备并进行如下设定：

1. output mode：以哪种方式将 result table 的数据写入 sink,即是全部输出 complete 还是只输出新增数据；
2. format/output sink 的一些细节：数据格式、位置等。如 console；
3. query name：指定查询的标识。类似 tempview 的名字；
4. trigger interval：触发间隔，如果不指定，默认会尽可能快速地处理数据；
5. checkpointLocation：一般是 hdfs 上的目录。注意：Socket 不支持数据恢复，如果设置了，第二次启动会报错，Kafka 支持。

**output mode**：

![1639395595333](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/193957-774946.png)

每当结果表更新时，我们都希望将更改后的结果行写入外部接收器。

这里有三种输出模型:

1. Append mode：默认模式，新增的行才输出，每次更新结果集时，只将新添加到结果集的结果行输出到接收器。仅支持那些添加到结果表中的行永远不会更改的查询。因此，此模式保证每行仅输出一次。例如，仅查询 select，where，map，flatMap，filter，join 等会支持追加模式。**不支持聚合**
2. Complete mode：所有内容都输出，每次触发后，整个结果表将输出到接收器。聚合查询支持此功能。**仅适用于包含聚合操作的查询**。
3. Update mode：更新的行才输出，每次更新结果集时，仅将被更新的结果行输出到接收器(自 Spark 2.1.1 起可用)，**不支持排序**

**output sink**：

![1639395620395](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194021-63180.png)

- 说明：

**File sink**：输出存储到一个目录中。支持 parquet 文件，以及 append 模式。

```
writeStream
    .format("parquet")        // can be "orc", "json", "csv", etc.
    .option("path", "path/to/destination/dir")
    .start()
```

**Kafka sink**：将输出存储到 Kafka 中的一个或多个 topics 中。

```
writeStream
    .format("kafka")
    .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
    .option("topic", "updates")
    .start()
```

**Foreach sink**：对输出中的记录运行任意计算

```
writeStream
    .foreach(...)
    .start()
```

**Console sink**：将输出打印到控制台

```
writeStream
    .format("console")
    .start()
```

## 六、Spark 的两种核心 Shuffle

在 MapReduce 框架中， Shuffle 阶段是连接 Map 与 Reduce 之间的桥梁， Map 阶段通过 Shuffle 过程将数据输出到 Reduce 阶段中。**由于 Shuffle 涉及磁盘的读写和网络 I/O，因此 Shuffle 性能的高低直接影响整个程序的性能**。Spark 也有 Map 阶段和 Reduce 阶段，因此也会出现 Shuffle 。

### Spark Shuffle

Spark Shuffle 分为两种：一种是基于 Hash 的 Shuffle；另一种是基于 Sort 的 Shuffle。先介绍下它们的发展历程，有助于我们更好的理解 Shuffle：

在 Spark 1.1 之前， Spark 中只实现了一种 Shuffle 方式，即基于 Hash 的 Shuffle 。在 Spark 1.1 版本中引入了基于 Sort 的 Shuffle 实现方式，并且 Spark 1.2 版本之后，默认的实现方式从基于 Hash 的 Shuffle 修改为基于 Sort 的 Shuffle 实现方式，即使用的 ShuffleManager 从默认的 hash 修改为 sort。**在 Spark 2.0 版本中， Hash Shuffle 方式己经不再使用**。

Spark 之所以一开始就提供基于 Hash 的 Shuffle 实现机制，其主要目的之一就是为了避免不需要的排序，大家想下 Hadoop 中的 MapReduce，是将 sort 作为固定步骤，有许多并不需要排序的任务，MapReduce 也会对其进行排序，造成了许多不必要的开销。

在基于 Hash 的 Shuffle 实现方式中，每个 Mapper 阶段的 Task 会为每个 Reduce 阶段的 Task 生成一个文件，通常会产生大量的文件（即对应为 M*R 个中间文件，其中， M 表示 Mapper 阶段的 Task 个数， R 表示 Reduce 阶段的 Task 个数） 伴随大量的随机磁盘 I/O 操作与大量的内存开销。

为了缓解上述问题，在 Spark 0.8.1 版本中为基于 Hash 的 Shuffle 实现引入了 **Shuffle Consolidate 机制（即文件合并机制）**，将 Mapper 端生成的中间文件进行合并的处理机制。通过配置属性`spark.shuffie.consolidateFiles=true`，减少中间生成的文件数量。**通过文件合并，可以将中间文件的生成方式修改为每个执行单位为每个 Reduce 阶段的 Task 生成一个文件**。

> 执行单位对应为：每个 Mapper 端的 Cores 数／每个 Task 分配的 Cores 数（默认为 1) 。最终可以将文件个数从 M*R 修改为 E*C/T*R，其中， E 表示 Executors 个数， C 表示可用 Cores 个数， T 表示 Task 分配的 Cores 数。

Spark1.1 版本引入了 Sort Shuffle：

基于 Hash 的 Shuffle 的实现方式中，**生成的中间结果文件的个数都会依赖于 Reduce 阶段的 Task 个数**，即 Reduce 端的并行度，因此文件数仍然不可控，无法真正解决问题。为了更好地解决问题，在 Spark1.1 版本引入了基于 Sort 的 Shuffle 实现方式，并且在 Spark 1.2 版本之后，默认的实现方式也从基于 Hash 的 Shuffle，修改为基于 Sort 的 Shuffle 实现方式，即使用的 ShuffleManager 从默认的 hash 修改为 sort。

在基于 Sort 的 Shuffle 中，每个 Mapper 阶段的 Task 不会为每 Reduce 阶段的 Task 生成一个单独的文件，**而是全部写到一个数据（Data）文件中，同时生成一个索引（Index）文件**， Reduce 阶段的各个 Task 可以通过该索引文件获取相关的数据。避免产生大量文件的直接收益就是降低随机磁盘 I/0 与内存的开销。

最终生成的文件个数减少到 2M ，其中 M 表示 Mapper 阶段的 Task 个数，每个 Mapper 阶段的 Task 分别生成两个文件（1 个数据文件、 1 个索引文件），最终的文件个数为 M 个数据文件与 M 个索引文件。因此，最终文件个数是 2*M 个。

从 Spark 1.4 版本开始，在 Shuffle 过程中也引入了基于 Tungsten-Sort 的 Shuffie 实现方式，通 Tungsten 项目所做的优化，可以极大提高 Spark 在数据处理上的性能。(Tungsten 翻译为中文是钨丝)

> 注：在一些特定的应用场景下，采用基于 Hash 实现 Shuffle 机制的性能会超过基于 Sort 的 Shuffle 实现机制。

一张图了解下 Spark Shuffle 的迭代历史：

![1639395702977](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194144-503864.png)

**为什么 Spark 最终还是放弃了 HashShuffle ，使用了 Sorted-Based Shuffle？**

我们可以从 Spark 最根本要优化和迫切要解决的问题中找到答案，使用 HashShuffle 的 Spark 在 Shuffle 时产生大量的文件。当数据量越来越多时，**产生的文件量是不可控的**，这严重制约了 Spark 的性能及扩展能力，所以 Spark 必须要解决这个问题，减少 Mapper 端 ShuffleWriter 产生的文件数量，这样便可以让 Spark 从几百台集群的规模瞬间变成可以支持几千台，甚至几万台集群的规模。

但使用 Sorted-Based Shuffle 就完美了吗，答案是否定的，Sorted-Based Shuffle 也有缺点，其缺点反而是它**排序**的特性，它强制要求数据在 Mapper 端必须先进行排序，所以导致它排序的速度有点慢。好在出现了 Tungsten-Sort Shuffle ，**它对排序算法进行了改进，优化了排序的速度**。Tungsten-Sort Shuffle 已经并入了 Sorted-Based Shuffle，Spark 的引擎会自动识别程序需要的是 Sorted-Based Shuffle，还是 Tungsten-Sort Shuffle。

**下面详细剖析每个 Shuffle 的底层执行原理：**

### 一、Hash Shuffle 解析

以下的讨论都假设每个 Executor 有 1 个 cpu core。

#### 1. HashShuffleManager

shuffle write 阶段，主要就是在一个 stage 结束计算之后，为了下一个 stage 可以执行 shuffle 类的算子（比如 reduceByKey），而将每个 task 处理的数据按 **key** 进行“划分”。所谓“划分”，就是**对相同的 key 执行 hash 算法**，从而将**相同 key 都写入同一个磁盘文件中**，**而每一个磁盘文件都只属于下游 stage 的一个 task**。在将数据写入磁盘之前，会先将数据写入内存缓冲中，当内存缓冲填满之后，才会溢写到磁盘文件中去。

下一个 stage 的 task 有多少个，当前 stage 的每个 task 就要创建多少份磁盘文件。比如下一个 stage 总共有 100 个 task，那么当前 stage 的每个 task 都要创建 100 份磁盘文件。

如果当前 stage 有 50 个 task，总共有 10 个 Executor，每个 Executor 执行 5 个 task，那么每个 Executor 上总共就要创建 500 个磁盘文件，所有 Executor 上会创建 5000 个磁盘文件。由此可见，**未经优化的 shuffle write 操作所产生的磁盘文件的数量是极其惊人的**。

shuffle read 阶段，通常就是一个 stage 刚开始时要做的事情。此时该 stage 的**每一个 task 就需要将上一个 stage 的计算结果中的所有相同 key，从各个节点上通过网络都拉取到自己所在的节点上，然后进行 key 的聚合或连接等操作**。由于 shuffle write 的过程中，map task 给下游 stage 的每个 reduce task 都创建了一个磁盘文件，因此 shuffle read 的过程中，每个 reduce task 只要从上游 stage 的所有 map task 所在节点上，拉取属于自己的那一个磁盘文件即可。

shuffle read 的拉取过程是一边拉取一边进行聚合的。每个 shuffle read task 都会有一个自己的 buffer 缓冲，每次都只能拉取与 buffer 缓冲相同大小的数据，然后通过内存中的一个 Map 进行聚合等操作。聚合完一批数据后，再拉取下一批数据，并放到 buffer 缓冲中进行聚合操作。以此类推，直到最后将所有数据到拉取完，并得到最终的结果。

HashShuffleManager 工作原理如下图所示：

![1639395721678](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194203-662364.png)未优化的HashShuffleManager工作原理

#### 2. 优化的 HashShuffleManager

为了优化 HashShuffleManager 我们可以设置一个参数：`spark.shuffle.consolidateFiles`，该参数默认值为 false，将其设置为 true 即可开启优化机制，通常来说，**如果我们使用 HashShuffleManager，那么都建议开启这个选项**。

开启 consolidate 机制之后，在 shuffle write 过程中，task 就不是为下游 stage 的每个 task 创建一个磁盘文件了，此时会出现**shuffleFileGroup**的概念，每个 shuffleFileGroup 会对应一批磁盘文件，磁盘文件的数量与下游 stage 的 task 数量是相同的。一个 Executor 上有多少个 cpu core，就可以并行执行多少个 task。而第一批并行执行的每个 task 都会创建一个 shuffleFileGroup，并将数据写入对应的磁盘文件内。

当 Executor 的 cpu core 执行完一批 task，接着执行下一批 task 时，下一批 task 就会复用之前已有的 shuffleFileGroup，包括其中的磁盘文件，也就是说，此时 task 会将数据写入已有的磁盘文件中，而不会写入新的磁盘文件中。因此，**consolidate 机制允许不同的 task 复用同一批磁盘文件，这样就可以有效将多个 task 的磁盘文件进行一定程度上的合并，从而大幅度减少磁盘文件的数量，进而提升 shuffle write 的性能**。

假设第二个 stage 有 100 个 task，第一个 stage 有 50 个 task，总共还是有 10 个 Executor（Executor CPU 个数为 1），每个 Executor 执行 5 个 task。那么原本使用未经优化的 HashShuffleManager 时，每个 Executor 会产生 500 个磁盘文件，所有 Executor 会产生 5000 个磁盘文件的。但是此时经过优化之后，每个 Executor 创建的磁盘文件的数量的计算公式为：`cpu core的数量 * 下一个stage的task数量`，也就是说，每个 Executor 此时只会创建 100 个磁盘文件，所有 Executor 只会创建 1000 个磁盘文件。

> 这个功能优点明显，但为什么 Spark 一直没有在基于 Hash Shuffle 的实现中将功能设置为默认选项呢，官方给出的说法是这个功能还欠稳定。

优化后的 HashShuffleManager 工作原理如下图所示：

![1639395737705](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194218-766086.png)

优化后的HashShuffleManager工作原理

##### 基于 Hash 的 Shuffle 机制的优缺点

**优点**：

- 可以省略不必要的排序开销。
- 避免了排序所需的内存开销。

**缺点**：

- 生产的文件过多，会对文件系统造成压力。
- 大量小文件的随机读写带来一定的磁盘开销。
- 数据块写入时所需的缓存空间也会随之增加，对内存造成压力。

### 二、SortShuffle 解析

SortShuffleManager 的运行机制主要分成三种：

1. **普通运行机制**；
2. **bypass 运行机制**，当 shuffle read task 的数量小于等于`spark.shuffle.sort.bypassMergeThreshold`参数的值时（默认为 200），就会启用 bypass 机制；
3. **Tungsten Sort 运行机制**，开启此运行机制需设置配置项 `spark.shuffle.manager=tungsten-sort`。开启此项配置也不能保证就一定采用此运行机制（后面会解释）。

#### 1. 普通运行机制

在该模式下，**数据会先写入一个内存数据结构中**，此时根据不同的 shuffle 算子，可能选用不同的数据结构。如果是 reduceByKey 这种聚合类的 shuffle 算子，那么会选用 Map 数据结构，一边通过 Map 进行聚合，一边写入内存；如果是 join 这种普通的 shuffle 算子，那么会选用 Array 数据结构，直接写入内存。接着，每写一条数据进入内存数据结构之后，就会判断一下，是否达到了某个临界阈值。如果达到临界阈值的话，那么就会尝试将内存数据结构中的数据溢写到磁盘，然后清空内存数据结构。

在溢写到磁盘文件之前，会先根据 key 对内存数据结构中已有的数据进行排序。排序过后，会分批将数据写入磁盘文件。默认的 batch 数量是 10000 条，也就是说，排序好的数据，会以每批 1 万条数据的形式分批写入磁盘文件。写入磁盘文件是通过 Java 的 BufferedOutputStream 实现的。**BufferedOutputStream 是 Java 的缓冲输出流，首先会将数据缓冲在内存中，当内存缓冲满溢之后再一次写入磁盘文件中，这样可以减少磁盘 IO 次数，提升性能**。

一个 task 将所有数据写入内存数据结构的过程中，会发生多次磁盘溢写操作，也就会产生多个临时文件。最后会将之前所有的临时磁盘文件都进行合并，这就是**merge 过程**，此时会将之前所有临时磁盘文件中的数据读取出来，然后依次写入最终的磁盘文件之中。此外，由于一个 task 就只对应一个磁盘文件，也就意味着该 task 为下游 stage 的 task 准备的数据都在这一个文件中，因此还会单独写一份**索引文件**，其中标识了下游各个 task 的数据在文件中的 start offset 与 end offset。

SortShuffleManager 由于有一个磁盘文件 merge 的过程，因此大大减少了文件数量。比如第一个 stage 有 50 个 task，总共有 10 个 Executor，每个 Executor 执行 5 个 task，而第二个 stage 有 100 个 task。由于每个 task 最终只有一个磁盘文件，因此此时每个 Executor 上只有 5 个磁盘文件，所有 Executor 只有 50 个磁盘文件。

普通运行机制的 SortShuffleManager 工作原理如下图所示：

![1639395756169](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194237-596327.png)

普通运行机制的SortShuffleManager工作原理

#### 2. bypass 运行机制

**Reducer 端任务数比较少的情况下，基于 Hash Shuffle 实现机制明显比基于 Sort Shuffle 实现机制要快，因此基于 Sort huffle 实现机制提供了一个回退方案，就是 bypass 运行机制**。对于 Reducer 端任务数少于配置属性`spark.shuffle.sort.bypassMergeThreshold`设置的个数时，使用带 Hash 风格的回退计划。

bypass 运行机制的触发条件如下：

- shuffle map task 数量小于`spark.shuffle.sort.bypassMergeThreshold=200`参数的值。
- 不是聚合类的 shuffle 算子。

此时，每个 task 会为每个下游 task 都创建一个临时磁盘文件，并将数据按 key 进行 hash 然后根据 key 的 hash 值，将 key 写入对应的磁盘文件之中。当然，写入磁盘文件时也是先写入内存缓冲，缓冲写满之后再溢写到磁盘文件的。最后，同样会将所有临时磁盘文件都合并成一个磁盘文件，并创建一个单独的索引文件。

该过程的磁盘写机制其实跟未经优化的 HashShuffleManager 是一模一样的，因为都要创建数量惊人的磁盘文件，只是在最后会做一个磁盘文件的合并而已。因此少量的最终磁盘文件，也让该机制相对未经优化的 HashShuffleManager 来说，shuffle read 的性能会更好。

而该机制与普通 SortShuffleManager 运行机制的不同在于：

第一，磁盘写机制不同；sorted是基于排序的写磁盘，但是bypass是基于hash的写磁盘。

第二，不会进行排序。也就是说，**启用该机制的最大好处在于，shuffle write 过程中，不需要进行数据的排序操作**，也就节省掉了这部分的性能开销。

bypass 运行机制的 SortShuffleManager 工作原理如下图所示：

![1639395772870](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/06/135614-25707.png)

bypass运行机制的SortShuffleManager工作原理

#### 3. Tungsten Sort Shuffle 运行机制

基于 Tungsten Sort 的 Shuffle 实现机制主要是借助 Tungsten 项目所做的优化来高效处理 Shuffle。

Spark 提供了配置属性，用于选择具体的 Shuffle 实现机制，但需要说明的是，虽然默认情况下 Spark 默认开启的是基于 SortShuffle 实现机制，但实际上，参考 Shuffle 的框架内核部分可知基于 SortShuffle 的实现机制与基于 Tungsten Sort Shuffle 实现机制都是使用 SortShuffleManager，而内部使用的具体的实现机制，是通过提供的两个方法进行判断的：

> **对应非基于 Tungsten Sort 时，通过 SortShuffleWriter.shouldBypassMergeSort 方法判断是否需要回退到 Hash 风格的 Shuffle 实现机制，当该方法返回的条件不满足时，则通过 SortShuffleManager.canUseSerializedShuffle 方法判断是否需要采用基于 Tungsten Sort Shuffle 实现机制，而当这两个方法返回都为 false，即都不满足对应的条件时，会自动采用普通运行机制。**

因此，当设置了 `spark.shuffle.manager=tungsten-sort` 时，也不能保证就一定采用基于 Tungsten Sort 的 Shuffle 实现机制。

要实现 Tungsten Sort Shuffle 机制需要满足以下条件：

1. **Shuffle 依赖中不带聚合操作或没有对输出进行排序的要求**。
2. Shuffle 的序列化器支持序列化值的重定位（当前仅支持 KryoSerializer Spark SQL 框架自定义的序列化器）。
3. Shuffle 过程中的输出分区个数少于 16777216 个。

实际上，使用过程中还有其他一些限制，如引入 Page 形式的内存管理模型后，内部单条记录的长度不能超过 128 MB （具体内存模型可以参考 PackedRecordPointer 类）。另外，分区个数的限制也是该内存模型导致的。

所以，目前使用基于 Tungsten Sort Shuffle 实现机制条件还是比较苛刻的。

##### 基于 Sort 的 Shuffle 机制的优缺点

**优点**：

- 小文件的数量大量减少，Mapper 端的内存占用变少；
- Spark 不仅可以处理小规模的数据，即使处理大规模的数据，也不会很容易达到性能瓶颈。

**缺点**：

- 如果 Mapper 中 Task 的数量过大，依旧会产生很多小文件，此时在 Shuffle 传数据的过程中到 Reducer 端， Reducer 会需要同时大量地记录进行反序列化，导致大量内存消耗和 GC 负担巨大，造成系统缓慢，甚至崩溃；
- 强制了在 Mapper 端必须要排序，即使数据本身并不需要排序；
- 它要基于记录本身进行排序，这就是 Sort-Based Shuffle 最致命的性能消耗。

## 七、Spark 底层执行原理

### Spark 运行流程

![1639395827782](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194348-308663.png)

Spark运行流程

具体运行流程如下：

1. SparkContext 向资源管理器注册并向资源管理器申请运行 Executor
2. 资源管理器分配 Executor，然后资源管理器启动 Executor
3. Executor 发送心跳至资源管理器
4. **SparkContext 构建 DAG 有向无环图**
5. **将 DAG 分解成 Stage（TaskSet）**
6. **把 Stage 发送给 TaskScheduler**
7. **Executor 向 SparkContext 申请 Task**
8. **TaskScheduler 将 Task 发送给 Executor 运行**
9. **同时 SparkContext 将应用程序代码发放给 Executor**
10. Task 在 Executor 上运行，运行完毕释放所有资源

#### 1. 从代码角度看 DAG 图的构建

```scala
Val lines1 = sc.textFile(inputPath1).map(...).map(...)

Val lines2 = sc.textFile(inputPath2).map(...)

Val lines3 = sc.textFile(inputPath3)

Val dtinone1 = lines2.union(lines3)

Val dtinone = lines1.join(dtinone1)

dtinone.saveAsTextFile(...)

dtinone.filter(...).foreach(...)
```

上述代码的 DAG 图如下所示：

![1639395845014](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194406-812232.png)

Spark 内核会在需要计算发生的时刻绘制一张关于计算路径的有向无环图，也就是如上图所示的 DAG。

**Spark 的计算发生在 RDD 的 Action 操作，而对 Action 之前的所有 Transformation，Spark 只是记录下 RDD 生成的轨迹，而不会触发真正的计算**。

#### 2. 将 DAG 划分为 Stage 核心算法

一个 Application 可以有多个 job 多个 Stage：

Spark Application 中可以因为不同的 Action 触发众多的 job，一个 Application 中可以有很多的 job，每个 job 是由一个或者多个 Stage 构成的，后面的 Stage 依赖于前面的 Stage，也就是说只有前面依赖的 Stage 计算完毕后，后面的 Stage 才会运行。

划分依据：

**Stage 划分的依据就是宽依赖**，像 reduceByKey，groupByKey 等算子，会导致宽依赖的产生。

> 回顾下宽窄依赖的划分原则：
> **窄依赖**：父 RDD 的一个分区只会被子 RDD 的一个分区依赖。即一对一或者多对一的关系，可理解为独生子女。常见的窄依赖有：map、filter、union、mapPartitions、mapValues、join（父 RDD 是 hash-partitioned）等。
>
> **宽依赖**：父 RDD 的一个分区会被子 RDD 的多个分区依赖(涉及到 shuffle)。即一对多的关系，可理解为超生。常见的宽依赖有 groupByKey、partitionBy、reduceByKey、join（父 RDD 不是 hash-partitioned）等。

**核心算法：回溯算法**

**从后往前回溯/反向解析，遇到窄依赖加入本 Stage，遇见宽依赖进行 Stage 切分。**

Spark 内核会从触发 Action 操作的那个 RDD 开始**从后往前推**，首先会为最后一个 RDD 创建一个 Stage，然后继续倒推，如果发现对某个 RDD 是宽依赖，那么就会将宽依赖的那个 RDD 创建一个新的 Stage，那个 RDD 就是新的 Stage 的最后一个 RDD。然后依次类推，继续倒推，根据窄依赖或者宽依赖进行 Stage 的划分，直到所有的 RDD 全部遍历完成为止。

#### 3. 将 DAG 划分为 Stage 剖析

![1639395893175](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194454-947407.png)

**一个 Spark 程序可以有多个 DAG(有几个 Action，就有几个 DAG，上图最后只有一个 Action（图中未表现）,那么就是一个 DAG)**。

一个 DAG 可以有多个 Stage(根据宽依赖/shuffle 进行划分)。

**同一个 Stage 可以有多个 Task 并行执行**(**task 数=分区数**，如上图，Stage1 中有三个分区 P1、P2、P3，对应的也有三个 Task)。

可以看到这个 DAG 中只 reduceByKey 操作是一个宽依赖，Spark 内核会以此为边界将其前后划分成不同的 Stage。

同时我们可以注意到，在图中 Stage1 中，**从 textFile 到 flatMap 到 map 都是窄依赖，这几步操作可以形成一个流水线操作，通过 flatMap 操作生成的 partition 可以不用等待整个 RDD 计算结束，而是继续进行 map 操作，这样大大提高了计算的效率**。

#### 4. 提交 Stages

调度阶段的提交，最终会被转换成一个**任务集**的提交，DAGScheduler 通过 TaskScheduler 接口提交任务集，这个任务集最终会触发 TaskScheduler 构建一个 **TaskSetManager 的实例来管理这个任务集的生命周期**，对于 DAGScheduler 来说，提交调度阶段的工作到此就完成了。

而 TaskScheduler 的具体实现则会在得到计算资源的时候，进一步通过 TaskSetManager 调度具体的任务到对应的 Executor 节点上进行运算。

![1639395933635](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194535-882822.png)

#### 5. 监控 Job、Task、Executor

1. DAGScheduler 监控 Job 与 Task：

要保证相互依赖的作业调度阶段能够得到顺利的调度执行，**DAGScheduler 需要监控当前作业调度阶段乃至任务的完成情况**。

这通过对外暴露一系列的回调函数来实现的，对于 TaskScheduler 来说，这些回调函数主要包括任务的开始结束失败、任务集的失败，DAGScheduler 根据这些任务的生命周期信息进一步维护作业和调度阶段的状态信息。

1. DAGScheduler 监控 Executor 的生命状态：

TaskScheduler 通过回调函数通知 DAGScheduler 具体的 Executor 的生命状态，**如果某一个 Executor 崩溃了，则对应的调度阶段任务集的 ShuffleMapTask 的输出结果也将标志为不可用，这将导致对应任务集状态的变更，进而重新执行相关计算任务，以获取丢失的相关数据**。

#### 6. 获取任务执行结果

1. 结果 DAGScheduler：

一个具体的任务在 Executor 中执行完毕后，其结果需要以某种形式返回给 DAGScheduler，根据任务类型的不同，任务结果的返回方式也不同。

1. 两种结果，中间结果与最终结果：

对于 FinalStage 所对应的任务，返回给 DAGScheduler 的是运算结果本身。

而对于中间调度阶段对应的任务 ShuffleMapTask，返回给 DAGScheduler 的是一个 **MapStatus** 里的相关存储信息，而非结果本身，这些存储位置信息将作为下一个调度阶段的任务获取输入数据的依据。

1. 两种类型，**DirectTaskResult 与 IndirectTaskResult**：

根据任务结果大小的不同，ResultTask 返回的结果又分为两类：

如果结果足够小，则直接放在 DirectTaskResult 对象内中。

如果超过特定尺寸则在 Executor 端会将 DirectTaskResult 先序列化，再把序列化的结果作为一个数据块存放在 BlockManager 中，然后将 BlockManager 返回的 BlockID 放在 IndirectTaskResult 对象中返回给 TaskScheduler，TaskScheduler 进而调用 TaskResultGetter 将 IndirectTaskResult 中的 BlockID 取出并通过 BlockManager 最终取得对应的 DirectTaskResult。

#### 7. 任务调度总体诠释

**一张图说明任务总体调度：**

![1639395949322](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194550-864075.png)

### Spark 运行架构特点

#### 1. Executor 进程专属

**每个 Application 获取专属的 Executor 进程，该进程在 Application 期间一直驻留，并以多线程方式运行 Tasks**。

Spark Application 不能跨应用程序共享数据，除非将数据写入到外部存储系统。如图所示：

![1639395985521](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194627-15396.png)

#### 2. 支持多种资源管理器

Spark 与资源管理器无关，只要能够获取 Executor 进程，并能保持相互通信就可以了。

Spark 支持资源管理器包含：Standalone、On Mesos、On YARN、Or On EC2。如图所示:

![1639396010079](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194651-952705.png)

#### 3. Job 提交就近原则

**提交 SparkContext 的 Client 应该靠近 Worker 节点(运行 Executor 的节点)**，最好是在同一个 Rack(机架)里，因为 Spark Application 运行过程中 SparkContext 和 Executor 之间有大量的信息交换;

如果想在远程集群中运行，最好使用 RPC 将 SparkContext 提交给集群，**不要远离 Worker 运行 SparkContext**。

如图所示:

![1639396030617](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194711-410137.png)

#### 4. 移动程序而非移动数据的原则执行

**移动程序而非移动数据的原则执行，Task 采用了数据本地性和推测执行的优化机制**。

关键方法：taskIdToLocations、getPreferedLocations。

如图所示:

![1639396053165](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194734-242051.png)

## 八、Spark 数据倾斜

就是数据分到各个区的数量不太均匀,**可以自定义分区器**,想怎么分就怎么分。

**Spark中的数据倾斜问题主要指shuffle过程中出现的数据倾斜问题，是由于不同的key对应的数据量不同导致的不同task所处理的数据量不同的问题**。

例如，reduced端一共要处理100万条数据，第一个和第二个task分别被分配到了1万条数据，计算5分钟内完成，第三个task分配到了98万数据，此时第三个task可能需要10个小时完成，这使得整个Spark作业需要10个小时才能运行完成，这就是数据倾斜所带来的后果。

> 注意，要区分开**数据倾斜**与**数据过量**这两种情况：
>
> 数据倾斜是指少数task被分配了绝大多数的数据，因此少数task运行缓慢；
>
> 数据过量是指所有task被分配的数据量都很大，相差不多，所有task都运行缓慢。

数据倾斜的表现：

1. Spark作业的大部分task都执行迅速，**只有有限的几个task执行的非常慢**，此时可能出现了数据倾斜，作业可以运行，但是运行得非常慢；
2. Spark作业的大部分task都执行迅速，但是有的**task在运行过程中会突然报出OOM**，反复执行几次都在某一个task报出OOM错误，此时可能出现了数据倾斜，作业无法正常运行。定位数据倾斜问题：
3. 查阅代码中的shuffle算子，例如reduceByKey、countByKey、groupByKey、join等算子，**根据代码逻辑判断此处是否会出现数据倾斜**；
4. **查看Spark作业的log文件，log文件对于错误的记录会精确到代码的某一行**，可以根据异常定位到的代码位置来明确错误发生在第几个stage，对应的shuffle算子是哪一个；

### 1. 预聚合原始数据

**1. 避免shuffle过程**

绝大多数情况下，Spark作业的数据来源都是Hive表，这些Hive表基本都是经过ETL之后的昨天的数据。为了避免数据倾斜，我们可以考虑避免shuffle过程，如果避免了shuffle过程，那么从根本上就消除了发生数据倾斜问题的可能。

如果Spark作业的数据来源于Hive表，**那么可以先在Hive表中对数据进行聚合**，例如按照key进行分组，将同一key对应的所有value用一种特殊的格式拼接到一个字符串里去，这样，一个key就只有一条数据了；之后，对一个key的所有value进行处理时，只需要进行map操作即可，无需再进行任何的shuffle操作。通过上述方式就避免了执行shuffle操作，也就不可能会发生任何的数据倾斜问题。

对于Hive表中数据的操作，不一定是拼接成一个字符串，也可以是直接对key的每一条数据进行累计计算。要区分开，处理的数据量大和数据倾斜的区别。

**2. 增大key粒度（减小数据倾斜可能性，增大每个task的数据量）**

如果没有办法对每个key聚合出来一条数据，在特定场景下，可以考虑扩大key的聚合粒度。

例如，目前有10万条用户数据，当前key的粒度是（省，城市，区，日期），现在我们考虑扩大粒度，将key的粒度扩大为（省，城市，日期），这样的话，key的数量会减少，key之间的数据量差异也有可能会减少，由此可以减轻数据倾斜的现象和问题。（此方法只针对特定类型的数据有效，当应用场景不适宜时，会加重数据倾斜）

### 2. 预处理导致倾斜的key

**1. 过滤**

如果在Spark作业中允许丢弃某些数据，那么可以考虑将可能导致数据倾斜的key进行过滤，滤除可能导致数据倾斜的key对应的数据，这样，在Spark作业中就不会发生	数据倾斜了。

**2. 使用随机key**

当使用了类似于groupByKey、reduceByKey这样的算子时，可以考虑使用随机key实现双重聚合，如下图所示：

![1639396075335](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194756-554148.png)

随机key实现双重聚合

首先，通过map算子给每个数据的key添加随机数前缀，对key进行打散，将原先一样的key变成不一样的key，然后进行第一次聚合，这样就可以让原本被一个task处理的数据分散到多个task上去做局部聚合；随后，去除掉每个key的前缀，再次进行聚合。

此方法对于由groupByKey、reduceByKey这类算子造成的数据倾斜有比较好的效果，仅仅适用于聚合类的shuffle操作，适用范围相对较窄。如果是join类的shuffle操作，还得用其他的解决方案。

此方法也是前几种方案没有比较好的效果时要尝试的解决方案。

**3. sample采样对倾斜key单独进行join**

在Spark中，**如果某个RDD只有一个key，那么在shuffle过程中会默认将此key对应的数据打散，由不同的reduce端task进行处理**。

所以当由单个key导致数据倾斜时，可有将发生数据倾斜的key单独提取出来，组成一个RDD，然后用这个原本会导致倾斜的key组成的RDD和其他RDD单独join，此时，根据Spark的运行机制，此RDD中的数据会在shuffle阶段被分散到多个task中去进行join操作。

倾斜key单独join的流程如下图所示：

![1639396089108](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194810-77770.png)

倾斜key单独join流程

适用场景分析：

对于RDD中的数据，可以将其转换为一个中间表，或者是直接使用countByKey()的方式，看一下这个RDD中各个key对应的数据量，此时如果你发现整个RDD就一个key的数据量特别多，那么就可以考虑使用这种方法。

当数据量非常大时，可以考虑使用sample采样获取10%的数据，然后分析这10%的数据中哪个key可能会导致数据倾斜，然后将这个key对应的数据单独提取出来。

不适用场景分析：

如果一个RDD中导致数据倾斜的key很多，那么此方案不适用。

### 3. 提高reduce并行度

当方案一和方案二对于数据倾斜的处理没有很好的效果时，可以考虑提高shuffle过程中的reduce端并行度，reduce端并行度的提高就增加了reduce端task的数量，那么每个task分配到的数据量就会相应减少，由此缓解数据倾斜问题。

**1. reduce端并行度的设置**

在大部分的shuffle算子中，都可以传入一个并行度的设置参数，比如reduceByKey(500)，这个参数会决定shuffle过程中reduce端的并行度，在进行shuffle操作的时候，就会对应着创建指定数量的reduce task。对于Spark SQL中的shuffle类语句，比如group by、join等，需要设置一个参数，即`spark.sql.shuffle.partitions`，该参数代表了shuffle read task的并行度，该值默认是200，对于很多场景来说都有点过小。

增加shuffle read task的数量，可以让原本分配给一个task的多个key分配给多个task，从而让每个task处理比原来更少的数据。

举例来说，如果原本有5个key，每个key对应10条数据，这5个key都是分配给一个task的，那么这个task就要处理50条数据。而增加了shuffle read task以后，每个task就分配到一个key，即每个task就处理10条数据，那么自然每个task的执行时间都会变短了。

**2. reduce端并行度设置存在的缺陷**

**提高reduce端并行度并没有从根本上改变数据倾斜的本质和问题（方案一和方案二从根本上避免了数据倾斜的发生）**，只是尽可能地去缓解和减轻shuffle reduce task的数据压力，以及数据倾斜的问题，适用于有较多key对应的数据量都比较大的情况。

该方案通常无法彻底解决数据倾斜，因为如果出现一些极端情况，比如某个key对应的数据量有100万，那么无论你的task数量增加到多少，这个对应着100万数据的key肯定还是会分配到一个task中去处理，因此注定还是会发生数据倾斜的。所以这种方案只能说是在发现数据倾斜时尝试使用的一种手段，尝试去用最简单的方法缓解数据倾斜而已，或者是和其他方案结合起来使用。

在理想情况下，reduce端并行度提升后，会在一定程度上减轻数据倾斜的问题，甚至基本消除数据倾斜；但是，在一些情况下，只会让原来由于数据倾斜而运行缓慢的task运行速度稍有提升，或者避免了某些task的OOM问题，但是，仍然运行缓慢，此时，要及时放弃方案三，开始尝试后面的方案。

### 4. 使用map join

正常情况下，join操作都会执行shuffle过程，并且执行的是reduce join，也就是先将所有相同的key和对应的value汇聚到一个reduce task中，然后再进行join。普通join的过程如下图所示：

![1639396148487](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194909-324674.png)

普通join过程

普通的join是会走shuffle过程的，而一旦shuffle，就相当于会将相同key的数据拉取到一个shuffle read task中再进行join，此时就是reduce join。但是如果一个RDD是比较小的，则可以采用广播小RDD全量数据+map算子来实现与join同样的效果，也就是map join，此时就不会发生shuffle操作，也就不会发生数据倾斜。

> **注意：RDD是并不能直接进行广播的，只能将RDD内部的数据通过collect拉取到Driver内存然后再进行广播**。

**1. 核心思路：**

不使用join算子进行连接操作，而使用broadcast变量与map类算子实现join操作，进而完全规避掉shuffle类的操作，彻底避免数据倾斜的发生和出现。将较小RDD中的数据直接通过collect算子拉取到Driver端的内存中来，然后对其创建一个broadcast变量；接着对另外一个RDD执行map类算子，在算子函数内，从broadcast变量中获取较小RDD的全量数据，与当前RDD的每一条数据按照连接key进行比对，如果连接key相同的话，那么就将两个RDD的数据用你需要的方式连接起来。

根据上述思路，根本不会发生shuffle操作，从根本上杜绝了join操作可能导致的数据倾斜问题。

当join操作有数据倾斜问题并且其中一个RDD的数据量较小时，可以优先考虑这种方式，效果非常好。

map join的过程如下图所示：

![1639396166581](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/194928-798959.png)

map join过程

**2. 不适用场景分析：**

由于Spark的广播变量是在每个Executor中保存一个副本，如果两个RDD数据量都比较大，那么如果将一个数据量比较大的RDD做成广播变量，那么很有可能会造成内存溢出。

## 九、Spark性能优化

### Spark调优之RDD算子调优

### 1. RDD复用

在对RDD进行算子时，要避免相同的算子和计算逻辑之下对RDD进行重复的计算，如下图所示：

![1639396344028](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/195224-281715.png)

RDD的重复计算

对上图中的RDD计算架构进行修改，得到如下图所示的优化结果：

![1639396360580](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/195241-221777.png)

RDD架构优化

### 2. 尽早filter

获取到初始RDD后，应该考虑**尽早地过滤掉不需要的数据**，进而减少对内存的占用，从而提升Spark作业的运行效率。

### 3. 读取大量小文件-用wholeTextFiles

当我们将一个文本文件读取为 RDD 时，输入的每一行都会成为RDD的一个元素。

也可以将多个完整的文本文件一次性读取为一个pairRDD，其中键是文件名，值是文件内容。

```scala
val input:RDD[String] = sc.textFile("dir/*.log") 
```

如果传递目录，则将目录下的所有文件读取作为RDD。文件路径支持通配符。

但是这样对于大量的小文件读取效率并不高，应该使用 **wholeTextFiles**返回值为RDD[(String, String)]，其中Key是文件的名称，Value是文件的内容。

```scala
def wholeTextFiles(path: String, minPartitions: Int = defaultMinPartitions): RDD[(String, String)])
```

------

wholeTextFiles读取小文件:

```scala
val filesRDD: RDD[(String, String)] =
sc.wholeTextFiles("D:\\data\\files", minPartitions = 3)
val linesRDD: RDD[String] = filesRDD.flatMap(_._2.split("\\r\\n"))
val wordsRDD: RDD[String] = linesRDD.flatMap(_.split(" "))
wordsRDD.map((_, 1)).reduceByKey(_ + _).collect().foreach(println)
```

### 4. mapPartition和foreachPartition

- **mapPartitions**

map(_....)  表示每一个元素

mapPartitions(_....)  表示每个分区的数据组成的迭代器

普通的map算子对RDD中的每一个元素进行操作，而mapPartitions算子对RDD中每一个分区进行操作。

如果是普通的map算子，假设一个partition有1万条数据，那么map算子中的function要执行1万次，也就是对每个元素进行操作。

![1639396321891](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/195203-304611.png)

map 算子

如果是mapPartition算子，由于一个task处理一个RDD的partition，那么一个task只会执行一次function，function一次接收所有的partition数据，效率比较高。

![1639396304247](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/195204-120050.png)

mapPartition 算子

比如，**当要把RDD中的所有数据通过JDBC写入数据，如果使用map算子，那么需要对RDD中的每一个元素都创建一个数据库连接，这样对资源的消耗很大，如果使用mapPartitions算子，那么针对一个分区的数据，只需要建立一个数据库连接**。

mapPartitions算子也存在一些缺点：对于普通的map操作，一次处理一条数据，如果在处理了2000条数据后内存不足，那么可以将已经处理完的2000条数据从内存中垃圾回收掉；但是如果使用mapPartitions算子，但数据量非常大时，function一次处理一个分区的数据，如果一旦内存不足，此时无法回收内存，就可能会OOM，即内存溢出。

因此，**mapPartitions算子适用于数据量不是特别大的时候，此时使用mapPartitions算子对性能的提升效果还是不错的**。（当数据量很大的时候，一旦使用mapPartitions算子，就会直接OOM）

在项目中，应该首先估算一下RDD的数据量、每个partition的数据量，以及分配给每个Executor的内存资源，如果资源允许，可以考虑使用mapPartitions算子代替map。

- **foreachPartition**

rrd.foreache(_....) 表示每一个元素

rrd.forPartitions(_....)  表示每个分区的数据组成的迭代器

在生产环境中，通常使用foreachPartition算子来完成数据库的写入，通过foreachPartition算子的特性，可以优化写数据库的性能。

如果使用foreach算子完成数据库的操作，由于foreach算子是遍历RDD的每条数据，因此，每条数据都会建立一个数据库连接，这是对资源的极大浪费，因此，**对于写数据库操作，我们应当使用foreachPartition算子**。

与mapPartitions算子非常相似，foreachPartition是将RDD的每个分区作为遍历对象，一次处理一个分区的数据，也就是说，如果涉及数据库的相关操作，一个分区的数据只需要创建一次数据库连接，如下图所示：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)foreachPartition 算子

使用了foreachPartition 算子后，可以获得以下的性能提升：

1. 对于我们写的function函数，一次处理一整个分区的数据；
2. 对于一个分区内的数据，创建唯一的数据库连接；
3. 只需要向数据库发送一次SQL语句和多组参数；

**在生产环境中，全部都会使用foreachPartition算子完成数据库操作。foreachPartition算子存在一个问题，与mapPartitions算子类似，如果一个分区的数据量特别大，可能会造成OOM，即内存溢出**。

### 5. filter+coalesce/repartition(减少分区)

在Spark任务中我们经常会使用filter算子完成RDD中数据的过滤，在任务初始阶段，从各个分区中加载到的数据量是相近的，但是一旦进过filter过滤后，每个分区的数据量有可能会存在较大差异，

可以发现两个问题：

1. 每个partition的数据量变小了，如果还按照之前与partition相等的task个数去处理当前数据，有点浪费task的计算资源；
2. 每个partition的数据量不一样，会导致后面的每个task处理每个partition数据的时候，每个task要处理的数据量不同，这很有可能导致数据倾斜问题。

如上图所示，第二个分区的数据过滤后只剩100条，而第三个分区的数据过滤后剩下800条，在相同的处理逻辑下，第二个分区对应的task处理的数据量与第三个分区对应的task处理的数据量差距达到了8倍，这也会导致运行速度可能存在数倍的差距，这也就是**数据倾斜问题**。

针对上述的两个问题，我们分别进行分析：

1. 针对第一个问题，既然分区的数据量变小了，我们希望可以对分区数据进行重新分配，比如将原来4个分区的数据转化到2个分区中，这样只需要用后面的两个task进行处理即可，避免了资源的浪费。
2. 针对第二个问题，解决方法和第一个问题的解决方法非常相似，对分区数据重新分配，让每个partition中的数据量差不多，这就避免了数据倾斜问题。

那么具体应该如何实现上面的解决思路？我们需要coalesce算子。

**repartition与coalesce都可以用来进行重分区，其中repartition只是coalesce接口中shuffle为true的简易实现，coalesce默认情况下不进行shuffle，但是可以通过参数进行设置**。

假设我们希望将原本的分区个数A通过重新分区变为B，那么有以下几种情况：

1. A > B（多数分区合并为少数分区）

2. - A与B相差值不大

     此时使用coalesce即可，无需shuffle过程。

   - A与B相差值很大

     此时可以使用coalesce并且不启用shuffle过程，但是会导致合并过程性能低下，所以推荐设置coalesce的第二个参数为true，即启动shuffle过程。

3. A < B（少数分区分解为多数分区）

此时使用repartition即可，如果使用coalesce需要将shuffle设置为true，否则coalesce无效。

**我们可以在filter操作之后，使用coalesce算子针对每个partition的数据量各不相同的情况，压缩partition的数量，而且让每个partition的数据量尽量均匀紧凑，以便于后面的task进行计算操作，在某种程度上能够在一定程度上提升性能**。

注意：local模式是进程内模拟集群运行，已经对并行度和分区数量有了一定的内部优化，因此不用去设置并行度和分区数量。

### 6. 并行度设置

**Spark作业中的并行度指各个stage的task的数量**。

如果并行度设置不合理而导致并行度过低，会导致资源的极大浪费，例如，20个Executor，每个Executor分配3个CPU core，而Spark作业有40个task，这样每个Executor分配到的task个数是2个，这就使得每个Executor有一个CPU core空闲，导致资源的浪费。

理想的并行度设置，应该是让并行度与资源相匹配，简单来说就是在资源允许的前提下，并行度要设置的尽可能大，达到可以充分利用集群资源。合理的设置并行度，可以提升整个Spark作业的性能和运行速度。

Spark官方推荐，**task数量应该设置为Spark作业总CPU core数量的2~3倍**。之所以没有推荐task数量与CPU core总数相等，是因为task的执行时间不同，有的task执行速度快而有的task执行速度慢，如果task数量与CPU core总数相等，那么执行快的task执行完成后，会出现CPU core空闲的情况。如果task数量设置为CPU core总数的2~3倍，那么一个task执行完毕后，CPU core会立刻执行下一个task，降低了资源的浪费，同时提升了Spark作业运行的效率。

Spark作业并行度的设置如下：

```scala
val conf = new SparkConf().set("spark.default.parallelism", "500")
```

原则：**让 cpu 的 Core（cpu 核心数） 充分利用起来， 如有100个 Core,那么并行度可以设置为200~300**。

### 7. repartition/coalesce调节并行度

我们知道 Spark 中有并行度的调节策略，但是，**并行度的设置对于Spark SQL是不生效的，用户设置的并行度只对于Spark SQL以外的所有Spark的stage生效**。

Spark SQL的并行度不允许用户自己指定，Spark SQL自己会默认根据hive表对应的HDFS文件的split个数自动设置Spark SQL所在的那个stage的并行度，用户自己通 **spark.default.parallelism** 参数指定的并行度，只会在没Spark SQL的stage中生效。

由于Spark SQL所在stage的并行度无法手动设置，如果数据量较大，并且此stage中后续的transformation操作有着复杂的业务逻辑，而Spark SQL自动设置的task数量很少，这就意味着每个task要处理为数不少的数据量，然后还要执行非常复杂的处理逻辑，这就可能表现为第一个有Spark SQL的stage速度很慢，而后续的没有Spark SQL的stage运行速度非常快。

为了解决Spark SQL无法设置并行度和task数量的问题，我们可以使用repartition算子。

repartition 算子使用前后对比图如下：

![1639396286805](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/195127-191320.png)

repartition 算子使用前后对比图

**Spark SQL这一步的并行度和task数量肯定是没有办法去改变了，但是，对于Spark SQL查询出来的RDD，立即使用repartition算子，去重新进行分区，这样可以重新分区为多个partition，从repartition之后的RDD操作，由于不再涉及Spark SQL，因此stage的并行度就会等于你手动设置的值，这样就避免了Spark SQL所在的stage只能用少量的task去处理大量数据并执行复杂的算法逻辑。使用repartition算子的前后对比如上图所示**。

### 8. reduceByKey本地预聚合

**reduceByKey相较于普通的shuffle操作一个显著的特点就是会进行map端的本地聚合**，map端会先对本地的数据进行combine操作，然后将数据写入给下个stage的每个task创建的文件中，也就是在map端，对每一个key对应的value，执行reduceByKey算子函数。

reduceByKey算子的执行过程如下图所示：

![1639396267465](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/195108-668641.png)

reduceByKey 算子执行过程

使用reduceByKey对性能的提升如下：

1. 本地聚合后，在map端的数据量变少，减少了磁盘IO，也减少了对磁盘空间的占用；
2. 本地聚合后，下一个stage拉取的数据量变少，减少了网络传输的数据量；
3. 本地聚合后，在reduce端进行数据缓存的内存占用减少；
4. 本地聚合后，在reduce端进行聚合的数据量减少。

基于reduceByKey的本地聚合特征，我们应该考虑使用reduceByKey代替其他的shuffle算子，例如groupByKey。

groupByKey与reduceByKey的运行原理如下图1和图2所示：

![1639396246826](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/06/120610-12643.png)

图1：groupByKey原理

![1639396232173](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/13/195033-502606.png)

图2：reduceByKey原理

根据上图可知，groupByKey不会进行map端的聚合，而是将所有map端的数据shuffle到reduce端，然后在reduce端进行数据的聚合操作。由于reduceByKey有map端聚合的特性，使得网络传输的数据量减小，因此效率要明显高于groupByKey。

### 9. 使用持久化+checkpoint

Spark持久化在大部分情况下是没有问题的，但是有时数据可能会丢失，如果数据一旦丢失，就需要对丢失的数据重新进行计算，计算完后再缓存和使用，为了避免数据的丢失，可以选择对这个RDD进行checkpoint，也就是**将数据持久化一份到容错的文件系统上（比如HDFS）**。

一个RDD缓存并checkpoint后，如果一旦发现缓存丢失，就会优先查看checkpoint数据存不存在，如果有，就会使用checkpoint数据，而不用重新计算。也即是说，checkpoint可以视为cache的保障机制，如果cache失败，就使用checkpoint的数据。

使用checkpoint的**优点在于提高了Spark作业的可靠性，一旦缓存出现问题，不需要重新计算数据，缺点在于，checkpoint时需要将数据写入HDFS等文件系统，对性能的消耗较大**。

持久化设置如下：

```scala
sc.setCheckpointDir(‘HDFS’)
rdd.cache/persist(memory_and_disk)
rdd.checkpoint
```

### 10. 使用广播变量

默认情况下，task中的算子中如果使用了外部的变量，每个task都会获取一份变量的复本，这就造成了内存的极大消耗。一方面，如果后续对RDD进行持久化，可能就无法将RDD数据存入内存，只能写入磁盘，磁盘IO将会严重消耗性能；另一方面，task在创建对象的时候，也许会发现堆内存无法存放新创建的对象，这就会导致频繁的GC，GC会导致工作线程停止，进而导致Spark暂停工作一段时间，严重影响Spark性能。

假设当前任务配置了20个Executor，指定500个task，有一个20M的变量被所有task共用，此时会在500个task中产生500个副本，耗费集群10G的内存，如果使用了广播变量， 那么每个Executor保存一个副本，一共消耗400M内存，内存消耗减少了5倍。

广播变量在每个Executor保存一个副本，此Executor的所有task共用此广播变量，这让变量产生的副本数量大大减少。

在初始阶段，广播变量只在Driver中有一份副本。task在运行的时候，想要使用广播变量中的数据，此时首先会在自己本地的Executor对应的BlockManager中尝试获取变量，如果本地没有，BlockManager就会从Driver或者其他节点的BlockManager上远程拉取变量的复本，并由本地的**BlockManager**进行管理；之后此Executor的所有task都会直接从本地的BlockManager中获取变量。

对于多个Task可能会共用的数据可以广播到每个Executor上：

```scala
val 广播变量名= sc.broadcast(会被各个Task用到的变量,即需要广播的变量)

广播变量名.value//获取广播变量
```

### 11. 使用Kryo序列化

默认情况下，Spark使用Java的序列化机制。Java的序列化机制使用方便，不需要额外的配置，在算子中使用的变量实现Serializable接口即可，但是，Java序列化机制的效率不高，序列化速度慢并且序列化后的数据所占用的空间依然较大。

Spark官方宣称Kryo序列化机制比Java序列化机制**性能提高10倍左右**，Spark之所以没有默认使用Kryo作为序列化类库，是因为**它不支持所有对象的序列化**，同时Kryo需要用户在使用前注册需要序列化的类型，不够方便，**但从Spark 2.0.0版本开始，简单类型、简单类型数组、字符串类型的Shuffling RDDs 已经默认使用Kryo序列化方式了**。

Kryo序列化注册方式的代码如下：

```scala
public class MyKryoRegistrator implements KryoRegistrator{
  @Override
  public void registerClasses(Kryo kryo){
    kryo.register(StartupReportLogs.class);
  }
}
```

配置Kryo序列化方式的代码如下：

```scala
//创建SparkConf对象
val conf = new SparkConf().setMaster(…).setAppName(…)
//使用Kryo序列化库
conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer");  
//在Kryo序列化库中注册自定义的类集合
conf.set("spark.kryo.registrator", "bigdata.com.MyKryoRegistrator"); 
```

## Spark调优之Shuffle调优

### 1. map和reduce端缓冲区大小

在Spark任务运行过程中，如果shuffle的map端处理的数据量比较大，但是map端缓冲的大小是固定的，可能会出现map端缓冲数据频繁spill溢写到磁盘文件中的情况，使得性能非常低下，**通过调节map端缓冲的大小，可以避免频繁的磁盘IO操作，进而提升Spark任务的整体性能**。

map端缓冲的默认配置是32KB，如果每个task处理640KB的数据，那么会发生640/32 = 20次溢写，如果每个task处理64000KB的数据，即会发生64000/32=2000次溢写，这对于性能的影响是非常严重的。

map端缓冲的配置方法：

```scala
val conf = new SparkConf()
  .set("spark.shuffle.file.buffer", "64")
```

Spark Shuffle过程中，shuffle reduce task的buffer缓冲区大小决定了reduce task每次能够缓冲的数据量，也就是每次能够拉取的数据量，如果内存资源较为充足，适当增加拉取数据缓冲区的大小，可以减少拉取数据的次数，也就可以减少网络传输的次数，进而提升性能。

reduce端数据拉取缓冲区的大小可以通过`spark.reducer.maxSizeInFlight`参数进行设置，默认为48MB。该参数的设置方法如下：

reduce端数据拉取缓冲区配置：

```scala
val conf = new SparkConf()
  .set("spark.reducer.maxSizeInFlight", "96")
```

### 2. reduce端重试次数和等待时间间隔

Spark Shuffle过程中，reduce task拉取属于自己的数据时，如果因为网络异常等原因导致失败会自动进行重试。**对于那些包含了特别耗时的shuffle操作的作业，建议增加重试最大次数**（比如60次），以避免由于JVM的full gc或者网络不稳定等因素导致的数据拉取失败。在实践中发现，对于针对超大数据量（数十亿~上百亿）的shuffle过程，调节该参数可以大幅度提升稳定性。

reduce端拉取数据重试次数可以通过`spark.shuffle.io.maxRetries`参数进行设置，该参数就代表了可以重试的最大次数。如果在指定次数之内拉取还是没有成功，就可能会导致作业执行失败，默认为3，该参数的设置方法如下：

reduce端拉取数据重试次数配置：

```scala
val conf = new SparkConf()
  .set("spark.shuffle.io.maxRetries", "6")
```

Spark Shuffle过程中，reduce task拉取属于自己的数据时，如果因为网络异常等原因导致失败会自动进行重试，在一次失败后，会等待一定的时间间隔再进行重试，**可以通过加大间隔时长（比如60s），以增加shuffle操作的稳定性**。

reduce端拉取数据等待间隔可以通过`spark.shuffle.io.retryWait`参数进行设置，默认值为5s，该参数的设置方法如下：

reduce端拉取数据等待间隔配置：

```scala
val conf = new SparkConf()
  .set("spark.shuffle.io.retryWait", "60s")
```

### 3. bypass机制开启阈值

对于SortShuffleManager，如果shuffle reduce task的数量小于某一阈值则shuffle write过程中不会进行排序操作，而是直接按照未经优化的HashShuffleManager的方式去写数据，但是最后会将每个task产生的所有临时磁盘文件都合并成一个文件，并会创建单独的索引文件。

当你使用SortShuffleManager时，如果的确不需要排序操作，那么建议将这个参数调大一些，大于shuffle read task的数量，那么此时map-side就不会进行排序了，减少了排序的性能开销，但是这种方式下，依然会产生大量的磁盘文件，因此shuffle write性能有待提高。

SortShuffleManager排序操作阈值的设置可以通过`spark.shuffle.sort.bypassMergeThreshold`这一参数进行设置，默认值为200，该参数的设置方法如下：

reduce端拉取数据等待间隔配置：

```scala
val conf = new SparkConf()
  .set("spark.shuffle.sort.bypassMergeThreshold", "400")
```

## 十、故障排除

### 1. 避免OOM-out of memory

在Shuffle过程，reduce端task并不是等到map端task将其数据全部写入磁盘后再去拉取，而是map端写一点数据，reduce端task就会拉取一小部分数据，然后立即进行后面的聚合、算子函数的使用等操作。

reduce端task能够拉取多少数据，由reduce拉取数据的缓冲区buffer来决定，因为**拉取过来的数据都是先放在buffer中，然后再进行后续的处理，buffer的默认大小为48MB**。

reduce端task会一边拉取一边计算，不一定每次都会拉满48MB的数据，可能大多数时候拉取一部分数据就处理掉了。

虽然说增大reduce端缓冲区大小可以减少拉取次数，提升Shuffle性能，但是有时map端的数据量非常大，写出的速度非常快，此时reduce端的所有task在拉取的时候，有可能全部达到自己缓冲的最大极限值，即48MB，此时，再加上reduce端执行的聚合函数的代码，可能会创建大量的对象，这可能会导致内存溢出，即**OOM**。

**如果一旦出现reduce端内存溢出的问题，我们可以考虑减小reduce端拉取数据缓冲区的大小，例如减少为12MB**。

在实际生产环境中是出现过这种问题的，这是典型的**以性能换执行的原理**。reduce端拉取数据的缓冲区减小，不容易导致OOM，但是相应的，reudce端的拉取次数增加，造成更多的网络传输开销，造成性能的下降。

> 注意，要保证任务能够运行，再考虑性能的优化。

### 2. 避免GC导致的shuffle文件拉取失败

在Spark作业中，有时会出现`shuffle file not found`的错误，这是非常常见的一个报错，有时出现这种错误以后，选择重新执行一遍，就不再报出这种错误。

出现上述问题可能的原因**是Shuffle操作中，后面stage的task想要去上一个stage的task所在的Executor拉取数据，结果对方正在执行GC，执行GC会导致Executor内所有的工作现场全部停止**，比如BlockManager、基于netty的网络通信等，这就会导致后面的task拉取数据拉取了半天都没有拉取到，就会报出`shuffle file not found`的错误，而第二次再次执行就不会再出现这种错误。

可以通过调整reduce端拉取数据重试次数和reduce端拉取数据时间间隔这两个参数来对Shuffle性能进行调整，增大参数值，使得reduce端拉取数据的重试次数增加，并且每次失败后等待的时间间隔加长。

JVM GC导致的shuffle文件拉取失败调整数据重试次数和reduce端拉取数据时间间隔：

```scala
val conf = new SparkConf()
  .set("spark.shuffle.io.maxRetries", "6")
  .set("spark.shuffle.io.retryWait", "60s")
```

### 3. YARN-CLIENT模式导致的网卡流量激增问题

在YARN-client模式下，Driver启动在本地机器上，而Driver负责所有的任务调度，需要与YARN集群上的多个Executor进行频繁的通信。

假设有100个Executor，1000个task，那么每个Executor分配到10个task，之后，Driver要频繁地跟Executor上运行的1000个task进行通信，通信数据非常多，并且通信品类特别高。这就导致有可能在Spark任务运行过程中，由于频繁大量的网络通讯，本地机器的网卡流量会激增。

注意，**YARN-client模式只会在测试环境中使用，而之所以使用YARN-client模式，是由于可以看到详细全面的log信息**，通过查看log，可以锁定程序中存在的问题，避免在生产环境下发生故障。

**在生产环境下，使用的一定是YARN-cluster模式。在YARN-cluster模式下，就不会造成本地机器网卡流量激增问题**，如果YARN-cluster模式下存在网络通信的问题，需要运维团队进行解决。

### 4. YARN-CLUSTER模式的JVM栈内存溢出无法执行问题

当Spark作业中包含SparkSQL的内容时，可能会碰到YARN-client模式下可以运行，但是YARN-cluster模式下无法提交运行（报出OOM错误）的情况。

**YARN-client模式下**，Driver是运行在本地机器上的，Spark使用的JVM的PermGen的配置，是本地机器上的spark-class文件，JVM永久代的大小是128MB，这个是没有问题的，但是在**YARN-cluster模式下**，Driver运行在YARN集群的某个节点上，使用的是没有经过配置的默认设置，PermGen永久代大小为82MB。

SparkSQL的内部要进行很复杂的SQL的语义解析、语法树转换等等，非常复杂，如果sql语句本身就非常复杂，那么很有可能会导致性能的损耗和内存的占用，特别是对PermGen的占用会比较大。

所以，此时如果PermGen占用好过了82MB，但是又小于128MB，就会出现YARN-client模式下可以运行，YARN-cluster模式下无法运行的情况。

**解决上述问题的方法是增加PermGen(永久代)的容量**，需要在spark-submit脚本中对相关参数进行设置，设置方法如下：

```
--conf spark.driver.extraJavaOptions="-XX:PermSize=128M -XX:MaxPermSize=256M"
```

通过上述方法就设置了Driver永久代的大小，默认为128MB，最大256MB，这样就可以避免上面所说的问题。

### 5. 避免SparkSQL JVM栈内存溢出

当SparkSQL的sql语句有成百上千的or关键字时，就可能会出现Driver端的JVM栈内存溢出。

**JVM栈内存溢出基本上就是由于调用的方法层级过多，产生了大量的，非常深的，超出了JVM栈深度限制的递归**。（我们猜测SparkSQL有大量or语句的时候，在解析SQL时，例如转换为语法树或者进行执行计划的生成的时候，对于or的处理是递归，or非常多时，会发生大量的递归）

此时，**建议将一条sql语句拆分为多条sql语句来执行，每条sql语句尽量保证100个以内的子句**。根据实际的生产环境试验，一条sql语句的or关键字控制在100个以内，通常不会导致JVM栈内存溢出。

## 十一、Spark大厂面试真题

#### 1. 通常来说，Spark与MapReduce相比，Spark运行效率更高。请说明效率更高来源于Spark内置的哪些机制？

spark是借鉴了Mapreduce,并在其基础上发展起来的，继承了其分布式计算的优点并进行了改进，spark生态更为丰富，功能更为强大，性能更加适用范围广，mapreduce更简单，稳定性好。主要区别：

1. spark把运算的中间数据(shuffle阶段产生的数据)存放在内存，迭代计算效率更高，mapreduce的中间结果需要落地，保存到磁盘；
2. Spark容错性高，它通过弹性分布式数据集RDD来实现高效容错，RDD是一组分布式的存储在节点内存中的只读性的数据集，这些集合石弹性的，某一部分丢失或者出错，可以通过整个数据集的计算流程的血缘关系来实现重建，mapreduce的容错只能重新计算；
3. Spark更通用，提供了transformation和action这两大类的多功能api，另外还有流式处理sparkstreaming模块、图计算等等，mapreduce只提供了map和reduce两种操作，流计算及其他的模块支持比较缺乏；
4. Spark框架和生态更为复杂，有RDD，血缘lineage、执行时的有向无环图DAG，stage划分等，很多时候spark作业都需要根据不同业务场景的需要进行调优以达到性能要求，mapreduce框架及其生态相对较为简单，对性能的要求也相对较弱，运行较为稳定，适合长期后台运行；
5. Spark计算框架对内存的利用和运行的并行度比mapreduce高，Spark运行容器为executor，内部ThreadPool中线程运行一个Task，mapreduce在线程内部运行container，container容器分类为MapTask和ReduceTask。Spark程序运行并行度高；
6. Spark对于executor的优化，在JVM虚拟机的基础上对内存弹性利用：storage memory与Execution memory的弹性扩容，使得内存利用效率更高

#### 2. hadoop和spark使用场景？

Hadoop/MapReduce和Spark最适合的都是做离线型的数据分析，但Hadoop特别适合是单次分析的数据量“很大”的情景，而Spark则适用于数据量不是很大的情景。

1. 一般情况下，对于中小互联网和企业级的大数据应用而言，单次分析的数量都不会“很大”，因此可以优先考虑使用Spark。
2. 业务通常认为Spark更适用于机器学习之类的“迭代式”应用，80GB的压缩数据（解压后超过200GB），10个节点的集群规模，跑类似“sum+group-by”的应用，MapReduce花了5分钟，而spark只需要2分钟。

#### 3. spark如何保证宕机迅速恢复?

1. 适当增加spark standby master
2. 编写shell脚本，定期检测master状态，出现宕机后对master进行重启操作

#### 4. hadoop和spark的相同点和不同点？

**Hadoop底层使用MapReduce计算架构，只有map和reduce两种操作，表达能力比较欠缺，而且在MR过程中会重复的读写hdfs，造成大量的磁盘io读写操作**，所以适合高时延环境下批处理计算的应用；

**Spark是基于内存的分布式计算架构，提供更加丰富的数据集操作类型，主要分成转化操作和行动操作**，包括map、reduce、filter、flatmap、groupbykey、reducebykey、union和join等，数据分析更加快速，所以适合低时延环境下计算的应用；

**spark与hadoop最大的区别在于迭代式计算模型**。基于mapreduce框架的Hadoop主要分为map和reduce两个阶段，两个阶段完了就结束了，所以在一个job里面能做的处理很有限；spark计算模型是基于内存的迭代式计算模型，可以分为n个阶段，根据用户编写的RDD算子和程序，在处理完一个阶段后可以继续往下处理很多个阶段，而不只是两个阶段。所以spark相较于mapreduce，计算模型更加灵活，可以提供更强大的功能。

但是spark也有劣势，由于spark基于内存进行计算，虽然开发容易，但是真正面对大数据的时候，在没有进行调优的情况下，可能会出现各种各样的问题，比如OOM内存溢出等情况，导致spark程序可能无法运行起来，而mapreduce虽然运行缓慢，但是至少可以慢慢运行完。

#### 5. RDD持久化原理？

spark非常重要的一个功能特性就是可以将RDD持久化在内存中。

调用cache()和persist()方法即可。cache()和persist()的区别在于，cache()是persist()的一种简化方式，cache()的底层就是调用persist()的无参版本persist(MEMORY_ONLY)，将数据持久化到内存中。

如果需要从内存中清除缓存，可以使用unpersist()方法。RDD持久化是可以手动选择不同的策略的。在调用persist()时传入对应的StorageLevel即可。

#### 6. checkpoint检查点机制？

应用场景：当spark应用程序特别复杂，从初始的RDD开始到最后整个应用程序完成有很多的步骤，而且整个应用运行时间特别长，这种情况下就比较适合使用checkpoint功能。

原因：对于特别复杂的Spark应用，会出现某个反复使用的RDD，即使之前持久化过但由于节点的故障导致数据丢失了，没有容错机制，所以需要重新计算一次数据。

Checkpoint首先会调用SparkContext的setCheckPointDIR()方法，设置一个容错的文件系统的目录，比如说HDFS；然后对RDD调用checkpoint()方法。之后在RDD所处的job运行结束之后，会启动一个单独的job，来将checkpoint过的RDD数据写入之前设置的文件系统，进行高可用、容错的类持久化操作。

检查点机制是我们在spark  streaming中用来保障容错性的主要机制，它可以使spark streaming阶段性的把应用数据存储到诸如HDFS等可靠存储系统中，以供恢复时使用。具体来说基于以下两个目的服务：

1. 控制发生失败时需要重算的状态数。Spark streaming可以通过转化图的谱系图来重算状态，检查点机制则可以控制需要在转化图中回溯多远。
2. 提供驱动器程序容错。如果流计算应用中的驱动器程序崩溃了，你可以重启驱动器程序并让驱动器程序从检查点恢复，这样spark streaming就可以读取之前运行的程序处理数据的进度，并从那里继续。

#### 7. checkpoint和持久化机制的区别？

最主要的区别在于持久化只是将数据保存在BlockManager中，但是RDD的lineage(血缘关系，依赖关系)是不变的。但是checkpoint执行完之后，rdd已经没有之前所谓的依赖rdd了，而只有一个强行为其设置的checkpointRDD，checkpoint之后rdd的lineage就改变了。

持久化的数据丢失的可能性更大，因为节点的故障会导致磁盘、内存的数据丢失。但是checkpoint的数据通常是保存在高可用的文件系统中，比如HDFS中，所以数据丢失可能性比较低

#### 8. RDD机制理解吗？

rdd分布式弹性数据集，简单的理解成一种数据结构，是spark框架上的通用货币。所有算子都是基于rdd来执行的，不同的场景会有不同的rdd实现类，但是都可以进行互相转换。rdd执行过程中会形成dag图，然后形成lineage保证容错性等。从物理的角度来看rdd存储的是block和node之间的映射。

RDD是spark提供的核心抽象，全称为弹性分布式数据集。

RDD在逻辑上是一个hdfs文件，在抽象上是一种元素集合，包含了数据。它是被分区的，分为多个分区，每个分区分布在集群中的不同结点上，从而让RDD中的数据可以被并行操作（分布式数据集）

比如有个RDD有90W数据，3个partition，则每个分区上有30W数据。RDD通常通过Hadoop上的文件，即HDFS或者HIVE表来创建，还可以通过应用程序中的集合来创建；RDD最重要的特性就是容错性，可以自动从节点失败中恢复过来。即如果某个结点上的RDD partition因为节点故障，导致数据丢失，那么RDD可以通过自己的数据来源重新计算该partition。这一切对使用者都是透明的。

RDD的数据默认存放在内存中，但是当内存资源不足时，spark会自动将RDD数据写入磁盘。比如某结点内存只能处理20W数据，那么这20W数据就会放入内存中计算，剩下10W放到磁盘中。RDD的弹性体现在于RDD上自动进行内存和磁盘之间权衡和切换的机制。

#### 9. Spark streaming以及基本工作原理？

Spark  streaming是spark  core  API的一种扩展，可以用于进行大规模、高吞吐量、容错的实时数据流的处理。

它支持从多种数据源读取数据，比如Kafka、Flume、Twitter和TCP Socket，并且能够使用算子比如map、reduce、join和window等来处理数据，处理后的数据可以保存到文件系统、数据库等存储中。

Spark streaming内部的基本工作原理是：接受实时输入数据流，然后将数据拆分成batch，比如每收集一秒的数据封装成一个batch，然后将每个batch交给spark的计算引擎进行处理，最后会生产处一个结果数据流，其中的数据也是一个一个的batch组成的。

#### 10. DStream以及基本工作原理？

DStream是spark  streaming提供的一种高级抽象，代表了一个持续不断的数据流。

DStream可以通过输入数据源来创建，比如Kafka、flume等，也可以通过其他DStream的高阶函数来创建，比如map、reduce、join和window等。

DStream内部其实不断产生RDD，每个RDD包含了一个时间段的数据。

Spark  streaming一定是有一个输入的DStream接收数据，按照时间划分成一个一个的batch，并转化为一个RDD，RDD的数据是分散在各个子节点的partition中。

#### 11. spark有哪些组件？

1. master：管理集群和节点，不参与计算。
2. worker：计算节点，进程本身不参与计算，和master汇报。
3. Driver：运行程序的main方法，创建spark context对象。
4. spark context：控制整个application的生命周期，包括dagsheduler和task scheduler等组件。
5. client：用户提交程序的入口。

#### 12. spark工作机制？

用户在client端提交作业后，会由Driver运行main方法并创建spark context上下文。执行add算子，形成dag图输入dagscheduler，按照add之间的依赖关系划分stage输入task scheduler。task scheduler会将stage划分为task set分发到各个节点的executor中执行。

#### 13. 说下宽依赖和窄依赖

- 宽依赖：

  本质就是shuffle。父RDD的每一个partition中的数据，都可能会传输一部分到下一个子RDD的每一个partition中，此时会出现父RDD和子RDD的partition之间具有交互错综复杂的关系，这种情况就叫做两个RDD之间是宽依赖。

- 窄依赖：

  父RDD和子RDD的partition之间的对应关系是一对一的。

#### 14. Spark主备切换机制原理知道吗？

Master实际上可以配置两个，Spark原生的standalone模式是支持Master主备切换的。当Active Master节点挂掉以后，我们可以将Standby Master切换为Active Master。

Spark Master主备切换可以基于两种机制，一种是基于文件系统的，一种是基于ZooKeeper的。

基于文件系统的主备切换机制，需要在Active Master挂掉之后手动切换到Standby Master上；

而基于Zookeeper的主备切换机制，可以实现自动切换Master。

#### 15. spark解决了hadoop的哪些问题？

1. **MR**：抽象层次低，需要使用手工代码来完成程序编写，使用上难以上手；

   **Spark**：Spark采用RDD计算模型，简单容易上手。

2. **MR**：只提供map和reduce两个操作，表达能力欠缺；

   **Spark**：Spark采用更加丰富的算子模型，包括map、flatmap、groupbykey、reducebykey等；

3. **MR**：一个job只能包含map和reduce两个阶段，复杂的任务需要包含很多个job，这些job之间的管理以来需要开发者自己进行管理；

   **Spark**：Spark中一个job可以包含多个转换操作，在调度时可以生成多个stage，而且如果多个map操作的分区不变，是可以放在同一个task里面去执行；

4. **MR**：中间结果存放在hdfs中；

   **Spark**：Spark的中间结果一般存在内存中，只有当内存不够了，才会存入本地磁盘，而不是hdfs；

5. **MR**：只有等到所有的map task执行完毕后才能执行reduce task；

   **Spark**：Spark中分区相同的转换构成流水线在一个task中执行，分区不同的需要进行shuffle操作，被划分成不同的stage需要等待前面的stage执行完才能执行。

6. **MR**：只适合batch批处理，时延高，对于交互式处理和实时处理支持不够；

   **Spark**：Spark streaming可以将流拆成时间间隔的batch进行处理，实时计算。

#### 16. 数据倾斜的产生和解决办法？

数据倾斜以为着某一个或者某几个partition的数据特别大，导致这几个partition上的计算需要耗费相当长的时间。

在spark中同一个应用程序划分成多个stage，这些stage之间是串行执行的，而一个stage里面的多个task是可以并行执行，task数目由partition数目决定，如果一个partition的数目特别大，那么导致这个task执行时间很长，导致接下来的stage无法执行，从而导致整个job执行变慢。

避免数据倾斜，一般是要选用合适的key，或者自己定义相关的partitioner，通过加盐或者哈希值来拆分这些key，从而将这些数据分散到不同的partition去执行。

如下算子会导致shuffle操作，是导致数据倾斜可能发生的关键点所在：groupByKey；reduceByKey；aggregaByKey；join；cogroup；

#### 17. 你用sparksql处理的时候， 处理过程中用的dataframe还是直接写的sql？为什么？

这个问题的宗旨是问你spark sql 中dataframe和sql的区别，从执行原理、操作方便程度和自定义程度来分析 这个问题。

#### 18. RDD中reduceBykey与groupByKey哪个性能好，为什么

**reduceByKey**：reduceByKey会在结果发送至reducer之前会对每个mapper在本地进行merge，有点类似于在MapReduce中的combiner。这样做的好处在于，在map端进行一次reduce之后，数据量会大幅度减小，从而减小传输，保证reduce端能够更快的进行结果计算。

**groupByKey**：groupByKey会对每一个RDD中的value值进行聚合形成一个序列(Iterator)，此操作发生在reduce端，所以势必会将所有的数据通过网络进行传输，造成不必要的浪费。同时如果数据量十分大，可能还会造成OutOfMemoryError。

所以在进行大量数据的reduce操作时候建议使用reduceByKey。不仅可以提高速度，还可以防止使用groupByKey造成的内存溢出问题。

#### 19. Spark master HA主从切换过程不会影响到集群已有作业的运行，为什么

不会的。

因为程序在运行之前，已经申请过资源了，driver和Executors通讯，不需要和master进行通讯的。

#### 20. spark master使用zookeeper进行ha，有哪些源数据保存到Zookeeper里面

spark通过这个参数spark.deploy.zookeeper.dir指定master元数据在zookeeper中保存的位置，包括Worker，Driver和Application以及Executors。standby节点要从zk中，获得元数据信息，恢复集群运行状态，才能对外继续提供服务，作业提交资源申请等，在恢复前是不能接受请求的。

> 注：Master切换需要注意2点：
> 1、在Master切换的过程中，所有的已经在运行的程序皆正常运行！因为Spark Application在运行前就已经通过Cluster Manager获得了计算资源，所以在运行时Job本身的 调度和处理和Master是没有任何关系。
> 2、在Master的切换过程中唯一的影响是不能提交新的Job：一方面不能够提交新的应用程序给集群， 因为只有Active Master才能接受新的程序的提交请求；另外一方面，已经运行的程序中也不能够因 Action操作触发新的Job的提交请求。


