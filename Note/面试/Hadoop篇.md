

<!-- TOC -->

- [Hadoop篇](#hadoop篇)
- [Hdfs](#hdfs)
  - [什么是Hadoop,介绍以下Hadoop](#什么是hadoop介绍以下hadoop)
  - [介绍下Hadoop和Spark的差异](#介绍下hadoop和spark的差异)
  - [Hadoop常见的版本有哪些，分别有哪些特点，你一般是如何进行选择的?](#hadoop常见的版本有哪些分别有哪些特点你一般是如何进行选择的)
  - [单介绍Hadoop1.0，2.0，3.0的区别吗？](#单介绍hadoop102030的区别吗)
  - [mapreduce和hdfs是一体的吗，有什么关系](#mapreduce和hdfs是一体的吗有什么关系)
  - [简单介绍一下搭建Hadoop集群的流程](#简单介绍一下搭建hadoop集群的流程)
  - [hadoop的TextInputFormat作⽤是什么，如何⾃定义实现？](#hadoop的textinputformat作是什么如何定义实现)
  - [Hadoop可以自定义哪些输入类型](#hadoop可以自定义哪些输入类型)
  - [OutputFormat 数据输出](#outputformat-数据输出)
  - [HDFS组成架构](#hdfs组成架构)
  - [HDFS 中的 block 默认保存几份？](#hdfs-中的-block-默认保存几份)
  - [HDFS 默认 BlockSize 是多大？](#hdfs-默认-blocksize-是多大)
  - [数据切片与mapTask并行度决定机制](#数据切片与maptask并行度决定机制)
  - [如何判定一个job的map和reduce的数量?](#如何判定一个job的map和reduce的数量)
  - [文件大小设置，增大有什么影响？](#文件大小设置增大有什么影响)
  - [说下Hadoop常用的端口号](#说下hadoop常用的端口号)
  - [NameNode的作⽤，NameNode在启动的时候会做哪些操作？](#namenode的作namenode在启动的时候会做哪些操作)
  - [介绍一下HDFS读写流程](#介绍一下hdfs读写流程)
    - [**读数据流程**](#读数据流程)
    - [**写数据流程**](#写数据流程)
  - [hdfs如果再读取数据过程中，读取的数据错误怎么办](#hdfs如果再读取数据过程中读取的数据错误怎么办)
  - [HDFS 在上传文件的时候，如果其中一个 DataNode 突然挂掉了怎么办](#hdfs-在上传文件的时候如果其中一个-datanode-突然挂掉了怎么办)
  - [NameNode与SecondaryNameNode 的区别与联系？](#namenode与secondarynamenode-的区别与联系)
  - [为什么要引入secondary namenode](#为什么要引入secondary-namenode)
  - [secondary namenode工作机制](#secondary-namenode工作机制)
  - [Secondary NameNode 不能恢复NameNode 的全部数据，那如何保证NameNode 数据存储安全](#secondary-namenode-不能恢复namenode-的全部数据那如何保证namenode-数据存储安全)
  - [在NameNode HA 中，会出现脑裂问题吗？怎么解决脑裂](#在namenode-ha-中会出现脑裂问题吗怎么解决脑裂)
  - [DataNode工作机制](#datanode工作机制)
  - [掉线时限参数设置](#掉线时限参数设置)
  - [HAnamenode 是如何工作的?](#hanamenode-是如何工作的)
  - [HDFS的数据压缩算法](#hdfs的数据压缩算法)
- [MapReduce](#mapreduce)
  - [MapReduce](#mapreduce-1)
    - [**MapReduce工作机制**](#mapreduce工作机制)
    - [**过程**](#过程)
  - [MapReduce的优缺点](#mapreduce的优缺点)
  - [简单说⼀下hadoop的map-reduce编程模型](#简单说下hadoop的map-reduce编程模型)
  - [MapReduce编程规范](#mapreduce编程规范)
  - [mapreduce有几种join以及实现方法](#mapreduce有几种join以及实现方法)
    - [reduce side join](#reduce-side-join)
    - [map side join](#map-side-join)
    - [SemiJoin](#semijoin)
  - [MapReduce工作原理](#mapreduce工作原理)
  - [请说下MR 中 中Map Task 的工作机制](#请说下mr-中-中map-task-的工作机制)
  - [请说下MR 中Reduce Task 的工作机制](#请说下mr-中reduce-task-的工作机制)
  - [介绍一下MapReduce的Shuffle过程，并给出Hadoop优化的方案(包括：压缩、小文件、集群的优化)](#介绍一下mapreduce的shuffle过程并给出hadoop优化的方案包括压缩小文件集群的优化)
    - [**Map阶段**](#map阶段)
    - [**Reduce阶段**](#reduce阶段)
    - [**IO 传输**](#io-传输)
    - [**整体**](#整体)
    - [**文件压缩**](#文件压缩)
  - [请说下MR 中 中Shuffle 阶段](#请说下mr-中-中shuffle-阶段)
  - [在写MR 时，什么情况下可以使用规约](#在写mr-时什么情况下可以使用规约)
  - [ReduceTask 并行度决定机制](#reducetask-并行度决定机制)
  - [描述mapReduce有几种排序及排序发生的阶段](#描述mapreduce有几种排序及排序发生的阶段)
  - [描述mapReduce中shuffle阶段的工作流程，如何优化shuffle阶段](#描述mapreduce中shuffle阶段的工作流程如何优化shuffle阶段)
  - [mapreduce中的分区](#mapreduce中的分区)
  - [如果没有定义partitioner，那数据在被送达reducer前是如何被分区的？](#如果没有定义partitioner那数据在被送达reducer前是如何被分区的)
  - [哪些场景才能使⽤Combiner呢？](#哪些场景才能使combiner呢)
  - [MapReduce 2.0 容错性](#mapreduce-20-容错性)
  - [如何使用mapReduce实现两个表的join?](#如何使用mapreduce实现两个表的join)
  - [序列化](#序列化)
  - [hadoop小文件问题](#hadoop小文件问题)
    - [HDFS小文件影响](#hdfs小文件影响)
    - [如何解决小文件](#如何解决小文件)
      - [Hadoop Archive](#hadoop-archive)
      - [Sequence file](#sequence-file)
      - [ConbinFileInputFormat](#conbinfileinputformat)
      - [开启jvm重用](#开启jvm重用)
  - [MapTask & ReduceTask 源码解析](#maptask--reducetask-源码解析)
    - [MapTask 源码](#maptask-源码)
    - [ReduceTask 源码](#reducetask-源码)
- [Yarn](#yarn)
  - [为什么会产生 yarn,它解决了什么问题，有什么优势？](#为什么会产生-yarn它解决了什么问题有什么优势)
  - [Yarn架构](#yarn架构)
  - [YARN 集群的架构和工作原理知道多少](#yarn-集群的架构和工作原理知道多少)
  - [Yarn工作机制](#yarn工作机制)
  - [YARN 的任务提交流程是怎样的](#yarn-的任务提交流程是怎样的)
  - [介绍一下 Yarn 的 Job 提交流程](#介绍一下-yarn-的-job-提交流程)
  - [介绍下Yarn默认的调度器，调度器分类，以及它们之间的区别](#介绍下yarn默认的调度器调度器分类以及它们之间的区别)
  - [了解过哪些Hadoop的参数优化](#了解过哪些hadoop的参数优化)
  - [了解过Hadoop的基准测试吗?](#了解过hadoop的基准测试吗)
  - [你是怎么处理Hadoop宕机的问题的?](#你是怎么处理hadoop宕机的问题的)
  - [你是如何解决Hadoop数据倾斜的问题的，能举个例子吗?](#你是如何解决hadoop数据倾斜的问题的能举个例子吗)
  - [map-reduce程序运⾏的时候会有什么⽐较常见的问题？](#map-reduce程序运的时候会有什么较常见的问题)
  - [Hadoop性能调优？](#hadoop性能调优)
  - [MapReduce 2.0 容错性](#mapreduce-20-容错性-1)
  - [MapReduce推测执行算法及原理](#mapreduce推测执行算法及原理)
  - [优化](#优化)
    - [MapReduce跑得慢的原因？](#mapreduce跑得慢的原因)
    - [MapReduce优化方法](#mapreduce优化方法)
    - [数据倾斜](#数据倾斜)
    - [常用调优参数](#常用调优参数)
  - [MapReduce 开发总结](#mapreduce-开发总结)
  - [数据压缩](#数据压缩)
    - [MR 支持的压缩编码](#mr-支持的压缩编码)
    - [压缩方式的选择](#压缩方式的选择)
    - [压缩位置的选择](#压缩位置的选择)

<!-- /TOC -->

## Hadoop篇

## Hdfs

### 什么是Hadoop,介绍以下Hadoop

Hadoop是一个能够对大量数据进行分布式处理的计算软件框架。以一种**可靠、高效、可伸缩**的方式进行数据处理。主要包括三部分内容：Hdfs，MapReduce，Yarn。

Hadoop在广义上指一个生态圈，泛指大数据技术相关的开源组件或产品，如HBase，Hive，Spark，Zookeeper，Kafka，flume。

### 介绍下Hadoop和Spark的差异

![1633158114339](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/150155-838283.png)

> hadoop集群最主要的瓶紧在磁盘的IO。
>
> Hadoop集群的三种运行方式：Hadoop单机版、伪分布式模式、完全分布式模式

### Hadoop常见的版本有哪些，分别有哪些特点，你一般是如何进行选择的?

由于Hadoop的飞速发展，功能不断更新和完善，Hadoop的版本非常多，同时也显得杂乱。目前市面上，主流的是以下几个版本：

**Apache 社区版本**

Apache 社区版本 完全开源，免费，是非商业版本。Apache社区的Hadoop版本分支较多，而且部分Hadoop存在Bug。在选择Hadoop、Hbase、Hive等时，需要考虑兼容性。同时，这个版本的Hadoop的部署对Hadoop开发人员或运维人员的技术要求比较高。

**Cloudera版本**

Cloudera 版本 开源，免费，有商业版和非商业版本，是在Apache社区版本的Hadoop基础上，选择相对稳定版本的Hadoop，进行开发和维护的Hadoop版本。由于此版本的Hadoop在开发过程中对其他的框架的集成进行了大量的兼容性测试，因此使用者不必考虑Hadoop、Hbase、Hive等在使用过程中版本的兼容性问题，大大节省了使用者在调试兼容性方面的时间成本。

**Hortonworks版本**

Hortonworks 版本 的 Hadoop 开源、免费，有商业和非商业版本，其在 Apache 的基础上修改，对相关的组件或功能进行了二次开发，其中商业版本的功能是最强大，最齐全的。

所以基于以上特点进行选择，我们一般刚接触大数据用的就是CDH，在工作中大概率用 Apache 或者 Hortonworks。

### 单介绍Hadoop1.0，2.0，3.0的区别吗？

**Hadoop1.0由分布式存储系统HDFS和分布式计算框架MapReduce组成,切记没有Yarn调度器**，其中HDFS由一个NameNode和多个DateNode组成，MapReduce由一个JobTracker和多个TaskTracker组成。在Hadoop1.0中容易导致单点故障，拓展性差，性能低，支持编程模型单一的问题。

Hadoop2.0即为克服Hadoop1.0中的不足，提出了以下关键特性：

1. **Yarn**：它是Hadoop2.0引入的一个全新的通用资源管理系统，完全代替了Hadoop1.0中的JobTracker。在MRv1 中的 JobTracker 资源管理和作业跟踪的功能被抽象为 ResourceManager 和 AppMaster 两个组件。Yarn 还支持多种应用程序和框架，提供统一的资源调度和管理功能。
2. **NameNode 单点故障得以解决，**Hadoop2.2.0 同时解决了 NameNode 单点故障问题和内存受限问题，并提供 NFS，QJM 和 Zookeeper 三种可选的共享存储系统。
3. **HDFS 快照**：指 HDFS（或子系统）在某一时刻的只读镜像，该只读镜像对于防止数据误删、丢失等是非常重要的。例如，管理员可定时为重要文件或目录做快照，当发生了数据误删或者丢失的现象时，管理员可以将这个数据快照作为恢复数据的依据。
4. **支持Windows 操作系统**：Hadoop 2.2.0 版本的一个重大改进就是开始支持 Windows 操作系统
5. **Append：新版本的 Hadoop 引入了对文件的追加操作**

> 同时，新版本的Hadoop对于HDFS做了两个非常重要的**「增强」**，分别是支持异构的存储层次和通过数据节点为存储在HDFS中的数据提供内存缓冲功能（待了解）

相比于Hadoop2.0，Hadoop3.0 是直接基于 JDK1.8 发布的一个新版本，同时，Hadoop3.0引入了一些重要的功能和特性：

1. **HDFS可擦除编码**：这项技术使HDFS在不降低可靠性的前提下节省了很大一部分存储空间
2. **多NameNode支持**：在Hadoop3.0中，新增了对多NameNode的支持。当然，处于Active状态的NameNode实例必须只有一个。也就是说，从Hadoop3.0开始，在同一个集群中，支持一个 ActiveNameNode 和 多个 StandbyNameNode 的部署方式。
3. MR Native Task优化
4. Yarn基于cgroup 的内存和磁盘 I/O 隔离
5. Yarn container resizing

### mapreduce和hdfs是一体的吗，有什么关系

简单来说,MapReduce是一种变编程模型，是一种实现分布式计算的方法，我们可以通过这个模型，去实现分布式计算的功能，有了这个编程模型，对于我们程序员来讲，就不需要过于的关注分布式问题，比如网络，故障，一致性，资源的分配以及调度问题，反而可以让我们更加的关注我们的业务问题，所以有了这个模型，大大方便我们实现分布式程序。

hdfs简单来说是一个分布式的存储系统，是用来存储我们的数据的，存储再hdfs上面的数据都存在副本机制，这样可以保证我们的数据的安全性。这两者不是一体的，但是是不可分割的关系，通常我们会将数据存储再hdfs上面供我们的mapreduce程序分析使用。

### 简单介绍一下搭建Hadoop集群的流程

在正式搭建之前，我们需要准备以下6步：

**「准备工作」**

1. 关闭防火墙
2. 关闭SELINUX
3. 修改主机名
4. ssh无密码拷贝数据
5. 设置主机名和IP对应
6. jdk1.8安装

**「搭建工作:」**

- 下载并解压Hadoop的jar包
- 配置hadoop的核心文件
- 格式化namenode
- 启动....

### hadoop的TextInputFormat作⽤是什么，如何⾃定义实现？

InputFormat会在map操作之前对数据进⾏两⽅⾯的预处理

1. 是getSplits，返回的是InputSplit数组，对数据进⾏split分⽚，每⽚交给map操作⼀次
2. 是getRecordReader，返回的是RecordReader对象，对每个split分⽚进⾏转换为key-value键值对格式传递给map，常⽤的InputFormat是TextInputFormat，使⽤的是LineRecordReader对每个分⽚进⾏键值对的转换，以⾏偏移量作为键，
   ⾏内容作为值。
3. ⾃定义类继承InputFormat接⼜，重写createRecordReader和isSplitable⽅法 在createRecordReader中可以⾃定义分隔符

### Hadoop可以自定义哪些输入类型

![1633519780907](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/06/192943-955084.png)

抽象类FileInputFormat实现了InputFormat接口，是所有操作文件类型输入类的父类。InputFormat常见的接口实现类包括：

- TextInputFormat
- KeyValueTextInputFormat
- NLineInputFormat
- CombineTextInputFormat
- SequenceFileInputFormat
- DBInputFormat

**TextInputFormat**

TextInputFormat是默认的InputFormat。TextInputFormat提供了一个LineRecordReader，这个类会把输入文件的每一行作为值，每一行在文件中的字节偏移量为键。每条记录是一行输入，键是LongWritable类型，存储该行在整个文件中的字节偏移量，值是这行的内容，不包括任何行终止符（换行符和回车符）。

**KeyValueTextInputFormat**

每一行均为一条记录，被分隔符分割为key，value。可以通过在驱动类中设置conf.set(KeyValueLineRecordReader.KEY_VALUE_SEPERATOR, "\t");来设定分隔符。默认分隔符是tab（\t）。

**CombineTextInputFormat**

如果已经是大量小文件在 HDFS 中了，可以使用另一种 InputFormat 来做切片(CombineTextInputFormat)，它的切片逻辑跟 TextFileInputFormat 不同:它可以将多个小文件从逻辑上规划到一个切片中，这样，多个小文件就可以交给一个 maptask。经常使用在小文件过多的场景，可以把小文件逻辑上合并。

**NLineInputFormat**

如果使用NlineInputFormat，代表每个map进程处理的InputSplit不再按Block块去划分，而是按NlineInputFormat指定的行数N来划分。即输入文件的总行数/N=切片数，如果不整除，切片数=商+1。

**自定义步骤**

1. 重写isSplitable()方法，返回false不可切割
2. 重写createRecordReader()，创建自定义的RecordReader对象，并初始化
3. 改写RecordReader，实现一次读取一个完整文件封装为KV。

### OutputFormat 数据输出

OutputFormat是MapReduce输出的基类，所有实现MapReduce输出都实现了OutputFormat接口。下面我们介绍几种常见的OutputFormat实现类。

![1633742055827](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/09/091418-794458.png)

**文本输出TextOutputFormat**

默认的输出格式是TextOutputFormat，它把每条记录写为文本行。它的键和值可以是任意类型，因为TextOutputFormat调用toString()方法把它们转换为字符串。

**SequenceFileOutputFormat**

将SequenceFileOutputFormat输出作为后续 MapReduce任务的输入，这便是一种好的输出格式，因为它的格式紧凑，很容易被压缩。

**自定义OutputFormat**

1. 自定义一个类继承FileOutputFormat。
2. 改写RecordWriter，具体改写输出数据的方法write()。

### HDFS组成架构

![1633162793938](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/161955-691731.png)

架构主要由四个部分组成，分别为**HDFS Client、NameNode、DataNode和Secondary NameNode**。下面我们分别介绍这四个组成部分。

**Client：就是客户端。**

- 文件切分，文件上传HDFS的时候，Client将文件切分成一个一个的Block，然后进行存储；
- 与NameNode交互，获取文件的位置信息；
- 与DataNode交互，读取或者写入数据；
- Client提供一些命令来管理HDFS，比如启动或者关闭HDFS；
- Client可以通过一些命令来访问HDFS；

**NameNode：就是Master，它是一个主管、管理者。**

- 管理HDFS的名称空间；
- 管理数据块（Block）映射信息；
- 配置副本策略；
- 处理客户端读写请求。
- 主要用来记录文件的元数据信息（文件名，副本数，权限，文件块信息，位置）。
- **一个NameNode会存在单点故障**

**DataNode：就是Slave。NameNode下达命令，DataNode执行实际的操作。**

- 存储实际的数据块；
- 执行数据块的读/写操作。
- 和NameNode之间存在心跳信息。

**Secondary NameNode：并非NameNode的热备。当NameNode挂掉的时候，它并不能马上替换NameNode并提供服务。**

- 辅助NameNode，分担其工作量；
- 定期合并Fsimage和Edits，并推送给NameNode；
- 在紧急情况下，可辅助恢复NameNode。

> ResourceManager（JobTracker）：JobTracker负责调度DataNode上的工作。每个DataNode有一个TaskTracker，它们执行实际工作。
>
> NodeManager（TaskTracker）：负责执行具体的任务

### HDFS 中的 block 默认保存几份？

默认存储3分文件

### HDFS 默认 BlockSize 是多大？

**分块：**
HDFS存储系统中，引⼊了⽂件系统的分块概念（block），**块**是存储的最⼩单位，HDFS1.0定义其⼤⼩为64MB。与单磁盘⽂件系统相似，存储在 HDFS上的⽂件均存储为多个块，不同的是，如果某⽂件⼤⼩没有到达64MB，该⽂件也不会占据整个块空间。在分布式的HDFS集群上，Hadoop系统 保证⼀个块存储在⼀个datanode上。

**HDFS的namenode只存储整个⽂件系统的元数据镜像**，这个镜像由配置dfs.name.dir指定，datanode则存有⽂件的metainfo和具体的分块，存储路径由dfs.data.dir指定。

**分片：**
hadoop的作业在提交过程中，需要把具体的输⼊进⾏分⽚。具体的分⽚细节由**InputSplitFormat**指定。分⽚的规则为

FileInputFormat.class中的getSplits()⽅法指定：

```java
//具体分片使用的算法
long splitSize = computeSplitSize(goalSize, minSize, blockSize）
computeSplitSize:
Math.max(minSize, Math.min(goalSize, blockSize));
```

其中goalSize为“InputFile⼤⼩，”/“我们在配置⽂件中定义的mapred.map.tasks”值，minsize为mapred.min.split.size，blockSize为64，所以，这个算式为取分⽚⼤⼩不⼤于block，并且不⼩于在mapred.min.split.size配置中定义的最⼩Size。

当某个分块分成均等的若⼲分⽚时，会有最后⼀个分⽚⼤⼩⼩于定义的分⽚⼤⼩，则该分⽚独⽴成为⼀个分⽚。

**FileInputFormat源码解析(input.getSplits(job))**

1. 找到你数据存储的目录
2. 开始遍历处理（规划切片）目录下的每一个文件
3. 遍历第一个文件（假设为ss.txt）
   1. 获取文件大小fs.sizeOf(ss.txt);
   2. 计算切片大小`computeSliteSize(Math.max(minSize,Math.min(maxSize,blocksize)))=blocksize=128M`
   3. 默认情况下，切片大小=blocksize
   4. 开始切，形成第1个切片：ss.txt—0:128M 第2个切片ss.txt—128:256M 第3个切片ss.txt—256M:300M（每次切片时，都要判断切完剩下的部分是否大于块的1.1倍，不大于1.1倍就划分一块切片）
   5. 将切片信息写到一个切片规划文件中
   6. 整个切片的核心过程在getSplit()方法中完成。
   7. 数据切片只是在**逻辑**上对输入数据进行分片，并不会再磁盘上将其切分成分片进行存储。InputSplit只记录了分片的元数据信息，比如起始位置、长度以及所在的节点列表等。
   8. **注意：block是HDFS上物理上存储的存储的数据，切片是对数据逻辑上的划分。**
   9. 提交切片规划文件到yarn上，yarn上的MrAppMaster就可以根据切片规划文件计算开启maptask个数。

![20211208185455](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211208185455.png)

> 在hadoop1.x中默认是64M，在hadoop2.x中默认是128M。
> 
> 切片规划的配置文件上传到yarn上即可，因为切片是逻辑上的切片，并不会对物理数据执行真正的切片。

### 数据切片与mapTask并行度决定机制

1. 一个Job的**Map阶段并行度**由客户端在提交Job时的**切片数决定**。
2. 每一个Split切片分配一个MapTask并行实例处理
3. 默认情况下，切片大小=BlockSize(128m)
4. 切片时不考虑数据集整体，而是逐个针对每一个文件单独切片

**FileInputFormat 切片机制**

1. 简单地按照文件的内容长度进行切片
2. 切片大小，默认等于Block大小
3. 切片时不考虑数据集整体，而是逐个针对每一个文件单独切片

源码中计算切片大小的公式

```java
Math.max(minSize, Math.min(maxSize, blockSize));
mapreduce.input.fileinputformat.split.minsize=1 默认值为1
mapreduce.input.fileinputformat.split.maxsize= Long.MAXValue 默认值Long.MAXValue
因此，默认情况下，切片大小=blocksize。
```

切片大小设置

- maxsize（切片最大值）：参数如果调得比blockSize小，则会让切片变小，而且就等于配置的这个参数的值。
- minsize（切片最小值）：参数调的比blockSize大，则可以让切片变得比blockSize还大。

### 如何判定一个job的map和reduce的数量?

- map数量：splitSize=max{minSize,min{maxSize,blockSize}}，map的数量由切片数量决定。
  - map数量由处理的数据分成的block数量决定default_num = total_size / split_size;
- reduce数量
  - reduce的数量job.setNumReduceTasks(x);
  - x 为reduce的数量。不设置的话默认为 1。

### 文件大小设置，增大有什么影响？

HDFS中的文件在物理上是分块存储（block），块的大小可以通过配置参数( dfs.blocksize)来规定，默认大小在hadoop2.x版本中是128M，老版本中是64M。
**思考：为什么块的大小不能设置的太小，也不能设置的太大？**

- HDFS的块比磁盘的块大，**其目的是为了最小化寻址开销**。如果块设置得足够大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间。 因而，**传输一个由多个块组成的文件的时间取决于磁盘传输速率**。
- 如果寻址时间约为10ms，而传输速率为100MB/s，为了使寻址时间仅占传输时间的1%，我们要将块大小设置约为100MB。默认的块大小128MB。
- 块的大小：10ms×100×100M/s = 100M，如图

![1633162649884](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/161730-231582.png)

> 在企业中  一般128m（中小公司）   256m （大公司）

### 说下Hadoop常用的端口号

**Hadoop2.x**

- `dfs.namenode.http-address`:50070
- `dfs.datanode.http-address`:50075
- `SecondaryNameNode`辅助名称节点端口号：50090
- `dfs.datanode.address`:50010
- `fs.defaultFS`内部访问端口:8020 或者9000
- `yarn.resourcemanager.webapp.address`:8088
- 历史服务器web访问端口：19888

**Hadoop3.x**

- HDFS NameNode 内部通常端口(`fs.defaultFS`)：8020/9000/9820
- HDFS NameNode 对用户的查询端口（`dfs.namenode.http-address`）：9870
- Yarn查看任务运行情况的：8088
- 历史服务器：19888

> 常用的配置文件
>
> 3.x core-site.xml  hdfs-site.xml  yarn-site.xml  mapred-site.xml workers
> 2.x core-site.xml  hdfs-site.xml  yarn-site.xml  mapred-site.xml slaves

### NameNode的作⽤，NameNode在启动的时候会做哪些操作？

namenode总体来说是**管理和记录恢复**功能。⽐如管理datanode，保持⼼跳，如果超时则排除。对于上传⽂件都有镜像images和edits,这些可以⽤来恢复,总的来说是管理所有文件的元数据信息。

NameNode 数据存储在内存和本地磁盘，本地磁盘数据存储在fsimage 镜像文件 和edits 编辑日志文件 

**首次启动 NameNode：**

1. 格式化文件系统，为了生成fsimage 镜像文件 ；
2. 启动 NameNode：
  1. 读取 fsimage 文件，将文件内容加载进内存
  2. 等待 DataNade 注册与发送 block report
3. 启动 DataNode：
  1. 向 NameNode 注册
  2. 发送 block report
  3. 检查 fsimage 中记录的块的数量和 block report 中的块的总数是否相同
4. 对文件系统进行操作（创建目录，上传文件，删除文件等）：
  1. 此时内存中已经有文件系统改变的信息，但是磁盘中没有文件系统改变的信息，此时会将这些改变信息写入 edits 文件中，edits 文件中存储的是文件系统元数据改变的信息。

**第二次启动 NameNode：**

1. 读取 fsimage 和 edits 文件；
2. 将 fsimage 和 edits 文件合并成新的 fsimage 文件；
3. 创建新的 edits 文件，内容开始为空；
4. 启动 DataNode。

**NameNode启动的时候，会加载fsimage**，Fsimage加载过程完成的操作主要是为了：

1. 从fsimage中读取该HDFS中保存的每⼀个⽬录和每⼀个⽂件
2. 初始化每个⽬录和⽂件的元数据信息
3. 根据⽬录和⽂件的路径，构造出整个namespace在内存中的镜像
4. 如果是⽂件，则读取出该⽂件包含的所有blockid，并插⼊到BlocksMap中。

**启动过程如下**

![1633169830635](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/181711-861855.png)

如上图所⽰，namenode在加载fsimage过程其实⾮常简单，就是从fsimage中不停的顺序读取⽂件和⽬录的元数据信息，并在内存中构建整个namespace，同时将每个⽂件对应的blockid保存⼊BlocksMap中，此时BlocksMap中每个block对应的datanodes列表暂时为空。当fsimage加载完毕后，整个HDFS的⽬录结构在内存中就已经初始化完毕，所缺的就是每个⽂件对应的block对应的datanode列表信息。这些信息需要从datanode的blockReport中获取，所以加载fsimage完毕后，namenode进程进⼊rpc等待状态，等待所有的datanodes发送blockReports。

### 介绍一下HDFS读写流程

#### **读数据流程**

![1633159973884](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/153254-65470.png)

**hdsf读数据**

1. Client 向 NameNode 发送 RPC 请求。请求文件 block 的位置；
2. NameNode 收到请求之后会检查用户权限以及是否有这个文件，如果都符合，则会视情况返回部分或全部的 block 列表，对于每个 block，NameNode都会返回含有该block副本的DataNode地址；这些返回的DataNode地址，会按照集群拓扑结构得出 DataNode 与客户端的距离，然后进行排序， 排序两个规则 ：网络拓扑结构中距离 Client 近的排靠前；心跳机制中超时汇报的 DataNode 状态为 STALE，这样的排靠后；
3.  Client 选取排序靠前的 DataNode 来读取 block，如果客户端本身就是DataNode,那么将从本地直接获取数据( 短路读取特性 )；
4.  底层上本质是建立 Socket Strea（FSDataInputStream），重复的调用父类 DataInputStream 的 read 方法，直到这个块上的数据读取完毕；
5.  当读完列表的 block 后，若文件读取还没有结束，客户端会继续向NameNode 获取下一批的 block 列表；
6.  读取完一个 k block 都会进行 m checksum 验证 ，如果读取 DataNode 时出现错误，客户端会通知 NameNode，然后再从下一个拥有该 block 副本的DataNode 继续读；
7.  read 方法是并行的读取 k block 信息，不是一块一块的读取 ；NameNode 只是返回 Client 请求包含块的 DataNode 地址， 并不是返回请求块的数据 ；
8.  最终读取来所有的 block 会合并成一个完整的最终文件；

#### **写数据流程**

![1633240013210](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/03/134654-145286.png)

![1633160050454](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/153411-572238.png)

![1633169437806](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/181039-401280.png)

> HDFS 上传流程，举例说明⼀个256M的⽂件上传过程

1. 客户端通过Distributed FileSystem 模块向NameNode 请求上传文件，NameNode 检查目标文件是否已存在，父目录是否存在。
2. NameNode 返回是否可以上传。
3. 客户端请求第一个 Block 上传到哪几个DataNode 服务器上，客户端⾸先根据返回的信息先将⽂件分块(Hadoop2.X版本每⼀个block为 128M，⽽之前的版本为 64M);。
4. NameNode 返回3 个DataNode 节点，分别为dn1、dn2、dn3。
5. 客户端通过FSDataOutputStream 模块请求dn1 上传数据，dn1 收到请求会继续调用dn2，然后dn2 调用dn3，将这个通信管道建立完成。
6. dn1、dn2、dn3 逐级应答客户端。
7. 客户端开始往dn1 上传第一个Block（先从磁盘读取数据放到一个本地内存缓存），以Packet 为单位，dn1 收到一个Packet 就会传给dn2，dn2 传给dn3；dn1 每传一个packet会放入一个应答队列等待应答。
8. 当一个Block 传输完成之后，客户端再次请求NameNode 上传第二个Block 的服务器。（重复执行3-7 步）。

**hdfs写流程**

1. Client 客户端发送上传请求， 通过RPC与NameNode 建立通信 ，NameNode检查该用户是否有上传权限，以及上传的文件是否在 HDFS 对应的目录下重名，如果这两者有任意一个不满足，则直接报错，如果两者都满足，则返回给客户端一个可以上传的信息；
2. Client 根据文件的大小进行切分，默认 128M 一块，切分完成之后给NameNode 发送请求第一个 block 块上传到哪些服务器上；
3.  NameNode 收到请求之后，根据网络拓扑和**机架感知**以及**副本机制**进行文件分配，返回可用的 DataNode 的地址；
> 注：Hadoop 在设计时考虑到数据的**安全与高效**, 数据文件默认在HDFS 上存放三份, 存储策略为本地一份,同机架内其它某一节点上一份, 不同机架的某一节点上一份。

4. 客户端收到地址之后与服务器地址列表中的一个节点如 A 进行通信，本质上就是 RPC 调用，建立 pipeline，A 收到请求后会继续调用 B，B 在调用C，将整个 pipeline 建立完成，逐级返回 Client；
5. Client 开始向 A 上发送第一个 block（ 先从磁盘读取数据然后放到本地内存缓存）， 以packet（数据包，64kb ）为单位，A收到一个packet 就会发送给B ，然后B 发送给C ，A每传完一个packet 就会放入一个应答队列等待应答 ；
6.  数据被分割成一个个的 packet 数据包在 pipeline 上依次传输， 在pipeline 反向传输中，逐个发送 ack （命令正确应答） ，最终由 pipeline中第一个 DataNode 节点 A 将 pipelineack 发送给 Client；
7.  当一个block传输完成之后, Client再次请求NameNode上传第二个block，NameNode 重新选择三台 DataNode 给 Client。


> 存储文件副本的时候遵循机架感知策略：
>
> Hadoop3.1.3副本节点选择:
>
> 1. 第一个副本在Client所处的节点上。如果客户端在集群外，随机选一个。
> 2. 第二个副本在另一个机架的随机一个节点
> 3. 第三个副本在第二个副本所在机架的随机节点

![1633228735017](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/03/103856-113171.png)

### hdfs如果再读取数据过程中，读取的数据错误怎么办

客户端读取完 DataNode 上的块之后会进行 checksum 验证，也就是把客户端读取到本地的块与 HDFS 上的原始块进行校验，如果发现校验结果不一致，客户端会通知 NameNode，然后再从下一个拥有该block副本的DataNode 继续读 。

### HDFS 在上传文件的时候，如果其中一个 DataNode 突然挂掉了怎么办

客户端上传文件时与 DataNode 建立 pipeline 管道，管道的正方向是客户端向DataNode 发送的数据包，管道反向是DataNode 向客户端发送 ack 确认，也就是正确接收到数据包之后发送一个已确认接收到的应答。

当 DataNode 突然挂掉了，客户端接收不到这个 DataNode 发送的 ack 确认，客户端会通知 NameNode，NameNode 检查该块的副本与规定的不符，NameNode 会通知DataNode 去复制副本，并将挂掉的 DataNode 作下线处理，不再让它参与文件上传与下载。

### NameNode与SecondaryNameNode 的区别与联系？

**首先需要了解两个概念**


**Fsimage**

Fsimage文件是HDFS文件系统元数据的一个永久性检查点，其中包含HDFS文件系统的所有目录和文件inode的序列化信息。

**Edits文件**：

编辑日志文件，存放HDFS文件系统的所有更新操作的逻辑，文件系统客户端执行的所有写操作首先会记录大Edits文件中。

**Seen_txid**

文件保存是一个数字，就是最后一个edits_的数字。

**区别**

1. NameNode负责管理整个文件系统的元数据，以及每一个路径（文件）所对应的数据块信息。
2. SecondaryNameNode主要用于定期合并命名空间镜像和命名空间镜像的编辑日志。

**联系：**

1. SecondaryNameNode中保存了一份和namenode一致的镜像文件（fsimage）和编辑日志（edits）。
2. 在主namenode发生故障时（假设没有及时备份数据），可以从SecondaryNameNode恢复数据。

### 为什么要引入secondary namenode

**思考：NameNode 中的元数据是存储在哪里的？**

首先，我们做个假设，如果存储在NameNode 节点的磁盘中，因为经常需要进行随机访问，还有响应客户请求，必然是效率过低。因此，元数据需要存放在内存中。但如果只存在内存中，一旦断电，元数据丢失，整个集群就无法工作了。因此产生在磁盘中备份元数据的FsImage。

这样又会带来新的问题，当在内存中的元数据更新时，如果同时更新FsImage，就会导致效率过低，但如果不更新，就会发生一致性问题，一旦NameNode 节点断电，就会产生数据丢失。因此，引入Edits 文件（只进行追加操作，效率很高）。每当元数据有更新或者添加元数据时，修改内存中的元数据并追加到Edits 中。这样，一旦NameNode 节点断电，可以通过FsImage 和Edits 的合并，合成元数据。

但是，如果长时间添加数据到Edits 中，会导致该文件数据过大，效率降低，而且一旦断电，恢复元数据需要的时间过长。因此，需要定期进行FsImage 和Edits 的合并，如果这个操作由NameNode 节点完成，又会效率过低。因此，引入一个新的节点SecondaryNamenode，专门用于FsImage 和Edits 的合并。

### secondary namenode工作机制

![1633163160445](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/162600-21412.png)

**第一阶段：NameNode启动**

1. 第一次启动NameNode格式化后，创建fsimage和edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。
2. 客户端对元数据进行增删改的请求。
3. NameNode记录操作日志，更新滚动日志。
4. NameNode在内存中对数据进行增删改查。

**第二阶段：Secondary NameNode工作**

1. Secondary NameNode询问NameNode是否需要checkpoint。直接带回NameNode是否检查结果。
2. Secondary NameNode请求执行checkpoint。
3. NameNode滚动正在写的edits日志。
4. 将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode。
5. Secondary NameNode加载编辑日志和镜像文件到内存，并合并。
6. 生成新的镜像文件fsimage.chkpoint。
7. 拷贝fsimage.chkpoint到NameNode。
8. NameNode将fsimage.chkpoint重新命名成fsimage。

> CheckPoint时间设置：
>
> **通常情况下， SecondaryNameNode每隔一小时执行一次。**
>
> hdfs-default.xml
>
> <property> 
>
> <name> dfs.namenode.checkpoint.period</<name>
>
> </<value> 3600 s</value>//设置一小时执行一次
>
> </property >
>
> **一分钟检查一次操作数，当达到一百万次时， SecondaryNameNode执行一次**
>
> <property> 
>
> <name>dfs.namenode.checkpoint.txns</name>
> <value>1000000</value>
> <description>操作动作次数</description>
> </property>
> <property>
> <name>dfs.namenode.checkpoint.check.period</name>
> <value>60s</value>
> <description> 1 分钟检查一次操作次数</description>
> </property>

### Secondary NameNode 不能恢复NameNode 的全部数据，那如何保证NameNode 数据存储安全

这个问题就要说 NameNode 的高可用了，即NameNode HA 。一个 NameNode 有单点故障的问题，那就配置双 NameNode，配置有两个关键点，一是必须要保证这两个 NameNode 的元数据信息必须要同步的，二是一个NameNode 挂掉之后另一个要立马补上。

1. 元数据信息同步在HA 方案中采用的是 “ 共享存储 ” 。每次写文件时，需要将日志同步写入共享存储，这个步骤成功才能认定写文件成功。然后备份节点定期从共享存储同步日志，以便进行主备切换。
2. 监控 NameNode 状态采用 zookeeper，两个 NameNode 节点的状态存放在zookeeper 中，另外两个 NameNode 节点分别有一个进程监控程序，实施读取 zookeeper 中有 NameNode 的状态，来判断当前的 NameNode 是不是已经 down 机。如果 Standby 的 NameNode 节点的 ZKFC 发现主节点已经挂掉，那么就会强制给原本的 Active NameNode 节点发送强制关闭请求，之后将备用的 NameNode 设置为 Active。

### 在NameNode HA 中，会出现脑裂问题吗？怎么解决脑裂

假设 NameNode1 当前为 Active 状态，NameNode2 当前为 Standby 状态。如果某一时刻 NameNode1 对应的 ZKFailoverController 进程发生了“假死”现象，那么Zookeeper 服务端会认为 NameNode1 挂掉了，根据前面的主备切换逻辑，NameNode2会替代 NameNode1 进入 Active 状态。但是此时 NameNode1 可能仍然处于 Active状态正常运行，这样 NameNode1 和 NameNode2 都处于 Active 状态，都可以对外提供服务。这种情况称为脑裂。

脑裂对于 NameNode 这类对数据一致性要求非常高的系统来说是灾难性的，数据会发生错乱且无法恢复。zookeeper 社区对这种问题的解决方法叫做 fencing，中文翻译为隔离，也就是想办法把旧的 Active NameNode 隔离起来，使它不能正常对外提供服务。

在进行 fencing 的时候，会执行以下的操作：

1. 首先尝试调用这个旧 Active NameNode 的 HAServiceProtocol RPC 接口的 transitionToStandby 方法，看能不能把它转换为 Standby 状态。
2. 如果 transitionToStandby 方法调用失败，那么就执行 Hadoop 配置文件之中预定义的隔离措施，Hadoop 目前主要提供两种隔离措施，通常会选择 sshfence：

> sshfence：通过 SSH 登录到目标机器上，执行命令 fuser 将对应的进程杀死；
> shellfence：执行一个用户自定义的 shell 脚本来将对应的进程隔离。

### DataNode工作机制

![1633229449785](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/03/105051-961526.png)

1. 一个数据块在DataNode 上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。
2. DataNode 启动后向NameNode 注册，通过后，周期性（6 小时）的向NameNode 上报所有的块信息。
3. 心跳是每3 秒一次，心跳返回结果带有NameNode 给该DataNode 的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10 分钟没有收到某个DataNode 的心跳，则认为该节点不可用。
4. 集群运行中可以安全加入和退出一些机器。

### 掉线时限参数设置

![1633229638791](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/03/105359-282009.png)

> 需要注意的是hdfs-site.xml 配置文件中的heartbeat.recheck.interval 的单位为毫秒，dfs.heartbeat.interval 的单位为秒。

### HAnamenode 是如何工作的? 

![1633163094225](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/162454-597162.png)


在一个典型的HA集群中，每个NameNode是一立的服务器。在任一时刻，只有一个NameNode处于active状态，另一个处于standby状态。其中，active状态的NameNode负责所有的客户端操作，standby状态的NameNode处于从属地位，维护着数据状态，随时准备切换。

两个NameNode为了数据同步，会通过一组称作**JournalNodes的独立进程进行相互通信**。当active状态的NameNode的命名空间有任何修改时，会告知大部分的JournalNodes进程。standby状态的NameNode有能力读取JNs中的变更信息，并且一直监控edit log的变化，把变化应用于自己的命名空间。standby可以确保在集群出错时，命名空间状态已经完全同步。

为了确保快速切换，standby状态的NameNode有必要知道集群中所有数据块的位置。为了做到这点，所有的datanodes必须配置两个NameNode的地址，发送数据块位置信息和心跳给他们两个。

对于HA集群而言，确保同一时刻只有一个NameNode处于active状态是至关重要的。否则，两个NameNode的数据状态就会产生分歧，可能丢失数据，或者产生错误的结果。为了保证这点，JNs必须确保同一时刻只有一个NameNode可以向自己写数据。

**ZKFC**

ZKFC即ZKFailoverController，作为独立进程存在，负责控制NameNode的主备切换，ZKFC会监测NameNode的健康状况，当发现Active NameNode出现异常时会通过Zookeeper集群进行一次主备选举，完成Active和Standby状态的切换。

**HealthMonitor**

定时调用NameNode的HAServiceProtocol RPC接口(monitorHealth和getServiceStatus)，监控NameNode的健康状态并向ZKFC反馈。

**ActiveStandbyElector**

接收ZKFC的选举请求，通过Zookeeper自动完成主备选举，选举完成后回调ZKFC的主备切换方法对NameNode进行Active和Standby状态的切换。

**JouranlNode集群**

共享存储系统，负责存储HDFS的元数据，Active NameNode(写入)和Standby NameNode(读取)通过共享存储系统实现元数据同步，在主备切换过程中，新的Active NameNode必须确保元数据同步完成才能对外提供服务。

> NameNode的HA⼀个备⽤，⼀个⼯作，且⼀个失败后，另⼀个被激活。他们通过journal node来实现共享数据。

### HDFS的数据压缩算法

Hadoop中常用的压缩算法有bzip2、gzip、lzo、snappy，其中lzo、snappy需要操作系统安装native库才可以支持。

数据压缩的位置如下所示。

![1633677986721](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/08/152628-735757.png)

**MapReduce数据压缩解析**

**输入端采用压缩**

在有大量数据并计划重复处理的情况下，应该考虑对输入进行压缩。然而，你无须显示指定使用的编解码方式。Hadoop自动检查文件扩展名，如果扩展名能够匹配，就会用恰当的编解码方式对文件进行压缩和解压。否则，Hadoop就不会使用任何编解码器。

> 输入端的压缩需要支持数据的分片的压缩。

**mapper输出端采用压缩**

当map任务输出的中间数据量很大时，应考虑在此阶段采用压缩技术。这能显著改善内部数据Shuffle过程，而Shuffle过程在Hadoop处理过程中是资源消耗最多的环节。如果发现数据量大造成网络传输缓慢，应该考虑使用压缩技术。可用于压缩mapper输出的快速编解码器包括LZO或者Snappy。

> shuffle阶段的压缩更考虑压缩效率高的压缩算法。

**reducer输出采用压缩**

在此阶段启用压缩技术能够减少要存储的数据量，因此降低所需的磁盘空间。当mapreduce作业形成作业链条时，因为第二个作业的输入也已压缩，所以启用压缩同样有效。

> reduce如果直接输出文件到磁盘的时候，考虑压缩比高的算法，如果是mr的任务链，那就需要使用支持分片的压缩算法。

## MapReduce

### MapReduce

#### **MapReduce工作机制**

1. 分布式的运算程序往往需要分成至少2个阶段。
2. 第一个阶段的MapTask并发实例，完全并行运行，互不相干。
3. 第二个阶段的ReduceTask并发实例互不相干，但是他们的数据依赖于上一个阶段的所有MapTask并发实例的输出，map阶段输出的分区个数决定下一个阶段reduce的个数。
4. MapReduce编程模型只能包含一个Map阶段和一个Reduce阶段，如果用户的业务逻辑非常复杂，那就只能多个MapReduce程序，串行运行。

#### **过程**

1. InputFormat
   1. 默认的是TextInputformat  kv  key偏移量，v :一行内容
   2. 处理小文件CombineTextInputFormat 把多个文件合并到一起统一切片
2. Mapper 
   1. setup()初始化；  
   2. map()用户的业务逻辑； 
   3. clearup() 关闭资源；
3. 分区
   1. 默认分区HashPartitioner ，默认按照key的hash值%numreducetask个数
   2. 自定义分区
4. 排序
   1. 部分排序  每个输出的文件内部有序。
   2. 全排序：  一个reduce ,对所有数据大排序。
   3. 二次排序：  自定义排序范畴， 实现 writableCompare接口， 重写compareTo方法
   4. 流量倒序  按照上行流量 正序
5. Combiner 
   1. 前提：不影响最终的业务逻辑（求和 没问题   求平均值有问题）
   2. 提前聚合map  => 解决数据倾斜的一个方法
6. Reducer
   1. 用户的业务逻辑；
   2. setup()初始化；reduce()用户的业务逻辑； clearup() 关闭资源；
7. OutputFormat
   1. 默认TextOutputFormat  按行输出到文件
   2. 自定义

### MapReduce的优缺点

**优点**

1. MapReduce 易于编程
2. 良好的扩展性
3. 高容错性
4. 适合PB级以上海量数据的离线处理

**缺点**

1. 不擅长实时计算
2. 不擅长流式计算
3. 不擅长DAG（有向图）计算

### 简单说⼀下hadoop的map-reduce编程模型

1. map task会从本地⽂件系统读取数据，转换成key-value形式的键值对集合。使⽤的是hadoop内置的数据类型，⽐如longwritable、text等。
2. 将键值对集合输⼊mapper进⾏业务处理过程，将其转换成需要的key-value在输出之后会进⾏⼀个partition分区操作，默认使⽤的是hashpartitioner，可以通过重写hashpartitioner的getpartition⽅法来⾃定义分区规则。
3. 会对key进⾏进⾏sort排序，grouping分组操作将相同key的value合并分组输出，在这⾥可以使⽤⾃定义的数据类型，重写WritableComparator的Comparator⽅法来⾃定义排序规则，重写RawComparator的compara⽅法来⾃定义分组规则
4. 进⾏⼀个combiner归约操作，其实就是⼀个本地段的reduce预处理，以减⼩后⾯shufle和reducer的⼯作量,reduce task会通过⽹络将各个数据收集进⾏reduce处理，最后将数据保存或者显⽰，结束整个job。

### MapReduce编程规范

**用户编写的程序分成三个部分：Mapper，Reducer，Driver(提交运行mr程序的客户端)**

1. Mapper阶段
   1. 用户自定义的Mapper要继承自己的父类
   2. Mapper的输入数据是KV对的形式（KV的类型可自定义）
   3. Mapper中的业务逻辑写在map()方法中
   4. Mapper的输出数据是KV对的形式（KV的类型可自定义）
   5. map()方法（maptask进程）对每一个<K,V>调用一次
2. Reducer阶段
   1. 用户自定义的Reducer要继承自己的父类
   2. Reducer的输入数据类型对应Mapper的输出数据类型，也是KV
   3. Reducer的业务逻辑写在reduce()方法中
   4. Reducetask进程对每一组相同k的<k,v>组调用一次reduce()方法
3. Driver阶段

整个程序需要一个Drvier来进行提交，提交的是一个描述了各种必要信息的job对象

### mapreduce有几种join以及实现方法

#### reduce side join

reduce side join是所有join中用时最长的一种join，这种方法能够适用内连接、left外连接、right外连接、full外连接和反连接等所有的join方式。reduce side join不仅可以对小数据进行join，也可以对大数据进行join，**但是大数据会占用大量的集群内部网络IO，因为所有数据最终要写入到reduce端进行join**。如果要做join的数据量非常大的话，就不得不用reduce join了。

reduce side join是一种最简单的join方式，其主要思想如下： 

- 在map阶段，map函数同时读取两个文件File1和File2，为了区分两种来源的key/value数据对，对每条数据打一个标签（tag）,比如：tag=0表示来自文件File1，tag=2表示来自文件File2。即：map阶段的主要任务是对不同文件中的数据打标签,然后对数据进行按照数据的key进行分组处理。

- 在reduce阶段，reduce函数获取key相同的来自File1和File2文件的value list， 然后对于同一个key，对File1和File2中的数据进行join（笛卡尔乘积）。即：reduce阶段进行实际的连接操作。

> 内连接：如果带有标签‘A’和‘B’的数据都存在，遍历并连接这些数据，然后输出 。
> 
> 右边的数据如果存在就与左边连接，否则将右边的字段都赋null，只输出左边 。
> 
> 右外连接：与左外连接类似，左边为空就将左边赋值null，只输出右边 。
> 
> 全外连接：这个要相对复杂点，首先输出A和B都不为空的，然后输出某一边为空的 。
> 
> 输出A和B没有共同foreign key的值 。


#### map side join

之所以存在reduce side join，是因为在map阶段不能获取所有需要的join字段，即：同一个key对应的字段可能位于不同map中。

Reduce side join是非常低效的，因为shuffle阶段要进行大量的数据传输。 

Map side join是针对以下场景进行的优化：两个待连接表中，有一个表非常大，而另一个表非常小，以至于小表可以直接存放到内存中。这样，我们可以将小表复制多份，让每个map task内存中存在一份（比如存放到hash table中），然后只扫描大表：对于大表中的每一条记录key/value，在hash table中查找是否有相同的key的记录，如果有，则连接后输出即可。

- left outer join的左表必须是大表    
- right outer join的右表必须是大表
- inner join左表或右表均可以作为大表
- ull outer join不能使用mapjoin；

mapjoin支持小表为子查询，使用mapjoin时需要引用小表或是子查询时，需要引用别名；在mapjoin中，可以使用不等值连接或者使用or连接多个条件；    


#### SemiJoin

SemiJoin，也叫半连接，是从分布式数据库中借鉴过来的方法。它的产生动机是：对于reduce side join，跨机器的数据传输量非常大，这成了join操作的一个瓶颈，如果能够在map端过滤掉不会参加join操作的数据，则可以大大节省网络IO。 

实现方法很简单：选取一个小表，假设是File1，将其参与join的key抽取出来，保存到文件File3中，File3文件一般很小，可以放到 内存中。在map阶段，使用DistributedCache将File3复制到各个TaskTracker上，然后将File2中不在File3中的 key对应的记录过滤掉，剩下的reduce阶段的工作与reduce side join相同。

### MapReduce工作原理

**Map阶段**

![1633742632021](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/09/092353-81576.png)

**Read阶段**：Map Task通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。其中Read阶段可以有多重读取方式。

**Map阶段**：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。map()方法会调用多次，每读取一行数据，就会调用一次map方法。

**Collect收集阶段**：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。分区是根据key的hash()进行分区操作，可以自定义分区操作。

**Spill阶段**：即“溢写”，当环形缓冲区满后(默认是达到80%后开始发生溢写)，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序（也就是每次溢写出一个文件之前，都会对文件进行排序），并在必要时对数据进行合并、压缩等操作。在文件写出阶段会生成多个文件,每一个文件内部都存在多个分区的数据，多个分区之间是有序的，分区内部的数据是有序的。

1. 利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号Partition 进行排序，然后按照 key 进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照 key 有序。
2. 按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件output/spillN.out（N 表示当前溢写次数）中。如果用户设置了 Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。
3. 将分区数据的元信息写到内存索引数据结构 SpillRecord 中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过 1MB，则将内存索引写到文件 output/spillN.out.index 中。

在溢出本地文件之前，可能使用combine对分区数据进行局部的聚合操作。

**Merge阶段**：当所有数据处理完成后，MapTask 对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。

> 这里需要区分merge和combine的区别：
>
> combine:<a,1><a,1><b,1> ===><a,2><b,1>
>
> merge:<a,1><a,1><b,1>
>
> ​		<a,1><a,1><b,1>
>
> 上面两个文件merge到一起是：<a,1><a,1><a,1><a,1><b,1><b,1>

1. 当所有数据处理完后，MapTask 会将所有临时文件合并成一个大文件，并保存到文件output/file.out 中，同时生成相应的索引文件 output/file.out.index。
2. 在进行文件合并过程中，MapTask 以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并 mapreduce.task.io.sort.factor（默认 10）个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。
3. 让每个 MapTask 最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。

**Reduce阶段**

![1633670137899](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/08/131539-160421.png)

**Copy阶段**：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。

**Merge**:在copy阶段中，大的文件直接放到磁盘中，小的文件会加载到内存当中，最终还会使用merge归并排序将所有的文件归并为一个文件作为reduce的输入。

**Sort阶段**：在远程拷贝数据的同时，ReduceTask 启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。按照 MapReduce 语义，用户编写 reduce()函数输入数据是按 key 进行聚集的一组数据。为了将 key 相同的数据聚在一起，Hadoop 采用了基于排序的策略。由于各个 MapTask 已经实现对自己的处理结果进行了局部排序，因此，ReduceTask 只需对所有数据进行一次归并排序即可。

**Reduce阶段**：**某个键的所有键值对**都会被分发到同一个reduce操作中，确切的说，这个键和这个键所对应的所有值都会被传递给同一个Reducer。reduce过程的目的是将值的集合转换成一个值（例如求和或者求平均），或者转换成另一个集合。这个Reducer**最终会产生一个键值对**。

> （1）Shuffle 中的缓冲区大小会影响到 MapReduce 程序的执行效率，原则上说，缓冲区越大，磁盘 io 的次数越少，执行速度就越快。
> （2）缓冲区的大小可以通过参数调整，参数：mapreduce.task.io.sort.mb 默认 100M。

### 请说下MR 中 中Map Task 的工作机制

**简单过程**

inputFile 通过 split 被切割为多个 split 文件，通过 Record 按行读取内容给map（自己写的处理逻辑的方法） ，数据被 map 处理完之后交给 OutputCollect收集器，对其结果key进行分区（默认使用的hashPartitioner），然后写入buffer，每个map task 都有一个内存缓冲区 （环形缓冲区），存放着 map 的输出结果，当缓冲区快满的时候需要将缓冲区的数据以一个临时文件的方式溢写到磁盘，当
整个 map task 结束后再对磁盘中这个 maptask 产生的所有临时文件做合并，生成最终的正式输出文件，然后等待 reduce task 的拉取。

**详细过程**

1. 读取数据组件 InputFormat (默认 TextInputFormat) 会通过getSplits 方法对输入目录中的文件进行逻辑切片规划得到 block，有多少个 block 就对应启动多少个 MapTask。

2. 将输入文件切分为 block 之后，由 RecordReader 对象 (默认是LineRecordReader) 进行读取，以 \n 作为分隔符, 读取一行数据, 返回<key，value>， Key 表示每行首字符偏移值，Value 表示这一行文本内
    容。
3. 读取 block 返回 <key,value>, 进入用户自己继承的 Mapper 类中，执行用户重写的 map 函数，RecordReader 读取一行这里调用一次。
4. Mapper 逻辑结束之后，将 Mapper 的每条结果通过 context.write 进行collect 数据收集。在 collect 中，会先对其进行分区处理，默认使用HashPartitioner。也就是先把数据进行分区，数据以分区的形式放在一个文件中。
5. 接下来，会将数据写入内存，内存中这片区域叫做环形缓冲区( ( 默认 100M) ，是 缓冲区的作用是 集 批量收集 Mapper结果，减少磁盘  IO 的影响。我们的Key/Value 及 对以及Partition 的结果都会被写入缓冲区 。 当然 ， 写入之前 ， Key 与Value 值都会被序列化成字节数组 。
6. 当环形缓冲区的数据达到溢写比列(默认 0.8)，也就是 80M 时，溢写线程启动， 这 需要对这80MB 的 空间内的Key 序 做排序 (Sort) 。排序是 MapReduce模型默认的行为，这里的排序也是对序列化的字节做的排序，这里排序是先按照分区进行排序，然后再把分区中元素继续排序。
7. 合并溢写文件，每次溢写会在磁盘上生成一个临时文件 (写之前判断是否有 Combiner)，如果 Mapper 的输出结果真的很大，有多次这样的溢写发生，磁盘上相应的就会有多个临时文件存在。当整个数据处理结束之后开始对磁盘中的临时文件进行 Merge 合并，因为最终的文件只有一个写入磁盘，并且为这个文件提供了一个索引文件，以记录每个 reduce 对应数据的偏移量。

### 请说下MR 中Reduce Task 的工作机制

简单描述 ：
Reduce 大致分为 copy、sort、reduce 三个阶段，重点在前两个阶段。

copy 阶段包含一个 eventFetcher 来获取已完成的 map 列表，由 Fetcher 线程去 copy 数据，在此过程中会启动两个 merge 线程，分别为 inMemoryMerger和 onDiskMerger，分别将内存中的数据 merge 到磁盘和将磁盘中的数据进行merge。待数据 copy 完成之后，copy 阶段就完成了。开始进行 sort 阶段，sort 阶段主要是执行 finalMerge 操作，纯粹的 sort 阶段，完成之后就是 reduce 阶段，调用用户定义的 reduce 函数进行处理。

**详细步骤 ：**

1. Copy 阶段 ：简单地拉取数据。Reduce 进程启动一些数据 copy 线程(Fetcher)，通过 HTTP 方式请求 maptask 获取属于自己的文件（map task的分区会标识每个 map task 属于哪个 reduce task ，默认 reduce task的标识从 0 开始）。
2. Merge 阶段 ：在远程拷贝数据的同时，ReduceTask 启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。merge 有三种形式：内存到内存；内存到磁盘；磁盘到磁盘。默认情况下第一种形式不启用。当内存中的数据量到达一定阈值，就直接启动内存到磁盘的 merge。与 map 端类似，这也是溢写的过程，这个过程中如果你设置有 Combiner，也是会启用的，然后在磁盘中生成了众多的溢写文件。内存到磁盘的 merge 方式一直在运行，直到没有 map 端的数据时才结束，然后启动第三种磁盘到磁盘的 merge 方式生成最终的文件。
3. 合并排序 ：把分散的数据合并成一个大的数据后，还会再对合并后的数据排序。
4. 对排序后的键值对调用 reduce 方法 ：键相等的键值对调用一次 reduce 方法，每次调用会产生零个或者多个键值对，最后把这些输出的键值对写入到 HDFS 文件中。

### 介绍一下MapReduce的Shuffle过程，并给出Hadoop优化的方案(包括：压缩、小文件、集群的优化)

![1633740865646](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/09/085430-232977.png)

MapReduce数据读取并写入HDFS流程实际上是有10步：

![1633160204104](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/153645-130196.png)

那么到底什么是shuffle过程呢？

1. Map方法之后Reduce方法之前这段处理过程叫**「Shuffle」**
2. Map方法之后，数据首先进入到分区方法（partitioin()），把数据标记好分区，然后把数据发送到环形缓冲区；环形缓冲区默认大小100m，环形缓冲区达到80%时，进行溢写；溢写前对数据进行排序，排序按照对key的索引进行字典顺序排序，排序的手段**「快排」**；溢写产生大量溢写文件，需要对溢写文件进行**「归并排序」**；对溢写的文件也可以进行Combiner操作，前提是汇总操作，求平均值不行。最后将文件按照分区存储到磁盘，等待Reduce端拉取。
3. 每个Reduce拉取Map端对应分区的数据。拉取数据后先存储到内存中，内存不够了，再存储到磁盘。拉取完所有数据后，采用归并排序将内存和磁盘中的数据都进行排序。在进入Reduce方法前，可以对数据进行分组操作。

#### **Map阶段**

- 增大环形缓冲区大小。由100m扩大到200m。
- 增大环形缓冲区溢写的比例。由80%扩大到90%
- 减少对溢写文件的merge次数。（10个文件，一次20个merge）
- 不影响实际业务的前提下，采用Combiner提前合并，减少 I/O

#### **Reduce阶段**

- 合理设置Map和Reduce数：两个都不能设置太少，也不能设置太多。太少，会导致Task等待，延长处理时间；太多，会导致 Map、Reduce任务间竞争资源，造成处理超时等错误。
- 设置Map、Reduce共存：调整 `slowstart.completedmaps` 参数，使Map运行到一定程度后，Reduce也开始运行，减少Reduce的等待时间，也即是不必等待map任务执行完成后才执行reduce任务。
- 规避使用Reduce，因为Reduce在用于连接数据集的时候将会产生大量的网络消耗。
- 增加每个Reduce去Map中拿数据的并行数
- 集群性能可以的前提下，增大Reduce端存储数据内存的大小

#### **IO 传输**

- 采用数据压缩的方式，减少网络IO的的时间
- 使用SequenceFile二进制文件

#### **整体**

- MapTask默认内存大小为1G，可以增加MapTask内存大小为4G
- ReduceTask默认内存大小为1G，可以增加ReduceTask内存大小为4-5g
- 可以增加MapTask的cpu核数，增加ReduceTask的CPU核数
- 增加每个Container的CPU核数和内存大小
- 调整每个Map Task和Reduce Task最大重试次数

#### **文件压缩**

![1633160722552](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/14/200831-111343.png)

> 如果面试过程问起，我们一般回答压缩方式为Snappy，特点速度快，缺点无法切分（可以回答在链式MR中，Reduce端输出使用bzip2压缩，以便后续的map任务对数据进行split）

**行式存储和列式存储**

行式存储并不会破坏原来数据存储的行信息，行式存储对于条件查询，性能是比较差的，因为需要遍历表中所有数据。

![1633243363844](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/03/144244-269611.png)

对于列时存储，如果是条件查询，因为列上建立有索引，所以查询起来只会操作对应的列，磁盘io压力小很多。

![1633243494835](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/03/144456-160161.png)

### 请说下MR 中 中Shuffle 阶段

shuffle 阶段分为四个步骤：依次为：**分区，排序，规约，分组**，其中前三个步骤在 map 阶段完成，最后一个步骤在 reduce 阶段完成。

shuffle 是 Mapreduce 的核心，它分布在 Mapreduce 的 map 阶段和 reduce阶段。一般把从 Map 产生输出开始到 Reduce 取得数据作为输入之前的过程称作 shuffle。

1. Collect 阶段 ：将 MapTask 的结果输出到默认大小为 100M 的环形缓冲区，保存的是 key/value，Partition 分区信息等。
2. Spill 阶段 ：当内存中的数据量达到一定的阀值的时候，就会将数据写入本地磁盘，在将数据写入磁盘之前需要对数据进行一次排序的操作，如果配置了 combiner，还会将有相同分区号和 key 的数据进行排序。
3. MapTask 阶段的 Merge ：把所有溢出的临时文件进行一次合并操作，以确保一个 MapTask 最终只产生一个中间数据文件。
4. Copy 阶段 ：ReduceTask 启动 Fetcher 线程到已经完成 MapTask 的节点上复制一份属于自己的数据，这些数据默认会保存在内存的缓冲区中，当内存的缓冲区达到一定的阀值的时候，就会将数据写到磁盘之上。
5. ReduceTask 阶段的 Merge ：在 ReduceTask 远程复制数据的同时，会在后台开启两个线程对内存到本地的数据文件进行合并操作。
6. Sort 阶段 ：在对数据进行合并的同时，会进行排序操作，由于 MapTask 阶段已经对数据进行了局部的排序，ReduceTask 只需保证 Copy 的数据的最终整体有效性即可。Shuffle 中的缓冲区大小会影响到 mapreduce 程序的执行效率，原则上说，缓冲区越大，磁盘 io 的次数越少，执行速度就越快。缓冲区的大小可以通过参数调整, 参数： mapreduce.task.io.sort.mb 默认 100M.

### 在写MR 时，什么情况下可以使用规约

规约（combiner）是不能够影响任务的运行结果的局部汇总，适用于求和类，不适用于求平均值，如果 reduce 的输入参数类型和输出参数的类型是一样的，则规约的类可以使用 reduce 类，只需要在驱动类中指明规约的类即可。

### ReduceTask 并行度决定机制

回顾：MapTask 并行度由切片个数决定，切片个数由输入文件和切片规则决定。

思考：ReduceTask 并行度由谁决定？

1. 设置 ReduceTask 并行度（个数）：ReduceTask 的并行度同样影响整个 Job 的执行并发度和执行效率，但与 MapTask 的并发数由切片数决定不同，ReduceTask 数量的决定是可以直接手动设置：

```java
// 默认值是 1，手动设置为 4
job.setNumReduceTasks(4);
```

> **注意：**
>
> 1. ReduceTask=0，表示没有Reduce阶段，输出文件个数和Map个数一致。
>
> 2. ReduceTask默认值就是1，所以输出文件个数为一个。
>
> 3. 如果数据分布不均匀，就有可能在Reduce阶段产生数据倾斜
>
> 4. ReduceTask数量并不是任意设置，还要考虑业务逻辑需求，有些情况下，需要计算全局汇总结果，就只能有1个ReduceTask。
>
> 5. 具体多少个ReduceTask，需要根据集群性能而定。
>
> 6. 如果分区数不是1，但是ReduceTask为1，是否执行分区过程。
>
>    **答案是：不执行分区过程。因为在MapTask的源码中，执行分区的前提是先判断ReduceNum个数是否大于1。不大于1肯定不执行。**

### 描述mapReduce有几种排序及排序发生的阶段

对于MapTask，它会将处理的结果暂时放到环形缓冲区中，当环形缓冲区使用率达到一定阈值后，再对缓冲区中的数据进行一次快速排序，并将这些有序数据溢写到磁盘上，而当数据处理完毕后，它会对磁盘上所有文件进行归并排序，保证最终输出的只有一个文件。

对于ReduceTask，它从每个MapTask上远程拷贝相应的数据文件，如果文件大小超过一定阈值，则溢写磁盘上，否则存储在内存中。如果磁盘上文件数目达到一定阈值，则进行一次归并排序以生成一个更大文件；如果内存中文件大小或者数目超过一定阈值，则进行一次合并后将数据溢写到磁盘上。当所有数据拷贝完毕后，ReduceTask统一对内存和磁盘上的所有数据进行一次归并排序。

排序的分类

1. 部分排序：MapReduce根据输入记录的键对数据集排序。保证输出的每个文件内部排序。
2. 全排序：如何用Hadoop产生一个全局排序的文件？最简单的方法是使用一个分区。但该方法在处理大型文件时效率极低，因为一台机器必须处理所有输出文件，从而完全丧失了MapReduce所提供的并行架构。
   1. 替代方案：首先创建一系列排好序的文件；其次，串联这些文件；最后，生成一个全局排序的文件。主要思路是使用一个分区来描述输出的全局排序。例如：可以为待分析文件创建3个分区，在第一分区中，记录的单词首字母a-g，第二分区记录单词首字母h-n, 第三分区记录单词首字母o-z。
3. 辅助排序（GroupingComparator分组）：Mapreduce框架在记录到达reducer之前按键对记录排序，但键所对应的值并没有被排序。甚至在不同的执行轮次中，这些值的排序也不固定，因为它们来自不同的map任务且这些map任务在不同轮次中完成时间各不相同。一般来说，大多数MapReduce程序会避免让reduce函数依赖于值的排序。但是，有时也需要通过特定的方法对键进行排序和分组等以实现对值的排序。
4. 二次排序：在自定义排序过程中，如果compareTo中的判断条件为两个即为二次排序。

**自定义排序WritableComparable**

bean对象实现WritableComparable接口重写compareTo方法，就可以实现排序

```JAVA
@Override
public int compareTo(FlowBean o) {
// 倒序排列，从大到小
return this.sumFlow > o.getSumFlow() ? -1 : 1;
}
```

**排序发生的阶段**

1. 一个是在map side发生在spill后partition前。使用的是快速排序，对每一个溢出文件中的分区进行排序操作，然后在对分区内部的数据进行排序。在map端对溢出的多个本地文件进行排序的时候，也会使用到归并排序，因为再map端最终输出的是一个文件
2. 一个是在reduce side发生在copy后 reduce前。使用的是归并排序，对来自多个map端的相同key的数据进行归并排序，形成一个大文件。

### 描述mapReduce中shuffle阶段的工作流程，如何优化shuffle阶段

![1633676431270](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/08/150032-903341.png)

![1633740888066](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/09/085450-285051.png)

mr程序最大的瓶紧就在于磁盘的io，所以主要从下面几个方面进行优化：

**分区，排序，溢写，拷贝到对应reduce机器上，增加combiner，压缩溢写的文件。**

### mapreduce中的分区

默认的Partition 分区

![1633741014103](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/09/085656-914629.png)

**分区小结**

1. 如果ReduceTask的数量> getPartition的结果数，则会多产生几个空的输出文件part-r-000xx；
2. 如果1<ReduceTask的数量<getPartition的结果数，则有一部分分区数据无处安放，会Exception；
3. 如果ReduceTask的数量=1，则不管MapTask端输出多少个分区文件，最终结果都交给这一个ReduceTask，最终也就只会产生一个结果文件 part-r-00000；
4. 分区号必须从零开始，逐一累加。

### 如果没有定义partitioner，那数据在被送达reducer前是如何被分区的？

如果没有自定义的partitioning，则默认的partition算法，即根据每一条数据的key的hashcode值模运算（%）reduce的数量，得到的数字就是“分区号“。

### 哪些场景才能使⽤Combiner呢？

1. Combiner是MR程序中Mapper和Reducer之外的一种组件。
2. Combiner组件的父类就是Reducer。
3. Combiner和Reducer的区别在于运行的位置
   1. Combiner是在每一个MapTask所在的节点运行;
   2. Reducer是接收全局所有Mapper的输出结果；
4. Combiner的意义就是对每一个MapTask的输出进行局部汇总，以减小网络传输量。
5. Combiner能够应用的前提是不能影响最终的业务逻辑，而且，Combiner的输出kv应该跟Reducer的输入kv类型要对应起来。
6. Combiner的输出是Reducer的输⼊，Combiner绝不能改变最终的计算结果（比如求平均值）。所以从我的想法来看，Combiner只应该⽤于那种Reduce的输⼊key/value与输出key/value类型完全⼀致，且不影响最终结果的场景。⽐如累加，最⼤值等。Combiner的使⽤⼀定得慎重，如果⽤好，它对job执⾏效率有帮助，反之会影响reduce的最终结果。

**Combiner和reducer的区别在于运行的位置。**

- Combiner是在每一个maptask所在的节点运行；
- Reducer是接收全局所有Mapper的输出结果。

### MapReduce 2.0 容错性

**MRAppMaster容错性**

一旦运行失败，由YARN的ResourceManager负责重新启动，最多重启次数可由用户设置，默认是2次。一旦超过最高重启次数，则作业运行失败。

**Map Task/Reduce**

Task周期性向MRAppMaster汇报心跳；一旦Task 挂掉，则MRAppMaster将为之重新申请资源，并运行之。最多重新运行次数可由用户设置，默认4 次。

### 如何使用mapReduce实现两个表的join?

**reduce side join：**
在map阶段，map函数同时读取两个文件File1和File2，为了区分两种来源的key/value数据对，对每条数据打一个标签（tag）,比如：tag=0 表示来自文件File1，tag=2 表示来自文件File2。

**Map side join：**
Map side join是针对以下场景进行的优化：两个待连接表中，有一个表非常大，而另一个表非常小，以至于小表可以直接存放到内存中。这样，我们可以将小表复制多份，让每个map task 内存中存在一份（比如存放到hash table 中），然后只扫描大表：对于大表中的每一条记录key/value，在hash table中查找是否有相同的key 的记录，如果有，则连接后输出即可。

### 序列化

序列化就是把内存中的对象，转换成字节序列（或其他数据传输协议）以便于存储（持久化）和网络传输。 

反序列化就是将收到字节序列（或其他数据传输协议）或者是硬盘的持久化数据，转换成内存中的对象。

Java的序列化是一个重量级（实现的功能比较多）序列化框架（Serializable），一个对象被序列化后，会附带很多额外的信息（各种校验信息，header，继承体系等），不便于在网络中高效传输。所以，hadoop自己开发了一套序列化机制（Writable），精简、高效。

- 为什么要序列化？

  一般来说，“活的”对象只生存在内存里，关机断电就没有了。而且“活的”对象只能由本地的进程使用，不能被发送到网络上的另外一台计算机。 然而序列化可以存储“活的”对象，可以将“活的”对象发送到远程计算机。

Hadoop序列化的特点

1. 紧凑，高效使用存储空间。
2. 快速，读写数据额外开销小。
3. 可扩展，随着通信协议的升级可以升级。
4. 互操作，支持多种语言交互。

**如何实现自定义对象序列化**

自定义bean对象要想序列化传输，必须实现序列化接口，需要注意以下6项。

1. 必须实现Writable接口
2. 反序列化时，需要反射调用空参构造函数，所以必须有**空参构造函数**
3. 重写序列化和反序列化方法
4. 注意反序列化的顺序和序列化的顺序完全一致，序列化的管道可以想象为一个队列。
5. 要想把结果显示在文件中，需要重写toString()，且用”\t”分开，方便后续用
6. 如果需要将自定义的bean放在key中传输，则还需要实现comparable接口，因为mapreduce框中的shuffle过程一定会对key进行排序

> 之所以需要序列化，就是因为map阶段和reducer阶段在不同的服务器中，所以中间需要传输bean对象，所以需要先将对象进行序列化操作。

### hadoop小文件问题

#### HDFS小文件影响

- **影响NameNode的寿命**，因为文件元数据存储在NameNode的内存中,每一个文件占据namenode的存储空间是150字节，这样的话不管是大文件还是小文件，都需要占150字节，浪费存储空间。
- **影响计算引擎的任务数量**，比如每个小的文件都会生成一个Map任务，也就是说在启动map任务的时候，hadoop会针对单独文件进行划分并且启动map任务，这样的话就会启动很多map任务，非常影响集群的性能。
- **小文件多的话非常影响性能**，会导致一台namenode宕机后，另一台namenode长时间无法启动。
- **磁盘上面存放过多的小文件，磁盘io速度很慢**，所以hadoop集群在随机从磁盘上面读取小文件的话非常慢，定位每一个文件会花费大量时间，但是读取文件只需要一小段时间。
- **每一个小文件开启一个map和reduce**，那么每一次开启和关闭erduce会花费大量的时间。
- **集群中的资源有限**，所以大部分任务是处于等待资源的状态，拖慢系统的运行速度。
- HDFS上每个文件都要在namenode上建立一个索引，这个索引的大小约为150byte，这样当小文件比较多的时候，就会产生很多的索引文件，一方面会大量占用namenode的内存空间，另一方面就是索引文件过大是的索引速度变慢。

![1633241428595](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/08/085445-234620.png)

#### 如何解决小文件

##### Hadoop Archive

Hadoop Archive或者HAR，是一个高效地将小文件放入HDFS块中的文件存档工具，它能够将多个小文件打包成一个HAR文件，这样在减少namenode内存使用的同时，仍然允许对文件进行透明的访问。  

##### Sequence file

Sequence file： sequence file由一系列的二进制key/value组成，如果为key小文件名，value为文件内容，则可以将大批小文件合并成一个大文件。

##### ConbinFileInputFormat

采用ConbinFileInputFormat来作为输入，解决输入端大量小文件场景，把多个小文件逻辑上划分到一个大文件中。

##### 开启jvm重用

- 对于大量小文件Job，可以开启JVM重用。
  - JVM重用原理：一个Map运行在一个JVM上，开启重用的话，该Map在JVM上运行完毕后，JVM继续运行其他Map。
  - 具体设置：mapreduce.job.jvm.numtasks值在10-20之间。

  再spark中就没有jvm重用，因为spark是线程级别调度，而mapreduce是进程级别调度。

> 为什么需要开启jvm重用，因为mapreduce是进程级别的调度。

### MapTask & ReduceTask 源码解析

#### MapTask 源码

![1633743947700](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/09/094549-647470.png)

#### ReduceTask 源码

![1633743986625](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/09/094643-670040.png)

![1633744008207](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/09/094651-523443.png)

## Yarn

### 为什么会产生 yarn,它解决了什么问题，有什么优势？

1. Yarn最主要的功能就是解决运行的用户程序与yarn框架完全解耦。
2. Yarn上可以运行各种类型的分布式运算程序（mapreduce只是其中的一种），比如mapreduce、storm程序，spark程序……

### Yarn架构

![1633929954257](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/132557-163506.png)

### YARN 集群的架构和工作原理知道多少

YARN 的基本设计思想是将 MapReduce V1 中的 JobTracker 拆分为两个独立的服务：ResourceManager 和 ApplicationMaster。

ResourceManager 负责整个系统的资源管理和分配，ApplicationMaster 负责单个应用程序的的管理。

**ResourceManager ：** 

RM 是一个全局的资源管理器，负责整个系统的资源管理和分配，它主要由两个部分组成：调度器（Scheduler）和应用程序管理器（ApplicationManager）。

调度器根据容量、队列等限制条件，将系统中的资源分配给正在运行的应用程序，在保证容量、公平性和服务等级的前提下，优化集群资源利用率，让所有的资源都被充分利用应用程序管理器负责管理整个系统中的所有的应用程序，包括应用程序的提交、与调度器协商资源以启动 ApplicationMaster、监控
ApplicationMaster 运行状态并在失败时重启它。

**ApplicationMaster ：** 

用户提交的一个应用程序会对应于一个ApplicationMaster，它的主要功能有：

- 与 RM 调度器协商以获得资源，资源以 Container 表示。
- 将得到的任务进一步分配给内部的任务。
- 与 NM 通信以启动/停止任务。
- 监控所有的内部任务状态，并在任务运行失败的时候重新为任务申请资源以重启任务。

**NodeManager ：** 

NodeManager 是每个节点上的资源和任务管理器，一方面，它会定期地向 RM 汇报本节点上的资源使用情况和各个 Container 的运行状态；另一方面，他接收并处理来自 AM 的 Container 启动和停止请求。

**Container ：**

Container 是 YARN 中的资源抽象，封装了各种资源。 一个应用程序会分配一个 Container ，这个应用程序只能使用这个 r Container 中描述的资源 。不同于 MapReduceV1 中槽位 slot 的资源封装，Container 是一个动态资源的划分单位，更能充分利用资源。

### Yarn工作机制

![1633929992509](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/132634-903722.png)

1. MR程序提交到客户端所在的节点。
2. YarnRunner向ResourceManager申请一个Application。
3. RM将该应用程序的资源路径返回给YarnRunner。
4. 该程序将运行所需资源提交到HDFS上。
5. 程序资源提交完毕后，申请运行mrAppMaster。
6. RM将用户的请求初始化成一个Task。
7. 其中一个NodeManager领取到Task任务。
8. 该NodeManager创建容器Container，并产生MRAppmaster。
9. Container从HDFS上拷贝资源到本地。
10. MRAppmaster向RM 申请运行MapTask资源。
11. RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。
12. MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。
13. MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。
14. ReduceTask向MapTask获取相应分区的数据。
15. 程序运行完毕后，MR会向RM申请注销自己。

### YARN 的任务提交流程是怎样的

当 jobclient 向 YARN 提交一个应用程序后，YARN 将分两个阶段运行这个应用程序：一是启动 ApplicationMaster;第二个阶段是由 ApplicationMaster 创建应用程序，为它申请资源，监控运行直到结束。 具体步骤如下:

1. 用户向 YARN 提交一个应用程序，并指定 ApplicationMaster 程序、启动ApplicationMaster 的命令、用户程序。
2. RM 为这个应用程序分配第一个 Container，并与之对应的 NM 通讯，要求它在这个 Container 中启动应用程序 ApplicationMaster。
3. ApplicationMaster 向 RM 注册，然后拆分为内部各个子任务，为各个内部任务申请资源，并监控这些任务的运行，直到结束。
4. AM 采用轮询的方式向 RM 申请和领取资源。
5. RM 为 AM 分配资源，以 Container 形式返回。
6. AM 申请到资源后，便与之对应的 NM 通讯，要求 NM 启动任务。
7. NodeManager 为任务设置好运行环境，将任务启动命令写到一个脚本中，并通过运行这个脚本启动任务。
8. 各个任务向 AM 汇报自己的状态和进度，以便当任务失败时可以重启任务。
9. 应用程序完成后，ApplicationMaster 向 ResourceManager 注销并关闭自己。

### 介绍一下 Yarn 的 Job 提交流程

这里一共也有两个版本，分别是详细版和简略版，具体使用哪个还是分不同的场合。正常情况下，将简略版的回答清楚了就很OK，详细版的最多做个内容的补充：

![1633160796432](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/08/195240-108962.png)

**简略版步骤**

1. `client`向RM提交应用程序，其中包括启动该应用的`ApplicationMaster`的必须信息，例如`ApplicationMaster`程序、启动`ApplicationMaster`的命令、用户程序等
2. `ResourceManager`启动一个`container`用于运行`ApplicationMaster`
3. 启动中的`ApplicationMaster`向`ResourceManager`注册自己，启动成功后与RM保持心跳
4. `ApplicationMaster`向`ResourceManager`发送请求,申请相应数目的`container`
5. 申请成功的`container`，由`ApplicationMaster`进行初始化。`container`的启动信息初始化后，AM与对应的`NodeManager`通信，要求NM启动`container`
6. `NM`启动`container`
7. `container`运行期间，`ApplicationMaster`对`container`进行监控。`container`通过RPC协议向对应的AM汇报自己的进度和状态等信息
8. 应用运行结束后，`ApplicationMaster`向`ResourceManager`注销自己，并允许属于它的`container`被收回

**详细过程**

mapreduce过程

![1633930217670](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/133028-948748.png)

![1633930245989](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/133047-944890.png)

（1）作业提交

第1步：Client调用job.waitForCompletion方法，向整个集群提交MapReduce作业。

第2步：Client向RM申请一个作业id。

第3步：RM给Client返回该job资源的提交路径和作业id。

第4步：Client提交jar包、切片信息和配置文件到指定的资源提交路径。

第5步：Client提交完资源后，向RM申请运行MrAppMaster。

（2）作业初始化

第6步：当RM收到Client的请求后，将该job添加到容量调度器中。

第7步：某一个空闲的NM领取到该Job。

第8步：该NM创建Container，并产生MRAppmaster。

第9步：下载Client提交的资源到本地。

（3）任务分配

第10步：MrAppMaster向RM申请运行多个MapTask任务资源。

第11步：RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。

（4）任务运行

第12步：MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。

第13步：MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。

第14步：ReduceTask向MapTask获取相应分区的数据。

第15步：程序运行完毕后，MR会向RM申请注销自己。

（5）进度和状态更新

YARN中的任务将其进度和状态(包括counter)返回给应用管理器, 客户端每秒(通过mapreduce.client.progressmonitor.pollinterval设置)向应用管理器请求进度更新, 展示给用户。

（6）作业完成

除了向应用管理器请求作业进度外, 客户端每5秒都会通过调用waitForCompletion()来检查作业是否完成。时间间隔可以通过

mapreduce.client.completion.pollinterval来设置。作业完成之后, 应用管理器和Container会清理工作状态。作业的信息会被作业历史服务器存储以备之后用户核查。

### 介绍下Yarn默认的调度器，调度器分类，以及它们之间的区别

Hadoop调度器主要分为三类：

Hadoop作业调度器主要有三种：FIFO、Capacity Scheduler和Fair Scheduler。Hadoop3.1.3默认的资源调度器是Capacity Scheduler。

**先进先出调度器（FIFO）**

FIFO Scheduler 把应用按提交的顺序排成一个队列，这是一个先进先出队列，在进行资源分配的时候，先给队列中最头上的应用进行分配资源，待最头上的应用需求满足后再给下一个分配，以此类推。

FIFO Scheduler 是最简单也是最容易理解的调度器，也不需要任何配置，但它并不适用于共享集群。大的应用可能会占用所有集群资源，这就导致其它应用被阻塞，比如有个大任务在执行，占用了全部的资源，再提交一个小任务，则此小任务会一直被阻塞。

![1633678316114](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/08/153157-99769.png)

- FIFO Scheduler：先进先出调度器：优先提交的，优先执行，后面提交的等待【生产环境不会使用】
- 只有一个队列。

**容量调度器（Capacity Scheduler）**

![1633678392473](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/08/200659-397524.png)

- 支持多个队列，每个队列可配置一定的资源量，每个队列采用FIFO调度策略，既一个队列内部还是先进先出。
- 为了防止同一个用户的作业独占队列中的资源，该调度器会对同一用户提交的作业所占资源量进行限定。
  - 首先，计算每个队列中正在运行的任务数与其应该分得的计算资源之间的比值，选择一个该比值最小的队列。
  - 其次，按照作业优先级和提交时间顺序，同时考虑用户资源量限制和内存限制对队列内任务排序。
  - 三个队列同时按照任务的先后顺序依次执行。比如，job11、job21和job31分别排在队列最前面，是最先运行，也是同时运行。
  - 【Hadoop2.7.2默认的调度器】

**公平调度器（Fair Scheduler）**

![1633678541441](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/08/153542-707961.png)

- 支持多队列多用户，每个队列中的资源量可以配置，同一个队列中的作业公平共享队列中所有资源。
- 比如有三个队列: queueA、queueB 和queueC，每个队列中的job按照优先级分配资源，优先级越高分配的资源越多，但是每个job都会分配到资源以确保公平。在资源有限的情况下，每个job理想情况下获得的计算资源与实际获得的计算资源存在一种差距，这个差距就叫做缺额。在同一个队列中，job的资源缺额越大，越先获得资源优先执行。作业.是按照缺额的高低来先后执行的，而且可以看到上图有多个作业同时运行。

### 了解过哪些Hadoop的参数优化

我们常见的**Hadoop参数调优**有以下几种：

- 在hdfs-site.xml文件中配置多目录，最好提前配置好，否则更改目录需要重新启动集群,不配置多目录会导致所有数据文件全部存储在一个目录中。
- NameNode有一个工作线程池，用来处理不同DataNode的并发心跳以及客户端并发的元数据操作

```java
dfs.namenode.handler.count=20 * log2(Cluster Size)
  //比如集群规模为10台时，此参数设置为60
```

- 编辑日志存储路径dfs.namenode.edits.dir设置与镜像文件存储路径dfs.namenode.name.dir尽量分开，达到最低写入延迟
- 服务器节点上YARN可使用的物理内存总量，默认是8192（MB），注意，如果你的节点内存资源不够8GB，则需要调减小这个值，而YARN不会智能的探测节点的物理内存总量
- 单个任务可申请的最多物理内存量，默认是8192（MB）

### 了解过Hadoop的基准测试吗?

们搭建完Hadoop集群后需要对HDFS读写性能和MR计算能力测试。测试jar包在hadoop的share文件夹下。

为了搞清楚 HDFS 的读写性能，生产环境上非常需要对集群进行压测。

![1633932019053](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/140021-232746.png)

**测试 HDFS 写性能**

写数据原理

![1633932345580](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/140547-777496.png)

```java
hadoop jar /opt/module/hadoop3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-clientjobclient-3.1.3-tests.jar TestDFSIO -write -nrFiles 10 -fileSize 128MB
```

![1633932404360](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/140706-338859.png)

注意：nrFiles n 为生成 mapTask 的数量，生产环境一般可通过 hadoop103:8088 查看 CPU核数，设置为（CPU 核数 - 1）

![1633932487511](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/08/200824-157575.png)

![1633932503741](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/140836-459305.png)

测试结果分析

![1633932543040](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/140917-907634.png)

![1633932560802](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/140929-673277.png)

**测试 HDFS 读性能**

测试 HDFS 读性能

```java
hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-clientjobclient-3.1.3-tests.jar TestDFSIO -read -nrFiles 10 -fileSize  128MB
```

![1633932649707](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/141114-869276.png)

![1633932672034](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/11/141116-769533.png)

### 你是怎么处理Hadoop宕机的问题的?

- 如果MR造成系统宕机。此时要控制Yarn同时运行的任务数，和每个任务申请的最大内存。调整参数：`yarn.scheduler.maximum-allocation-mb`（单个任务可申请的最多物理内存量，默认是8192MB）。
- 如果写入文件过量造成NameNode宕机。那么调高Kafka的存储大小，控制从Kafka到HDFS的写入速度。高峰期的时候用Kafka进行缓存，高峰期过去数据同步会自动跟上。

### 你是如何解决Hadoop数据倾斜的问题的，能举个例子吗?

1. 提前在map进行combine，减少传输的数据量
   1. 在Mapper加上combiner相当于提前进行reduce，即把一个Mapper中的相同key进行了聚合，减少shuffle过程中传输的数据量，以及Reducer端的计算量。
   2. 如果导致数据倾斜的key 大量分布在不同的mapper的时候，这种方法就不是很有效了
2. 数据倾斜的key 大量分布在不同的mapper,在这种情况，大致有如下几种方法：

**「局部聚合加全局聚合」**

第一次在map阶段对那些导致了数据倾斜的key 加上1到n的随机前缀，这样本来相同的key 也会被分到多个Reducer 中进行局部聚合，数量就会大大降低。第一次执行是做预处理，尽量让数据分散：

第二次mapreduce，去掉key的随机前缀，进行全局聚合。

**「思想」**：二次mr，第一次将key随机散列到不同 reducer 进行处理达到负载均衡目的。第二次再根据去掉key的随机前缀，按原key进行reduce处理。**这个方法进行两次mapreduce，性能稍差**

**「增加Reducer，提升并行度」**

```java
JobConf.setNumReduceTasks(int)
```

**「实现自定义分区」**

根据数据分布情况，自定义散列函数，将key均匀分配到不同Reducer

### map-reduce程序运⾏的时候会有什么⽐较常见的问题？

⽐如说作业中⼤部分都完成了，但是总有⼏个reduce⼀直在运⾏。这是因为这⼏个reduce中的处理的数据要远远⼤于其他的reduce，可能是因为对键值对任务划分的不均匀造成的数据倾斜。解决的⽅法可以在分区的时候重新定义分区规则对于value数据很多的key可以进⾏拆分、均匀打散等处理，或者是在map端的combiner中进⾏数据预处理的操作。

### Hadoop性能调优？

1. 调优可以通过系统配置、程序编写和作业调度算法来进⾏。 hdfs的block.size可以调到128/256（⽹络很好的情况下，默认为64）
2. 调优的⼤头：mapred.map.tasks、mapred.reduce.tasks设置mr任务数（默认都是1）

```java
mapred.tasktracker.map.tasks.maximum //每台机器器上的最⼤大map任务数
mapred.tasktracker.reduce.tasks.maximum //每台机器器上的最⼤大reduce任务数
mapred.reduce.slowstart.completed.maps //配置reduce任务在map任务完成到百分之⼏几的时候开始进⼊入
//配置压缩项，消耗cpu提升⽹网络和磁盘io 合理理利利⽤用combiner 。注意重⽤用writable对象
mapred.compress.map.output,mapred.output.compress
```

这个⼏个参数要看实际节点的情况进⾏配置，reduce任务是在33%的时候完成copy，要在这之前完成map任务，（map可以提前完成）

**调优角度**

![1633245161347](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/08/200847-360774.png)

**分表：**对于一些大表，并且每一天是增量方式的增加存储数据，这样的话查询一次数据速度会很慢，所以可以采用分表，把一些常用的字段从大表中剥离出来，每天对小表进行操作，然后在更新大表。

**分区表：**比如用于的日志，每一天会产生大量的日志，如果存储在一个表中，会导致表过大，所以可以采取分区方式存储。分区表也是为了减少查询时候的查询范围。

**充分利用中间结果：**也就是针对一张大表，我们可以先查询出一部分数据，然后其他的查询都基于前面的子查询进行，这样显然可以提高效率。

![1633245878464](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/03/152439-646067.png)

**压缩**：好处是可以减少Io传输的数据量，节省磁盘空间，缺点是需要解压缩，耗费性能。压缩在大数据中的使用场景：输入数据，中间数据，输出数据。

![1633246243331](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/03/153045-471009.png)

mr过程：

![1633246315837](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/03/153158-889156.png)

1. 使用压缩后的数据作为map端的输入数据
2. map额度输出到reduce端的输入使用压缩
3. reduce端输出的结果可以压缩

上面在不同的阶段，我们需要的压缩比是不一样的，在reduce端我们更希望采用压缩比高的算法，因为更加的节省磁盘的空间，而在shuffle阶段的压缩，我们更新网压缩和解压缩速率高的。在map端的压缩，我们希望支持分片，这样可以有多个map。

### MapReduce 2.0 容错性

**MRAppMaster容错性**
一旦运行失败，由YARN的ResourceManager负责重新启动，最多重启次数可由用户设置，默认是2次。一旦超过最高重启次数，则作业运行失败。

**Map Task/Reduce**
Task Task周期性向MRAppMaster汇报心跳；一旦Task 挂掉，则MRAppMaster将为之重新申请资源，并运行之。最多重新运行次数可由用户设置，默认4 次。

### MapReduce推测执行算法及原理

作业完成时间取决于最慢的任务完成时间，一个作业由若干个Map 任务和Reduce 任务构成。因硬件老化、软件Bug 等，某些任务可能运行非常慢。

典型案例：系统中有99%的Map任务都完成了，只有少数几个Map老是进度很慢，完不成，怎么办？

**推测执行机制**

发现拖后腿的任务，比如某个任务运行速度远慢于任务平均速度。为拖后腿任务启动一个备份任务，同时运行。谁先运行完，则采用谁的结果

**不能启用推测执行机制情况**

（1）任务间存在严重的负载倾斜；
（2）特殊任务，比如任务向数据库中写数据。

**算法原理**

假设某一时刻，任务T的执行进度为progress，则可通过一定的算法推测出该任务的最终完成时刻estimateEndTime。另一方面，如果此刻为该任务启动一个备份任务，则可推断出它可能的完成时刻estimateEndTime,于是可得出以下几个公式：

```java
estimateEndTime = estimatedRunTime+taskStartTime（启动时刻）
estimatedRunTime = (currentTimestamp-taskStartTime)/progress
estimateEndTime = currentTimestamp(当前时刻)+averageRunTime（平均运行时间）
```

其中，currentTimestamp为当前时刻；taskStartTime为该任务的启动时刻；averageRunTime为已经成功运行完成的任务的平均运行时间。这样，MRv2总是选择（estimateEndTime-estimateEndTime·）差值最大的任务，并为之启动备份任务。为了防止大量任务同时启动备份任务造成的资源浪费，MRv2为每个作业设置了同时启动的备份任务数目上限。

推测执行机制实际上采用了经典的算法优化方法：以空间换时间，它同时启动多个相同任务处理相同的数据，并让这些任务竞争以缩短数据处理时间。显然，这种方法需要占用更多的计算资源。在集群资源紧缺的情况下，应合理使用该机制，争取在多用少量资源的情况下，减少作业的计算时间。

### 优化

#### MapReduce跑得慢的原因？

Mapreduce 程序效率的瓶颈在于两点：

**计算机性能**

CPU、内存、磁盘健康、网络

**I/O 操作优化**

1. 数据倾斜，通过合理分配数据到reduce进行解决。
2. map和reduce数设置不合理，设置合理的map输入和map输出
3. reduce等待过久，不必map执行完成后在执行reduce.
4. 小文件过多,合并小文件
5. 大量的不可分块的超大文件
6. spill次数过多，优化shuffle操作
7. merge次数过多等。优化shuffle操作

#### MapReduce优化方法

**数据输入阶段**

1. 合并小文件：在执行mr任务前将小文件进行合并，大量的小文件会产生大量的map任务，增大map任务装载次数，而任务的装载比较耗时，从而导致 mr 运行较慢。
2. 采用ConbinFileInputFormat来作为输入，解决输入端大量小文件场景。

**map阶段**

1. 减少spill次数：通过调整io.sort.mb及sort.spill.percent参数值，增大触发spill的内存上限，减少spill次数，从而减少磁盘 IO。
2. 减少merge次数：通过调整io.sort.factor参数，增大merge的文件数目，减少merge的次数，从而缩短mr处理时间。
3. 在 map 之后先进行combine处理，减少 I/O。

**reduce阶段**

1. 合理设置map和reduce数：两个都不能设置太少，也不能设置太多。太少，会导致task等待，延长处理时间；太多，会导致 map、reduce任务间竞争资源，造成处理超时等错误。
2. 设置map、reduce共存：调整slowstart.completedmaps参数，使map运行到一定程度后，reduce也开始运行，减少reduce的等待时间。
3. 规避使用reduce，因为Reduce在用于连接数据集的时候将会产生大量的网络消耗。
4. 合理设置reduce端的buffer，默认情况下，数据达到一个阈值的时候，buffer中的数据就会写入磁盘，然后reduce会从磁盘中获得所有的数据。也就是说，buffer和reduce是没有直接关联的，中间多个一个写磁盘->读磁盘的过程，既然有这个弊端，那么就可以通过参数来配置，使得buffer中的一部分数据可以直接输送到reduce，从而减少IO开销：mapred.job.reduce.input.buffer.percent，默认为0.0。当值大于0的时候，会保留指定比例的内存读buffer中的数据直接拿给reduce使用。这样一来，设置buffer需要内存，读取数据需要内存，reduce计算也要内存，所以要根据作业的运行情况进行调整。

**IO传输**

1. 采用数据压缩的方式，减少网络IO的的时间。安装Snappy和LZOP压缩编码器。
2. 使用SequenceFile二进制文件

#### 数据倾斜

**数据倾斜是如何产生的及解决方案**

数据倾斜，就是说某一类数据占据了总的数据量的80%或者更多，而某一类数据占比很少，那么这样就导致了数据倾斜的发生，如果从执行任务的角度来看，有的任务很快的就执行完成了，但是剩下一两个任务可能卡在了99%一直完成不了，那么这种情况有很大的可能发生数据倾斜。处理的方案如下：

数据倾斜现象

- 数据频率倾斜——某一个区域的数据量要远远大于其他区域。
- 数据大小倾斜——部分记录的大小远远大于平均值。

如何收集倾斜数据

在reduce方法中加入记录map输出键的详细情况的功能。

```java
public void reduce(Text key, Iterator<Text> values,
          OutputCollector<Text, Text> output,
          Reporter reporter) throws IOException {
  int i = 0;
  while (values.hasNext()) {
    values.next();
    i++;
  }
  if (++i > maxValueThreshold) {
    log.info("Received " + i + " values for key " + key);
  }
}
```

**减少数据倾斜的方法**

**提前在map进行combine，减少传输的数据量**

在Mapper加上combiner相当于提前进行reduce，即把一个Mapper中的相同key进行了聚合，减少shuffle过程中传输的数据量，以及Reducer端的计算量。

如果导致数据倾斜的key大量分布在不同的mapper的时候，这种方法就不是很有效了。

**导致数据倾斜的key 大量分布在不同的mapper**

（1）局部聚合加全局聚合。

第一次在map阶段对那些导致了数据倾斜的key 加上1到n的随机前缀，这样本来相同的key 也会被分到多个Reducer中进行局部聚合，数量就会大大降低。

第二次mapreduce，去掉key的随机前缀，进行全局聚合。

思想：二次mr，第一次将key随机散列到不同reducer进行处理达到负载均衡目的。第二次再根据去掉key的随机前缀，按原key进行reduce处理。

这个方法进行两次mapreduce，性能稍差。

（2）增加Reducer，提升并行度

JobConf.setNumReduceTasks(int)

（3）实现自定义分区

根据数据分布情况，自定义散列函数，将key均匀分配到不同Reducer

#### 常用调优参数

**资源相关参数**

（1）以下参数是在用户自己的MR应用程序中配置就可以生效（mapred-default.xml）

| 配置参数                                      | 参数说明                                                                                                        |
| --------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| mapreduce.map.memory.mb                       | 一个MapTask可使用的资源上限（单位:MB），默认为1024。如果MapTask实际使用的资源量超过该值，则会被强制杀死。       |
| mapreduce.reduce.memory.mb                    | 一个ReduceTask可使用的资源上限（单位:MB），默认为1024。如果ReduceTask实际使用的资源量超过该值，则会被强制杀死。 |
| mapreduce.map.cpu.vcores                      | 每个MapTask可使用的最多cpu core数目，默认值: 1                                                                  |
| mapreduce.reduce.cpu.vcores                   | 每个ReduceTask可使用的最多cpu core数目，默认值: 1                                                               |
| mapreduce.reduce.shuffle.parallelcopies       | 每个Reduce去Map中取数据的并行数。默认值是5                                                                      |
| mapreduce.reduce.shuffle.merge.percent        | Buffer中的数据达到多少比例开始写入磁盘。默认值0.66                                                              |
| mapreduce.reduce.shuffle.input.buffer.percent | Buffer大小占Reduce可用内存的比例。默认值0.7                                                                     |
| mapreduce.reduce.input.buffer.percent         | 指定多少比例的内存用来存放Buffer中的数据，默认值是0.0                                                           |

（2）应该在YARN启动之前就配置在服务器的配置文件中才能生效（yarn-default.xml）

| 配置参数                                 | 参数说明                                        |
| ---------------------------------------- | ----------------------------------------------- |
| yarn.scheduler.minimum-allocation-mb     | 给应用程序Container分配的最小内存，默认值：1024 |
| yarn.scheduler.maximum-allocation-mb     | 给应用程序Container分配的最大内存，默认值：8192 |
| yarn.scheduler.minimum-allocation-vcores | 每个Container申请的最小CPU核数，默认值：1       |
| yarn.scheduler.maximum-allocation-vcores | 每个Container申请的最大CPU核数，默认值：32      |
| yarn.nodemanager.resource.memory-mb      | 给Containers分配的最大物理内存，默认值：8192    |

（3）Shuffle性能优化的关键参数，应在YARN启动之前就配置好（mapred-default.xml）

| 配置参数                         | 参数说明                          |
| -------------------------------- | --------------------------------- |
| mapreduce.task.io.sort.mb        | Shuffle的环形缓冲区大小，默认100m |
| mapreduce.map.sort.spill.percent | 环形缓冲区溢出的阈值，默认80%     |

**容错相关参数(MapReduce性能优化)**

| 配置参数                     | 参数说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| mapreduce.map.maxattempts    | 每个Map Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| mapreduce.reduce.maxattempts | 每个Reduce Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| mapreduce.task.timeout       | Task超时时间，经常需要设置的一个参数，该参数表达的意思为：如果一个Task在一定时间内没有任何进入，即不会读取新的数据，也没有输出数据，则认为该Task处于Block状态，可能是卡住了，也许永远会卡住，为了防止因为用户程序永远Block住不退出，则强制设置了一个该超时时间（单位毫秒），默认是600000。如果你的程序对每条输入数据的处理时间过长（比如会访问数据库，通过网络拉取数据等），建议将该参数调大，该参数过小常出现的错误提示是“AttemptID:attempt_14267829456721_123456_m_000224_0   Timed out after 300 secsContainer killed by the ApplicationMaster.”。 |

### MapReduce 开发总结

**输入数据接口：InputFormat**

1. 默认使用的实现类是：TextInputFormat
2. TextInputFormat 的功能逻辑是：一次读一行文本，然后将该行的起始偏移量作为key，行内容作为 value 返回。
3. CombineTextInputFormat 可以把多个小文件合并成一个切片处理，提高处理效率。
4. KeyValueTextInputFormat每一行均为一条记录，被分隔符分割为key，value。默认分隔符是tab（\t）。
5. NlineInputFormat按照指定的行数N来划分切片。

**逻辑处理接口：Mapper**

1. 用户根据业务需求实现其中三个方法：map() setup() cleanup ()

**Partitioner 分区**

1. 有默认实现 HashPartitioner，逻辑是根据 key 的哈希值和 numReduces 来返回一个分区号；key.hashCode()&Integer.MAXVALUE % numReduces
2. 如果业务上有特别的需求，可以自定义分区。

**Comparable 排序**

1. 当我们用自定义的对象作为 key 来输出时，就必须要实现 WritableComparable 接口，重写其中的 compareTo()方法。
2. 部分排序：对最终输出的每一个文件进行内部排序。
3. 全排序：对所有数据进行排序，通常只有一个 Reduce。
4. 二次排序：排序的条件有两个。

**Combiner 合并**

1. Combiner 合并可以提高程序执行效率，减少 IO 传输。但是使用时必须不能影响原有的业务处理结果。

**逻辑处理接口：Reducer**

1. 用户根据业务需求实现其中三个方法：reduce() setup() cleanup ()

**输出数据接口：OutputFormat**

1. 默认实现类是 TextOutputFormat，功能逻辑是：将每一个 KV 对，向目标文本文件输出一行。
2. 用户还可以自定义 OutputFormat。

### 数据压缩

**压缩原则**

1. 运算密集型的 Job，少用压缩
2. IO 密集型的 Job，多用压缩

#### MR 支持的压缩编码

![1638965975481](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/08/201937-689257.png)

再实际项目中我们一般用Snappy，特点速度快，缺点无法切分（可以回答在链式MR中，Reduce端输出使用bzip2压缩，以便后续的map任务对数据进行split）

#### 压缩方式的选择

压缩方式选择时重点考虑：压缩/解压缩速度、压缩率（压缩后存储大小）、压缩后是否可以支持切片。

**Gzip 压缩**

优点：压缩率比较高，而且压缩/解压速度也比较快；hadoop本身支持，在应用中处理gzip格式的文件就和直接处理文本一样；大部分linux系统都自带gzip命令，使用方便。
缺点：不支持split。压缩/解压速度一般；

应用场景：当每个文件压缩之后在130M以内的（1个块大小内），都可以考虑用gzip压缩格式。例如说一天或者一个小时的日志压缩成一个gzip文件，运行mapreduce程序的时候通过多个gzip文件达到并发。hive程序，streaming程序，和java写的mapreduce程序完全和文本处理一样，压缩之后原来的程序不需要做任何修改。

**Bzip2 压缩**

优点：支持split；具有很高的压缩率，比gzip压缩率都高；hadoop本身支持，但不支持native；在linux系统下自带bzip2命令，使用方便。
缺点：压缩/解压速度慢；不支持native。

应用场景：适合对速度要求不高，但需要较高的压缩率的时候，可以作为mapreduce作业的输出格式；或者输出之后的数据比较大，处理之后的数据需要压缩存档减少磁盘空间并且以后数据用得比较少的情况；或者对单个很大的文本文件想压缩减少存储空间，同时又需要支持split，而且兼容之前的应用程序（即应用程序不需要修改）的情况 。

**Lzo 压缩**

优点：压缩/解压速度也比较快，合理的压缩率；支持split，是hadoop中最流行的压缩格式；可以在linux系统下安装lzop命令，使用方便。

缺点：压缩率比gzip要低一些；hadoop本身不支持，需要安装；在应用中对lzo格式的文件需要做一些特殊处理（为了支持split需要建索引，还需要指定inputformat为lzo格式）。

应用场景：一个很大的文本文件，压缩之后还大于200M以上的可以考虑，而且单个文件越大，lzo优点越越明显。

**Snappy 压缩**

优点：高速压缩速度和合理的压缩率。

缺点：不支持split；压缩率比gzip要低；hadoop本身不支持，需要安装；

应用场景：当Mapreduce作业的Map输出的数据比较大的时候，作为Map到Reduce的中间数据的压缩格式；或者作为一个Mapreduce作业的输出和另外一个Mapreduce作业的输入。

#### 压缩位置的选择

压缩可以在 MapReduce 作用的任意阶段启用。

![1633744754048](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/09/095917-842809.png)