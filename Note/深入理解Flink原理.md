## 深入理解Flink原理

[TOC]

### Flink是什么

Apache Flink 是一个框架和分布式处理引擎，用于对**无界和有界**数据流进行状态计算。

### 为什么选择Flink

流数据更真实地反映了我们的生活方式

传统的数据架构是基于有限数据集的

我们的目标

- 低延迟
- 高吞吐
- 结果的准确性和良好的容错性

### Flink流处理的特征

Flink 流处理特性：

- 支持高吞吐、低延迟、高性能的流处理
- 支持带有事件时间的窗口（Window）操作
- 支持有状态计算的 Exactly-once 语义
- 支持高度灵活的窗口（Window）操作，支持基于 time、count、session，以及 data-driven 的窗口操作
- 支持具有 Backpressure 功能的持续流模型
- 支持基于轻量级分布式快照（Snapshot）实现的容错
- 一个运行时同时支持 Batch on Streaming 处理和 Streaming 处理
- Flink 在 JVM 内部实现了自己的内存管理
- 支持迭代计算
- 支持程序自动优化：避免特定情况下 Shuffle、排序等昂贵操作，中间结果有必要进行缓存

### Flink的基石

Flink之所以能这么流行，离不开它最重要的四个基石：**Checkpoint（检查点）、State（状态）、Time（时间语义）、Window（窗口函数）。**

#### Checkpoint

这是Flink最重要的一个特性。

Flink基于Chandy-Lamport算法实现了一个分布式的一致性的快照，从而提供了一致性的语义。

Chandy-Lamport算法实际上在1985年的时候已经被提出来，但并没有被很广泛的应用，而Flink则把这个算法发扬光大了。

Spark最近在实现Continue streaming，Continue streaming的目的是为了降低处理的延时，其也需要提供这种一致性的语义，最终也采用了Chandy-Lamport这个算法，说明Chandy-Lamport算法在业界得到了一定的肯定。

#### State 

提供了一致性的语义之后，Flink为了让用户在编程时能够更轻松、更容易地去管理状态，还提供了一套非常简单明了的State API，包括里面的有ValueState、ListState、MapState，近期添加了BroadcastState，使用State API能够自动享受到这种一致性的语义。

#### Time

除此之外，Flink还实现了Watermark的机制，能够支持基于事件的时间的处理，能够容忍迟到/乱序的数据。

#### Window

另外流计算中一般在对流数据进行操作之前都会先进行开窗，即基于一个什么样的窗口上做这个计算。Flink提供了开箱即用的各种窗口，比如滑动窗口、滚动窗口、会话窗口以及非常灵活的自定义的窗口。

### 组件栈

![1621563860910](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621563860910.png)

**各层详细介绍：**

- 物理部署层：Flink 支持本地运行、能在独立集群或者在被 YARN 管理的集群上运行， 也能部署在云上，该层主要涉及Flink的部署模式，目前Flink支持多种部署模式：本地、集群(Standalone、YARN)、云(GCE/EC2)、Kubenetes。Flink能够通过该层能够支持不同平台的部署，用户可以根据需要选择使用对应的部署模式。

- Runtime核心层：Runtime层提供了支持Flink计算的全部核心实现，为上层API层提供基础服务，该层主要负责对上层不同接口提供基础服务，也是Flink分布式计算框架的核心实现层，支持分布式Stream作业的执行、JobGraph到ExecutionGraph的映射转换、任务调度等。将DataSteam和DataSet转成统一的可执行的Task Operator，达到在流式引擎下同时处理批量计算和流式计算的目的。

- API&Libraries层：Flink 首先支持了 Scala 和 Java 的 API，Python 也正在测试中。DataStream、DataSet、Table、SQL API，作为分布式数据处理框架，Flink同时提供了支撑计算和批计算的接口，两者都提供给用户丰富的数据处理高级API，例如Map、FlatMap操作等，也提供比较低级的Process Function API，用户可以直接操作状态和时间等底层数据。

- 扩展库：Flink 还包括用于复杂事件处理的CEP，机器学习库FlinkML，图处理库Gelly等。Table 是一种接口化的 SQL 支持，也就是 API 支持(DSL)，而不是文本化的SQL 解析和执行。

### 哪些行业需要处理流数据

![1621659568517](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621659568517.png)

电商和市场营销

- 数据报表、广告投放、业务流程需要

物联网（IOT）

-  传感器实时数据采集和显示、实时报警，交通运输业

电信业

-  基站流量调配

银行和金融业

- 实时结算和通知推送，实时检测异常行为

### 传统数据处理架构

**事务处理**

![1614254351391](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/195911-165114.png)

上面一层是计算层，下面层是存储层，数据计算和数据存储分开，事务处理就是当前应对的是一个一个的事务，但是事务性数据处理应对不了数据量很大的情况。但是这种情况实时性比较好，但是不能应对高并发，大数据量的情况。

**分析处理**

将数据从业务数据库复制到数仓，再进行分析和查询

![1614254709920](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/200511-798848.png)

将数据从数据库中取出先做ETL数据清洗，然后在放到数据仓库中，然后用分析计算的引擎计算处理，数据的来源可以从不同的数据库，统一处理后放到数据仓库，但是这个过程很慢，需要离线处理，不能做到事实处理。可以做到应对大数据量，但是不能做到实时处理。

**第一代：有状态的流式处理**

比如storm，低延迟做的很好，但是吞吐量不是很大，保证不了正确，数据乱序结果不能保证。

Spark Streaming:可以保证高吞吐，在高压下保证结果正确，容错性也很好，但是不能保证低延迟，对乱序数据处理不是很好。

![1614255372417](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/082406-103256.png)

这种数据的处理方式无法保证数据的来的顺序关系，所以产生第二代流式处理。

**第二代：lambda 架构**

用两套系统，同时保证低延迟和结果准确

![1614255489481](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/201809-596238.png)

事件日志，每一条日志对应一个事务的发生，Batch Layer是批处理层，Stream Processor是流处理层。用这两套系统同时保证结果的低延迟和准确性。Stream Processor层保证的是速度，但是存在的问题就是系统太麻烦，一个处理需要实现两套系统，代价很高。

**第三代流处理**

![1614255866157](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/082410-502959.png)

flink可以保证低延迟，高吞吐量，容错性比较好。

### 状态化流处理

几乎所有数据都是以连续事件流的形式产生。请考虑一下，无论是网站或移动应用中的用户交互或订单下达，还是服务器日志或传感器测量结果，这些数据本质上都是事件流。事实上，现实世界中很难找到那种瞬间就生成完整数据集的例子。作为一类面向无限事件流的应用设计模式，状态化流处理适用于企业 IT 基础设施中的很多应用场景。 

任何一个处理事件流的应用，如果要支持跨多条记录的转换操作，都必须是有状态的，即能够存储和访问中间结果。应用收到事件后可以执行包括读写状态在内的任意计算。原则上，需要在应用中访问的状态有多种可选的存储位置，例如:程序变量、本地文件、嵌入式或外部数据库等。 

Apache Flink 会将应用状态存储在本地内存或嵌入式数据库中。由 于采用 的是分布式架构， Flink 需要对本地状态予以保护，以避免因应用或机器故障导致数据丢失。为了实现该特性， Flink 会定期将应用状态的一致性检查点(checkpoint) 写入远程持久化存储。 

![1614424052530](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/190737-12284.png)

有状态的流处理应用通常会从事件日志中读取事件记录。事件日志负责存储事件流并将其分布式化。由于事件只 能以追加的形式写入持久化日志中，所以其顺序无法在后期改变 。写入事件日志的数据流可以被相同或不同 的消费者重复读取。得益于日志的追加特性，无论向消费者发布几次，事件的顺序都能保持一致。有不少事件日志系统都是开惊软件，其中最流行的当属Apache Kafka ，也有部分系统会以云计算提供商集成服务的形式提供。 

出于很多原因，将运行在 Flink 之上的有状态的流处理应用和事件日志系统相连会很有意义。在该架构下，事件日志系统可以持久化输入事件并以确定的顺序将其重放 。一旦出现故障， Flink 会利用之前的检查点恢复状态并重置事件日志的读取位置，以此来使有状态的流处理应用恢复正常。随后应用会从 事件日志中读取井(快速)重放输入事件，直到追赶上数据流当前的进度。 

### 一些概念

#### Dataflow 图

Dataflow 程序描述了数据如何在不同操作之间流动。Dataflow 程序通常表示为有向图。图中**顶点称为算子，表示计算**， **而边表示数据依赖关系。**算子是Dataflow 程序的基本功能单元，它们从输入获取数据，对其进行计算，然后产生数据并发往输出以供后续处理。没有输入端的算子称为数据源，没有输出端的算子称为数据汇。一个Dataflow 图至少要有一个数据源和一个数据汇

**逻辑图**

![1614346618229](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/082415-248304.png)

类似图2- 1 的Dataflow 图被称作逻辑图，因为它们表达了高层视角下的计算逻辑。为了执行Dataflow 程序，需要将逻辑图转化为物理Dataflow 图，后者会指定程序的执行细节。例如: 当我们使用分布式处理引擎时，**每个算子可能会在不同物理机器上运行多个并行任务**。图2-2 展示了图2-1 中逻辑图所对应的物理Dataflow 图。在逻辑Dataflow 图中，**顶点代表算子;在物理Dataflow 图中，顶点代表任务**。**"抽取主题标签"和"计数"算子都包含两个并行算子任务，每个任务负责计算一部分输入数据。**

也就是说一个算子可以分解为多个并行执行的任务。

**物理DataFlow图**

![1614346702527](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/26/213823-83256.png)

#### 数据并行和任务并行

Dataflow 图的并行性可以通过多种方式加以利用。首先，你可以将输入**数据分组**，让同一操作的多个任务并行执行在不同数据子集上，这种并行称为数据并行(data parallelism) 。数据井行非常有用，因为它能够将计算负载分配到多个节点上从而允许处理大规模的数据。再者，你可以让不同算子的任务(基于相同或不同的数据)并行计算，这种并行称为任务并行( task parallelism) 。通过任务并行，可以更好地利用集群的计算资源。

#### 数据交换策略

数据交换策略定义了如何将数据项分配给物理Dataflow 图中的不同任务。这些策略可以由执行引擎根据算子的语义自动选择，也可以由Dataflow 编程人员显式指定。接下来，我们结合图2-3 来简单了解一下常见的数据交换策略。

![1614346967396](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/082416-355216.png)

- 转发策略(forward strategy) 在发送端任务和接收端任务之间一对一地进行数据传输。如果两端任务运行在同一物理机器上(通常由任务调度器决定) ，该交换策略可以避免网络通信。
- 广播策略(broadcast strategy) 会把每个数据项发往下游算子的全部并行任务。该策略会把数据复制多份且涉及网络通信，因此代价十分昂贵。
- 基于键值的策略(key-based strategy) 根据某一键值属性对数据分区，并保证键值相同的数据项会交由同一任务处理。图2- 2 中， "抽取主题标签"算子的输出就是按照键值(主题标签)划分的，因此下游的计数算子可以正确计算出每个主题标签的出现次数。
- 随机策略(random strategy) 会将数据均句分配至算子的所有任务，以实现计算任务的负载均衡。

#### 并行流处理

数据流的定义: 数据流是一个可能无限的事件序列。数据流中的事件可以表示监控数据、传感器测量值、信用卡交易、气象站观测数据、在线用户交互，以及网络搜索等。

**延迟**

表示处理一个事件所需要的时间，在流式应用中，延时是以时间片为单位测量。

**吞吐**

吞吐量是用来衡量系统处理能力的指标，他告诉我们系统每单位时间可以处理多少事件。

> 其实上面的两个指标相互影响

#### 数据流上的操作

流处理引擎通常会提供一系列内置操作来实现数据流的获取、转换，以及输出。这些算子可以组合生成 Dataflow 处理图，从而实现流式应用所需的逻辑。 

- 数据接入和数据输出
- 转换操作
  - 转换操作是一类"只过一次"的操作，它们会分别处理每个事件。这些操作逐个读取事件，对其应用某些转换并产生一条新的输出流。
- 滚动聚合
  - 滚动聚合(如求和、求最小值和求最大值)会根据每个到来的事件持续更新结果。聚合操作都是有状态的，它们通过将新到来的事件合并到已有状态来生成更新后的聚合值
- 窗口操作

#### 时间语义

**处理时间 **

处理时间是当前流处理算子所在机器上的本地时钟时间 

**事件时间 **

事件时间是数据流中事件实际发生的时间，它以附加在数据流中事件的时间戳为依据 

### Flink 的主要特点

#### **事件驱动（Event-driven）**

Flink处理数据和传统的事件驱动型数据处理很像，先读取事件日志，然后存储到本地状态上面，如果要保证容错性，还可以定期进行存盘保存在磁盘中，最后输出到日志文件或者触发操作。

事件驱动型应用是一类具有**状态**的应用，它从一个或多个事件流提取数据，并根据到来的事件触发计算、状态更新或其他外部动作。比较典型的就是以 kafka 为代表的消息队列几乎都是事件驱动型应用。与之不同的就是 SparkStreaming 微批次，如图：

![1614327856778](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/222645-348669.png)

**事件驱动型：**

![1614256146967](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/202908-84379.png)

#### 流与批的世界观

批处理的特点是有界、持久、大量，非常适合需要访问全套记录才能完成的计算工作，一般用于离线统计。

流处理的特点是无界、实时, 无需针对整个数据集执行操作，而是对通过系统传输的每个数据项执行操作，一般用于实时统计。

在 spark 的世界观中，一切都是由批次组成的，离线数据是一个大批次，而实时数据是由一个一个无限的小批次组成的。

而在 flink 的世界观中，一切都是由流组成的，离线数据是有界限的流，实时数据是一个没有界限的流，这就是所谓的有界流和无界流。

**无界数据流：**

无界数据流有一个开始但是没有结束，它们不会在生成时终止并提供数据，必须连续处理无界流，也就是说必须在获取后立即处理 event。对于无界数据流我们无法等待所有数据都到达，因为输入是无界的，并且在任何时间点都不会完成。处理无界数据通常要求以特定顺序（例如事件发生的顺序）获取 event，以便能够推断结果完整性。

 **有界数据流：**

有界数据流有明确定义的开始和结束，可以在执行任何计算之前通过获取所有数据来处理有界流，处理有界流不需要有序获取，因为可以始终对有界数据集进行排序，有界流的处理也称为批处理。

![1614331802959](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/222640-998093.png)

这种以流为世界观的架构，获得的最大好处就是具有极低的延迟。

#### 分层 api

- 越顶层越抽象，表达含义越简明，使用越方便
- 越底层越具体，表达能力越丰富，使用越灵活

![1614331948887](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/26/173514-873064.png)

最底层级的抽象仅仅提供了有状态流，它将通过过程函数（Process Function）被嵌入到 DataStream API 中。底层过程函数（Process Function） 与 DataStream API相集成，使其可以对某些特定的操作进行底层的抽象，它允许用户可以自由地处理来自一个或多个数据流的事件，并使用一致的容错的状态。除此之外，用户可以注册事件时间并处理时间回调，从而使程序可以处理复杂的计算。

实际上，大多数应用并不需要上述的底层抽象，而是针对核心 API（Core APIs）进行编程，比如 DataStream API（有界或无界流数据）以及 DataSet API（有界数据集）。这些 API 为数据处理提供了通用的构建模块，比如由用户定义的多种形式的转换（transformations），连接（joins），聚合（aggregations），窗口操作（windows）等等。DataSet API 为有界数据集提供了额外的支持，例如循环与迭代。这些 API处理的数据类型以类（classes）的形式由各自的编程语言所表示。

Table API 是以表为中心的声明式编程，其中表可能会动态变化（在表达流数据时）。Table API 遵循（扩展的）关系模型：表有二维数据结构（schema）（类似于关系数据库中的表），同时 API 提供可比较的操作，例如 select、project、join、group-by、aggregate 等。Table API 程序声明式地定义了什么逻辑操作应该执行，而不是准确地确定这些操作代码的看上去如何。

尽管 Table API 可以通过多种类型的用户自定义函数（UDF）进行扩展，其仍不如核心 API 更具表达能力，但是使用起来却更加简洁（代码量更少）。除此之外，Table API 程序在执行之前会经过内置优化器进行优化。你可以在表与 DataStream/DataSet 之间无缝切换，以允许程序将 Table API 与DataStream 以及 DataSet 混合使用。

Flink 提供的最高层级的抽象是 SQL 。这一层抽象在语法与表达能力上与Table API 类似，但是是以 SQL 查询表达式的形式表现程序。SQL 抽象与 Table API交互密切，同时 SQL 查询可以直接在 Table API 定义的表上执行。

目前 Flink 作为批处理还不是主流，不如 Spark 成熟，所以 DataSet 使用的并不是很多。Flink Table API 和 Flink SQL 也并不完善，大多都由各大厂商自己定制。所以我们主要学习 DataStream API 的使用。实际上 Flink 作为最接近 Google DataFlow模型的实现，是流批统一的观点，所以基本上使用 DataStream 就可以了。

**Flink 几大模块**

- Flink Table & SQL(还没开发完)
- Flink Gelly(图计算)
- Flink CEP(复杂事件处理)

#### Flink其他的特点

- 支持事件时间（event-time）和处理时间（processing-time）语义
- 精确一次（exactly-once）的状态一致性保证，出现故障之后可以恢复到原来的状态、
- 低延迟，每秒处理数百万个事件，毫秒级延迟，低延迟，高吞吐量
- 与众多常用存储系统的连接
- 高可用，动态扩展，实现7*24小时全天候运行

#### Flink vs Spark Streaming

流（stream）和微批（micro-batching）

![1614332595714](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/26/174318-581777.png)

sparking streaming是批处理，只是每一批数据积攒的很小，而flink是真正意义上的流式处理，从架构上来说，spark就是批处理的，所以存在延迟，不管批次有多小，总存在延迟发生，但是flink底层就是流处理，两个框架的底层架构不同

**底层数据模型的区别**

- 数据模型
  - spark 采用 RDD 模型，spark streaming 的 DStream 实际上也就是一组 组小批数据 RDD 的集合
  - flink 基本数据模型是数据流（dataflow），以及事件（Event）序列,处理的是一条一条的数据
- 运行时架构
  - spark 是批计算，将 DAG 划分为不同的 stage，一个完成后才可以计算下一个，所以阶段之间存在依赖性，上一个阶段如果没有完成的话，就不能进行下一个阶段
  - flink 是标准的流执行模式，一个事件在一个节点处理完后可以直接发往下一个节点进行处理，中间没有阶段的划分，所以也没有延迟。

### 为什么选择Flink

**主要原因**

1. Flink 具备统一的框架处理有界和无界两种数据流的能力
2. 部署灵活，Flink 底层支持多种资源调度器，包括Yarn、Kubernetes 等。Flink 自身带的Standalone 的调度器，在部署上也十分灵活。
3. 极高的可伸缩性，可伸缩性对于分布式系统十分重要，阿里巴巴双11大屏采用Flink 处理海量数据，使用过程中测得Flink 峰值可达17 亿条/秒。
4. 极致的流式处理性能。Flink 相对于Storm 最大的特点是将状态语义完全抽象到框架中，支持本地状态读取，避免了大量网络IO，可以极大提升状态存取的性能。

 **其他更多的原因:**

1. 同时支持高吞吐、低延迟、高性能

2. Flink 是目前开源社区中唯一一套集**高吞吐、低延迟、高性能**三者于一身的分布式流式数据处理框架。

3. Spark 只能兼顾高吞吐和高性能特性，无法做到低延迟保障,因为Spark是用批处理来做流处理

4. Storm 只能支持低延时和高性能特性，无法满足高吞吐的要求

**支持事件时间(Event Time)概念**

在流式计算领域中，窗口计算的地位举足轻重，但目前大多数框架窗口计算采用的都是系统时间(Process Time)，

也就是事件传输到计算框架处理时，系统主机的当前时间。

Flink 能够支持基于事件时间(Event Time)语义进行窗口计算

这种基于事件驱动的机制使得事件即使乱序到达甚至延迟到达，流系统也能够计算出精确的结果，保持了事件原本产生时的时序性，尽可能避免网络传输或硬件系统的影响。

![1621660006900](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/165452-629379.png)

**支持有状态计算**

Flink1.4开始支持有状态计算

所谓状态就是在流式计算过程中将算子的中间结果保存在内存或者文件系统中，等下一个事件进入算子后可以从之前的状态中获取中间结果，计算当前的结果，从而无须每次都基于全部的原始数据来统计结果，极大的提升了系统性能，状态化意味着应用可以维护随着时间推移已经产生的数据聚合

![1621660055700](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/165454-678728.png)

**支持高度灵活的窗口(Window)操作**

Flink 将窗口划分为基于 Time 、Count 、Session、以及Data-Driven等类型的窗口操作，窗口可以用灵活的触发条件定制化来达到对复杂的流传输模式的支持，用户可以定义不同的窗口触发机制来满足不同的需求

**基于轻量级分布式快照(Snapshot/Checkpoints)的容错机制**

Flink 能够分布运行在上千个节点上，通过基于分布式快照技术的Checkpoints，将执行过程中的状态信息进行持久化存储，一旦任务出现异常停止，Flink 能够从 Checkpoints 中进行任务的自动恢复，以确保数据处理过程中的一致性

Flink 的容错能力是轻量级的，允许系统保持高并发，同时在相同时间内提供强一致性保证。

![1621660134121](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/165455-500063.png)

**基于 JVM 实现的独立的内存管理**

Flink 实现了自身管理内存的机制，通过使用散列，索引，缓存和排序有效地进行内存管理，通过序列化/反序列化机制将所有的数据对象转换成二进制在内存中存储，降低数据存储大小的同时，更加有效的利用空间。使其独立于 Java 的默认垃圾收集器，尽可能减少 JVM GC 对系统的影响。

 **SavePoints 保存点**

对于 7 * 24 小时运行的流式应用，数据源源不断的流入，在一段时间内应用的终止有可能导致数据的丢失或者计算结果的不准确。

比如集群版本的升级，停机运维操作等。

值得一提的是，Flink 通过SavePoints 技术将任务执行的快照保存在存储介质上，当任务重启的时候，可以从事先保存的 SavePoints 恢复原有的计算状态，使得任务继续按照停机之前的状态运行。

Flink 保存点提供了一个状态化的版本机制，使得能以无丢失状态和最短停机时间的方式更新应用或者回退历史数据。

![1621660228183](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/142956-940682.png)

**灵活的部署方式，支持大规模集群**

Flink 被设计成能用上千个点在大规模集群上运行

除了支持独立集群部署外，Flink 还支持 YARN 和Mesos 方式部署。

**Flink 的程序内在是并行和分布式的**

数据流可以被分区成 stream partitions，

operators 被划分为operator subtasks; 

这些 subtasks 在不同的机器或容器中分不同的线程独立运行；

operator subtasks 的数量就是operator的并行计算数，不同的 operator 阶段可能有不同的并行数；

如下图所示，source operator 的并行数为 2，但最后的 sink operator 为1；

![1621660321711](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/165458-516420.png)

### 流处理和批处理

**数据的时效性**

日常工作中，我们一般会先把数据存储在**表**，然后对表的数据进行**加工、分析**。既然先存储在表中，那就会涉及到时效性概念。

如果我们处理以年，月为单位的级别的数据处理，进行**统计分析，个性化推荐**，那么数据的的最新日期离当前有**几个甚至上月**都没有问题。但是如果我们处理的是**以天为级别**，或者**一****小时**甚至**更小粒度**的数据处理，那么就要求数据的时效性更高了。比如：

- 对网站的实时监控

- 对异常日志的监控

这些场景需要工作人员**立即响应**，这样的场景下，传统的统一收集数据，再存到数据库中，再取出来进行分析就无法满足高时效性的需求了。

**流式计算和批量计算**

![1621660594100](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/165502-264342.png)

- Batch Analytics，右边是 Streaming Analytics。批量计算: 统一收集数据->存储到DB->对数据进行批量处理，就是传统意义上使用类似于 Map Reduce、Hive、Spark Batch 等，对作业进行分析、处理、生成离线报表

- Streaming Analytics 流式计算，顾名思义，就是对数据流进行处理，如使用流式分析引擎如 Storm，Flink 实时处理分析数据，应用较多的场景如实时大屏、实时报表。

**它们的主要区别是：**

- 与批量计算那样慢慢积累数据不同，流式计算立刻计算，数据持续流动，计算完之后就丢弃。
- 批量计算是维护一张表，对表进行实施各种计算逻辑。流式计算相反，是必须先定义好计算逻辑，提交到流式计算系统，这个计算作业逻辑在整个运行期间是不可更改的。
- 计算结果上，批量计算对全部数据进行计算后传输结果，流式计算是每次小批量计算后，结果可以立刻实时化展现。

**对比**

![1621749815430](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/140337-942786.png)

### 流处理和批处理统一

在大数据处理领域，批处理任务与流处理任务一般被认为是两种不同的任务，一个大数据框架一般会被设计为只能处理其中一种任务：

- MapReduce只支持批处理任务；

- Storm只支持流处理任务；

- Spark Streaming采用micro-batch架构，本质上还是基于Spark批处理对流式数据进行处理

- Flink通过灵活的执行引擎，能够同时支持批处理任务与流处理任务

![1621660852208](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/165504-865466.png)

在执行引擎这一层，流处理系统与批处理系统最大不同在于节点间的数据传输方式：

1. 对于一个流处理系统，其节点间数据传输的标准模型是：当一条数据被处理完成后，序列化到缓存中，然后立刻通过网络传输到下一个节点，由下一个节点继续处理

2. 对于一个批处理系统，其节点间数据传输的标准模型是：当一条数据被处理完成后，序列化到缓存中，并不会立刻通过网络传输到下一个节点，当缓存写满，就持久化到本地硬盘上，当所有数据都被处理完成后，才开始将处理后的数据通过网络传输到下一个节点

这两种数据传输模式是两个极端，对应的是流处理系统对低延迟的要求和批处理系统对高吞吐量的要求

Flink的执行引擎采用了一种十分灵活的方式，同时支持了这两种数据传输模型：

Flink以固定的缓存块为单位进行网络数据传输，用户可以通过设置缓存块超时值指定缓存块的传输时机。

如果缓存块的超时值为0，则Flink的数据传输方式类似上文所提到流处理系统的标准模型，此时系统可以获得最低的处理延迟

如果缓存块的超时值为无限大，则Flink的数据传输方式类似上文所提到批处理系统的标准模型，此时系统可以获得最高的吞吐量

同时缓存块的超时值也可以设置为0到无限大之间的任意值。缓存块的超时阈值越小，则Flink流处理执行引擎的数据处理延迟越低，但吞吐量也会降低，反之亦然。通过调整缓存块的超时阈值，用户可根据需求灵活地权衡系统延迟和吞吐量

默认情况下，流中的元素并不会一个一个的在网络中传输，而是缓存起来伺机一起发送(默认为32KB，通过taskmanager.memory.segment-size设置),这样可以避免导致频繁的网络传输,提高吞吐量，但如果数据源输入不够快的话会导致后续的数据处理延迟，所以可以使用env.setBufferTimeout(默认100ms)，来为缓存填入设置一个最大等待时间。等待时间到了之后，即使缓存还未填满，缓存中的数据也会自动发送。 

- timeoutMillis > 0 表示最长等待 timeoutMillis 时间，就会flush

- timeoutMillis = 0 表示每条数据都会触发 flush，直接将数据发送到下游，相当于没有Buffer了(避免设置为0，可能导致性能下降)

- timeoutMillis = -1 表示只有等到 buffer满了或 CheckPoint的时候，才会flush。相当于取消了 timeout 策略

**总结:**

Flink以缓存块为单位进行网络数据传输,用户可以设置缓存块超时时间和缓存块大小来控制缓冲块传输时机,从而控制Flink的延迟性和吞吐量

 ### 计算框架发展史

![1621660996990](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621660996990.png)

这几年大数据的飞速发展，出现了很多热门的开源社区，其中著名的有Hadoop、Storm，以及后来的 Spark，他们都有着各自专注的应用场景。Spark 掀开了内存计算的先河，也以内存为赌注，赢得了内存计算的飞速发展。Spark 的火热或多或少的掩盖了其他分布式计算的系统身影。就像Flink，也就在这个时候默默的发展着。

在国外一些社区，有很多人将大数据的计算引擎分成了 4 代，当然，也有很多人不会认同。我们先姑且这么认为和讨论。

1. 第1代——Hadoop MapReduce

首先第一代的计算引擎，无疑就是 Hadoop 承载的 MapReduce。它将计算分为两个阶段，分别为 Map 和 Reduce。对于上层应用来说，就不得不想方设法去拆分算法，甚至于不得不在上层应用实现多个 Job 的串联，以完成一个完整的算法，例如迭代计算。

- 批处理

- Mapper、Reducer

2. 第2代——DAG框架（Tez） + MapReduce

由于这样的弊端，催生了支持 DAG 框架的产生。因此，支持 DAG 的框架被划分为第二代计算引擎。如 Tez 以及更上层的 Oozie。这里我们不去细究各种 DAG 实现之间的区别，不过对于当时的 Tez 和 Oozie 来说，大多还是批处理的任务。

- 批处理

- 1个Tez = MR(1) + MR(2) + ... + MR(n)

- 相比MR效率有所提升

![1621661167571](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621661167571.png)

3. 第3代——Spark

接下来就是以 Spark 为代表的第三代的计算引擎。第三代计算引擎的特点主要是 Job 内部的 DAG 支持（不跨越 Job），以及强调的实时计算。在这里，很多人也会认为第三代计算引擎也能够很好的运行批处理的 Job。

- 批处理、流处理、SQL高层API支持

- 自带DAG

- 内存迭代计算、性能较之前大幅提升

4. 第4代——Flink

随着第三代计算引擎的出现，促进了上层应用快速发展，例如各种迭代计算的性能以及对流计算和 SQL 等的支持。Flink 的诞生就被归在了第四代。这应该主要表现在 Flink 对流计算的支持，以及更一步的实时性上面。当然 Flink 也可以支持 Batch 的任务，以及 DAG 的运算。

-  批处理、流处理、SQL高层API支持

- 自带DAG

- 流式计算性能更高、可靠性更高

## Flink版WordCount

### 批处理

~~~ java
/**
 * 批处理的wordCount程序
 */
public class WordCount {

    public static void main(String[] args) throws Exception {

//      创建执行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

//        从文件中读取数据
        String path="D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt";

//        读取数据,读取文本文件是按照行读取
//        DataSet主要用来做离线的数据处理
        DataSet<String> ds = env.readTextFile(path);

//        对数据及进行处理,把每一行中的单词分开，处理成(word,1)形式
//        groupby(0):按照第一个位置处的元素进行排序,sum(1)表示按照第二个位置处的元素累加
        DataSet<Tuple2<String, Integer>> result = ds.flatMap(new MyFlatMap()).groupBy(0).sum(1);

        result.print();
    }


//    定义MyFlatMap类，实现FlatMapFunction接口
    public static class MyFlatMap implements FlatMapFunction<String, Tuple2<String,Integer>> {

    //        o表示输入进来的数据，处理好的数据，从collector输出
    public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {

//        按照空格进行分词
        String[] words = s.split(" ");
        
//        遍历所有word,循环输出
        for (String word:words) {

//            输出数据
            collector.collect(new Tuple2<String ,Integer>(word,1));
        }
    }
}
}

~~~

### 流式处理

~~~ java
public class StreamWordCount {

    public static void main(String[] args) throws Exception {

//        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
//       设置并行度
        env.setParallelism(8);

        String path="D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt";
//        读取数据
        DataStream<String> dss = env.readTextFile(path);

//        基于数据流进行转换计算,按照第一个位置的元素分组，第二个位置的元素累加
        DataStream<Tuple2<String, Integer>> result = dss.flatMap(new WordCount.MyFlatMap()).keyBy(0).sum(1);

        result.print();

//        执行任务
        env.execute();
    }
}
~~~

## Flink部署

Flink支持多种安装模式

- Local—本地单机模式，学习测试时使用
- Standalone—独立集群模式，Flink自带集群，开发测试环境使用
- StandaloneHA—独立集群高可用模式，Flink自带集群，开发测试环境使用
- On Yarn—计算资源统一由Hadoop YARN管理，生产环境使用

### Local本地模式安装

#### 原理

![1621563914228](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/132832-31487.png)

本地模式下面是以多线程去模拟Flink集群资源。

1. Flink程序由JobClient进行提交

2. JobClient将作业提交给JobManager

3. JobManager负责协调资源分配和作业执行。资源分配完成后，任务将提交给相应的TaskManager

4. TaskManager启动一个线程以开始执行。TaskManager会向JobManager报告状态更改,如开始执行，正在进行或已完成。 

5. 作业执行完成后，结果将发送回客户端(JobClient)

#### 操作

1.下载安装包

https://archive.apache.org/dist/flink/

2.上传flink-1.12.0-bin-scala_2.12.tgz到node1的指定目录

3.解压

~~~ java
tar -zxvf flink-1.12.0-bin-scala_2.12.tgz 
~~~

4.如果出现权限问题，需要修改权限

~~~ java
chown -R root:root /export/server/flink-1.12.0
~~~

5.改名或创建软链接

~~~ java
mv flink-1.12.0 flink

ln -s /export/server/flink-1.12.0 /export/server/flink
~~~

#### 测试

1.准备文件/root/words.txt

```
vim /root/words.txt

hello me you her
hello me you
hello me
hello
```

2.启动Flink本地“集群”

  ~~~ java
/export/server/flink/bin/start-cluster.sh
  ~~~

3.使用jps可以查看到下面两个进程

~~~ java
  \- TaskManagerRunner

  \- StandaloneSessionClusterEntrypoint
~~~

4. 访问Flink的Web UI

![1621564188615](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621564188615.png)

slot在Flink里面可以认为是资源组，Flink是通过将任务分成子任务并且将这些子任务分配到slot来并行执行程序。

5. 执行官方案例

~~~ java
/export/server/flink/bin/flink run /export/server/flink/examples/batch/WordCount.jar --input /root/words.txt --output /root/out
~~~

6. 停止Flink

~~~ java
/export/server/flink/bin/stop-cluster.sh
~~~

### Standalone 模式

#### 原理，

![1621564602661](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/103729-430205.png)

![1621564329905](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/170405-405689.png)

**规划**

1.集群规划:

\- 服务器: hadoop100(Master + Slave): JobManager + TaskManager

\- 服务器: hadoop101(Slave): TaskManager

\- 服务器: hadoop102(Slave): TaskManager

#### 安装

1. 解压缩 flink-1.10.1-bin-scala_2.12.tgz，进入 conf 目录中。
2. 修改 flink/conf/flink-conf.yaml 文件：

~~~ java
jobmanager.rpc.address: hadoop100 //配置jobmanager地址,相当于spark中的driver
taskmanager.numberOfTaskSlots: 2 //配置每一个taskmanager的slot数量
web.submit.enable: true //配置可用通过web ui进行访问

#历史服务器
jobmanager.archive.fs.dir: hdfs://hadoop100:8020/flink/completed-jobs/
historyserver.web.address: hadoop100
historyserver.web.port: 8082
historyserver.archive.fs.dir: hdfs://hadoop100:8020/flink/completed-jobs/

//修改masters
vim /export/server/flink/conf/masters
hadoop100:8081
~~~

3. 修改 /conf/slaves 文件：

~~~ java
//添加如下内容，也就是集群节点的地址
hadoop101
hadoop102
~~~

4. 分发给另外两台机子：

~~~ java
xsync flink-1.10.1
~~~

5. 启动flink

~~~ java
[rzf@hadoop100 flink-1.10.1]$ bin/start-cluster.sh  //也可以进行单点启动
Starting cluster.
Starting standalonesession daemon on host hadoop100. //后面提交任务就是通过这个进程提交
Starting taskexecutor daemon on host hadoop101. //具体执行任务的节点
Starting taskexecutor daemon on host hadoop102.
~~~

6. 访问flink集群

~~~ java
http://hadoop100:8081/#/overview //通过8081端口上面已经配置过8081端口

http://hadoop100:8081/#/overview

http://hadoop100:8082/#/overview //访问历史服务器
~~~

![1614308298297](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/222551-596516.png)

7. 执行官方案例

~~~ java
/export/server/flink/bin/flink run /export/server/flink/examples/batch/WordCount.jar
~~~

#### StandAlone的HA模式

![1621565322255](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/170706-63710.png)

并不是说jobManager挂掉以后，另一个jobManager会立马上位，而是说还会有一个超时时间。

从之前的架构中我们可以很明显的发现 JobManager 有明显的单点问题(SPOF，single point of failure)。JobManager 肩负着任务调度以及资源分配，一旦 JobManager 出现意外，其后果可想而知。在 Zookeeper 的帮助下，一个 Standalone的Flink集群会同时有多个活着的 JobManager，其中只有一个处于工作状态，其他处于 Standby 状态。当工作中的 JobManager 失去连接后(如宕机或 Crash)，Zookeeper 会从 Standby 中选一个新的 JobManager 来接管 Flink 集群。

**操作**

1.集群规划

~~~ java
\- 服务器: hadoop100(Master + Slave): JobManager + TaskManager

\- 服务器: hadoop101(Master + Slave): JobManager + TaskManager

\- 服务器: hadoop102(Slave): TaskManager
~~~

2.启动ZooKeeper

~~~ ja
zkServer.sh status

zkServer.sh stop

zkServer.sh start
~~~

3.启动HDFS

~~~ java
/export/serves/hadoop/sbin/start-dfs.sh
~~~

4.停止Flink集群

~~~ java
/export/server/flink/bin/stop-cluster.sh
~~~

5.修改flink-conf.yaml,增加如下内容G

```
state.backend: filesystem
state.backend.fs.checkpointdir: hdfs://hadoop100:8020/flink-checkpoints
high-availability: zookeeper
high-availability.storageDir: hdfs://hadoop100:8020/flink/ha/
high-availability.zookeeper.quorum: hadoop100:2181,hadoop101:2181,hadoop102:2181
```

6.修改masters

vim /export/server/flink/conf/masters

~~~ java
hadoop100 8081
hadoop101 8081
~~~

7.同步

```
scp -r /export/server/flink/conf/flink-conf.yaml hadoop101:/export/server/flink/conf/
scp -r /export/server/flink/conf/flink-conf.yaml hadoop102:/export/server/flink/conf/
scp -r /export/server/flink/conf/masters hadoop101:/export/server/flink/conf/
scp -r /export/server/flink/conf/masters hadoop102:/export/server/flink/conf/
```

8.修改hadoop101上的flink-conf.yam

```
vim /export/server/flink/conf/flink-conf.yaml
jobmanager.rpc.address: hadoop101
```

9.重新启动Flink集群,node1上执行

~~~ java
/export/server/flink/bin/stop-cluster.sh

/export/server/flink/bin/start-cluster.sh
~~~

![1621565780708](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621565780708.png)

10.使用jps命令查看,发现没有Flink相关进程被启动

11.查看日志,发现如下错误

![1621565849864](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621565849864.png)

因为在Flink1.8版本后,Flink官方提供的安装包里没有整合HDFS的jar

12. 下载jar包并在Flink的lib目录下放入该jar包并分发使Flink能够支持对Hadoop的操作

13. 放入lib目录

![1621565909778](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621565909778.png)

14. 分发

15. 重新启动Flink集群,hadoop100上执行

~~~ java
/export/server/flink/bin/stop-cluster.sh

/export/server/flink/bin/start-cluster.sh
~~~

16. 使用jps命令查看,发现三台机器已经ok

**测试**

1. 访问WebUI

~~~ java
http://hadoop100:8081/#/job-manager/config

http://hadoop101:8081/#/job-manager/config
~~~

2. 执行wc

~~~ java
/export/server/flink/bin/flink run  /export/server/flink/examples/batch/WordCount.jar
~~~

3. kill掉其中一个master

4. 重新执行wc,还是可以正常执行

~~~java
/export/server/flink/bin/flink run  /export/server/flink/examples/batch/WordCount.jar

~~~

5. 停止集群

~~~ java
/export/server/flink/bin/stop-cluster.sh
~~~

### Flink-On-Yarn-模式

#### 原理

![1621569017735](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/170739-620796.png)

**Flink和Yarn的交互过程**

![1621569052576](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/115214-814296.png)

1. 用户从yarn客户端上传jar包和配置文件到hdfs文件系统上面。
2. 找Resourcemanager注册并且申请资源。
3. resourcemanager会找到一个空闲的nodemanager启动applicationmaster作业。启动完毕之后，就相当于启动了一个jobManager进程，负责管理整个作业，jobManager和ApplicationMaster运行在同一个container上。一旦他们被成功启动，AppMaster就知道JobManager的地址(AM它自己所在的机器)。它就会为TaskManager生成一个新的Flink配置文件(他们就可以连接到JobManager)。这个配置文件也被上传到HDFS上。此外，AppMaster容器也提供了Flink的web服务接口。YARN所分配的所有端口都是临时端口，这允许用户并行执行多个Flink
4. applicationmaster向resourcemanager申请资源，也就是申请节点使用.
5. applicationmaster启动其余的节点，也就是taskManager
6. 所有节点启动开之后，每隔节点都去hdfs上面下载jar包和配置文件信息。

#### 两种模式

**Session会话模式**

在yarn集群中启动了一个Flink集群，并且会重复利用该集群

![1621569957189](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/120655-703592.png)

![1621661620440](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/133341-669155.png)

**Job分离模式**

![1621570170828](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/120932-662903.png)

![1621661649056](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/133434-869292.png)

对于每一个作业都是申请一个flink集群，作业执行完毕之后就关闭集群资源。

#### 集群部署

1. 关闭yarn的内存检查

~~~ java
//因flink和spark一样，在运行的时候需要大量的内存
vim /export/server/hadoop/etc/hadoop/yarn-site.xml

<!-- 关闭yarn内存检查 -->
    <property>
        <name>yarn.nodemanager.pmem-check-enabled</name>
        <value>false</value>
    </property>
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>
~~~

2. 分发修改后的配置文件

3. 重启yarn

~~~ java
/export/server/hadoop/sbin/stop-yarn.sh

/export/server/hadoop/sbin/start-yarn.sh
~~~

##### Session会话模式

在Yarn上启动一个Flink集群,并重复使用该集群,后续提交的任务都是给该集群,资源会被一直占用,除非手动关闭该集群----适用于大量的小任务

1.在yarn上启动一个Flink集群/会话，hadoop100上执行以下命令

~~~ java
/export/server/flink/bin/yarn-session.sh -n 2 -tm 800 -s 1 -d
~~~

**说明:**

~~~ java
- 申请2个CPU、1600M内存

-n 表示申请2个容器，这里指的就是多少个taskmanager

-tm 表示每个TaskManager的内存大小

-s 表示每个TaskManager的slots数量

-d 表示以后台程序方式运行
~~~

注意:

~~~ java
该警告不用管

WARN  org.apache.hadoop.hdfs.DFSClient  - Caught exception 

java.lang.InterruptedException
~~~

2. 查看ui界面

~~~ java
http://hadoop100:8088/cluster
~~~

3. 使用flink run提交任务：

~~~ java
  /export/server/flink/bin/flink run  /export/server/flink/examples/batch/WordCount.jar
~~~

4.  运行完之后可以继续运行其他的小任务

~~~ java
  /export/server/flink/bin/flink run  /export/server/flink/examples/batch/WordCount.jar
~~~

5. 关闭yarn-session：关闭任务需要通过任务号关闭

yarn application -kill application_1609508087977_0005

##### Job分离模式--用的更多

针对每个Flink任务在Yarn上启动一个独立的Flink集群并运行,结束后自动关闭并释放资源,----适用于大任务

1.直接提交job

~~~ java
/export/server/flink/bin/flink run -m yarn-cluster -yjm 1024 -ytm 1024 /export/server/flink/examples/batch/WordCount.jar
~~~

**说明**

~~~ java
-m  jobmanager的地址

-yjm 1024 指定jobmanager的内存信息

-ytm 1024 指定taskmanager的内存信息
~~~

2.查看UI界面

`http://hadoop100:8088/cluster`

## Flink运行时架构

Flink 是一个用于**状态化并行流**处理的分布式系统。它的搭建涉及多个进程，这些进程通常会分布在多台机器上。分布式系统需要应对的常见挑战包括分配和管理集群计算资源，进程协调，持久且高可用的数据存储及故障恢复等。

### flink原理初探

![1621655348577](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/114910-990149.png)

flink客户端不仅会提交应用程序的代码，还会将提交的程序代码转换为数据流图，然后进行一些优化，形成一个数据流图，然后提交给资源管理的节点job manager节点，也就是master，负责整个作业的管理，任务的分配和资源的申请。job manager会将任务分给task mansger节点。task manager是工作节点，工作节点里面是task slot，可以把taskslot看做是线程，taskmanager看做的一个jvm进程，执行的是任务。如果和spark类比,job manager就相当于spark中的driver进程，taskmanager就相当于worker。

- 简单来说在clint上面会生成最初的程序流图，也就是streamGraph，
- 然后把oneToOne的算子进行合并，做一些优化生成jobGraph图，
- 接下来在jobManager上面根据程序中设置的并行度和资源的申请状况生成executorGraph图。
- 将executorGraph落实到具体的taskmanager上面，并将具体的subtask落实到具体的slot中进行运行，这一步是物理的执行图。

> 生成前两部分的图形是在clint端进行的。
>
> 生成后两步的图是在jobmanager上面进行的。

### Flink运行时的组件

![1614333429684](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/222550-73500.png)

Flink 运行时架构主要包括四个不同的组件，它们会在运行流处理应用程序时协同工作：作业管理器（JobManager）、资源管理器（ResourceManager）、任务管理器（TaskManager），以及分发器（Dispatcher）。因为 Flink 是用 Java 和 Scala 实现的，所以所有组件都会运行在Java 虚拟机上。每个组件的职责如下：

**作业管理器（JobManager）**可以类比spark中的driver

- 控制一个应用程序执行的**主进程**，也就是说，每个应用程序(作业)都会被一个不同的JobManager 所控制执行。也就是说提交一个flink程序，那么就由jobmanager来控制执行，提交作业就是提交给jobmanager。它扮演的是集群管理者的角色，负责调度任务、协调checkpoints、协调故障恢复、收集 Job 的状态信息，并管理 Flink 集群中的从节点 TaskManager。
- JobManager 会先接收到要执行的应用程序，这个应用程序会包括：作业图（JobGraph）、逻辑数据流图（logical dataflow graph）和打包了所有的类、库和其它资源的 JAR 包。
- JobManager 会把 JobGraph 转换成一个物理层面的数据流图dataflow图，这个图被叫做“执行图”（ExecutionGraph），包含了所有可以并发执行的任务。
- JobManager 会向资源管理器（ResourceManager）请求执行任务必要的资源，也就是任务管理器（TaskManager）上的插槽（slot）。一旦它获取到了足够的资源，就会将执行图分发到真正运行它们的TaskManager 上。而在运行过程中，JobManager 会负责所有需要中央协调的操作，比如说检查点（checkpoints）的协调。

**任务管理器（TaskManager）**类比spark中的executor

- Flink 中的工作进程。通常在 Flink 中会有多个 TaskManager 运行，每一个 TaskManager都包含了一定数量的插槽（slots）。**插槽的数量限制了 TaskManager 能够执行的任务数量。**
- 启动之后，TaskManager 会向资源管理器注册它的插槽；收到资源管理器的指令后，TaskManager 就会将一个或者多个插槽提供给 JobManager 调用。
- JobManager 就可以向插槽分配任务（tasks）来执行了。在执行过程中，一个 TaskManager 可以跟其它运行同一应用程序的 TaskManager 交换数据。所以交换数据有时候是夸槽，有时候是夸taskmanager。
- 实际负责执行计算的 Worker，在其上执行 Flink Job 的一组 Task；TaskManager
  还是所在节点的管理员，它负责把该节点上的服务器信息比如内存、磁盘、任务运行情况等向 JobManager 汇报。

**客户端**

- 用户在提交编写好的 Flink 工程时，会先创建一个客户端再进行提交，这个客户端就是 Client

**分发器（Dispatcher）**

- Dispatcher可以跨作业运行，它为应用提交提供了REST接口。来让我们提交需要执行的应用。
- 当一个应用被提交执行时，Dispatcher分发器就会启动并将应用移交给一个JobManager。
- Dispatcher也会启动一个Web UI，用来方便地展示和监控作业执行的信息。
- Dispatcher在架构中可能并不是必需的，这取决于应用提交运行的方式。

**资源管理器（ResourceManager）**

- 主要负责管理任务管理器（TaskManager）的插槽（slot,其实就是工作线程），TaskManger 插槽是Flink中定义的处理计算资源单元。
- Flink为不同的环境和资源管理工具提供了不同资源管理器，比如YARN、Mesos、K8s，以及standalone部署。
- 当JobManager申请插槽资源时，ResourceManager会将有空闲插槽的TaskManager分配给JobManager。如果ResourceManager没有足够的插槽来满足JobManager的请求，它还可以向资源提供平台发起会话，以提供启动TaskManager进程的容器。
- 针对不同的环境和资源提供者(resource provider) (如YARN 、Mesos 、Kubernetes 或独立部署) , Flink 提供了不同的ResourceManager 。ResourceManager 负责管理Flink 的处理资源单元一-TaskManager 处理槽。当JobManager 申请TaskManager 处理槽时， ResourceManager 会指示一个拥有空闲处理槽的TaskManager 将其处理槽提供给JobManager 。如果ResourceManager 的处理槽数无怯满足JobManager 的请求，则ResourceManager 可以和资源提供者通信，让它们提供额外容器来启动更多TaskManager 进程。同时， ResourceManager 还负责终止空闲的TaskManager 以释放计算资源。

![1621654816302](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/114018-741117.png)

这个JobManager很像Spark中的Driver进程，负责整个作业的运行，把整个作业分为多个任务，调度任务在TaskManager上面运行。

**各个组件之间的交互**

![1614337366271](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/26/190616-19485.png)

> JobManager<=>driver
>
> TaskManager<=>Executor

### 名词说明

 Dataflow、Operator、Partition、SubTask、Parallelism

1.Dataflow:Flink程序在执行的时候会被映射成一个数据流模型

2.Operator:数据流模型中的每一个操作被称作Operator,Operator分为:Source/Transform/Sink

3.Partition:数据流模型是分布式的和并行的,执行中会形成1~n个分区

4.Subtask:多个分区任务可以并行,每一个都是独立运行在一个线程中的,也就是一个Subtask子任务

5.Parallelism:并行度,就是可以同时真正执行的子任务数/分区数

**说明** ![1621656307856](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/120549-525043.png)

### 任务提交流程

![1614337366271](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/26/190616-19485.png)

注意，我们在flink配置文件中配置的jobmanager是对节点的配置，而在这里的jobmanager实际上是对每一个作业（job)的管理者jobmanager，需要注意区分，提交作业后，jobmanager知道所有的作业信息。

上图是从一个较为高层级的视角，来看应用中各组件的交互协作。如果部署的集群环境不同（例如 YARN，Mesos，Kubernetes，standalone 等），其中一些步骤可以被省略，或是有些组件会运行在同一个 JVM 进程中。

#### on Yarn

具体地，如果我们将 Flink 集群部署到 YARN 上，那么就会有如下的提交流程：

![1614339707142](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/101138-891397.png)

![1621655962978](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621655962978.png)

- Flink 任务提交后，yarn的Client （类似于spark中的driver）向 HDFS 上传 Flink 的 Jar 包和配置，之后向 YarnResourceManager 提交任务，申请运行applicationmaster进程，由applicationmaster来负责这个作业的资源和任务的管理，注册并且申请资源。
- ResourceManager 分配 Container 资源并通知对应的NodeManager 启动 ApplicationMaster，然后该applicationmaster就是该yarn集群上该作业的老大，负责管理作业的资源，ApplicationMaster 启动后加载 Flink 的 Jar 包和配置构建环境，然后启动 JobManager，ApplicationMaster和JobManager一般在同一个节点上面，具体的任务细节由jobmanager来负责。
- 之后 ApplicationMaster 向 ResourceManager申请资源启动 TaskManager ， ResourceManager 分 配 Container 资 源 后 ， 由ApplicationMaster 通 知 资 源 所 在 节 点 的 NodeManager 启 动 TaskManager ，
- NodeManager 加载 Flink 的 Jar 包和配置构建环境并启动 TaskManager，TaskManager启动后向 JobManager 发送心跳包，并等待 JobManager 向其分配任务。

#### Standalone版

![1621662107867](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/134157-303510.png)

### 任务调度原理

![1614340253325](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/222509-288284.png)



客户端不是运行时和程序执行 的一部分，但它用于准备并发送dataflow(JobGraph)给Master(JobManager)，然后，客户端断开连接或者维持连接以等待接收计算结果。

当 Flink 集 群 启 动 后 ， 首 先 会 启 动 一 个 JobManger 和一个或多个的TaskManager。由 Client 提交任务给 JobManager，JobManager 再调度任务到各个TaskManager 去执行，然后 TaskManager 将心跳和统计信息汇报给 JobManager。TaskManager 之间以流的形式进行数据的传输。上述三者均为独立的 JVM 进程。

Client 为提交 Job 的客户端，可以是运行在任何机器上（与 JobManager 环境连通即可）。提交 Job 后，Client 可以结束进程（Streaming 的任务），也可以不结束并等待结果返回。

JobManager 主 要 负 责 调 度 Job 并 协 调 Task 做 checkpoint， 职 责 上 很 像Storm 的 Nimbus。从 Client 处接收到 Job 和 JAR 包等资源后，会生成优化后的执行计划，并以 Task 的单元调度到各个 TaskManager 去执行。

TaskManager 在启动的时候就设置好了槽位数（Slot），每个 slot 能启动一个Task，Task 为线程。从 JobManager 处接收需要部署的 Task，部署启动后，与自己的上游建立 Netty 连接，接收数据并处理。

1. 怎样实现并行计算？

高吞吐量就是由我们的集群保证的，对于我们的每一个任务，都可以设置一个并行度，然后拆分成多个任务由task去执行，相当于多个线程同时去执行我们的任务，就相当于并行计算。不同的slot就是执行不同的任务，slot其实就是一个线程

2. 并行的任务，需要占用多少slot？

slot的个数个每一步的操作设置的slot个数有关

3. 一个流处理程序，到底包含多少个任务？

### Operator传递模式

数据在两个operator(算子)之间传递的时候有两种模式：

1. One to One模式：

两个operator用此模式传递的时候，会保持数据的分区数和数据的排序；如上图中的Source1到Map1，它就保留的Source的分区特性，以及分区元素处理的有序性。--类似于Spark中的窄依赖

2. Redistributing 模式：

这种模式会改变数据的分区数；每个一个operator subtask会根据选择transformation把数据发送到不同的目标subtasks,比如keyBy()会通过hashcode重新分区,broadcast()和rebalance()方法会随机重新分区。--类似于Spark中的宽依赖

### 并行度（Parallelism）

- Flink 程序的执行具有并行、分布式的特性。
- 在执行过程中，一个流（ stream） 包含一个或多个分区（ stream partition） ，而每一个算子（ operator）可以包含一个或多个子任务（ operator subtask） ，这些子任务在不同的线程、不同的物理机或不同的容器中彼此互不依赖地执行。 
- 一个特定算子的子任务（ subtask） 的个数被称之为其并行度（ parallelism） 。一般情况下， 一个流程序的并行度，可以认为就是其所有算子中最大的并行度。一个程序中，不同的算子可能具有不同的并行度。 

![1621656307856](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/120549-525043.png)

- 一个特定算子的 子任务（subtask）的个数被称之为其并行度（parallelism）。一般情况下，一个 stream 的并行度，可以认为就是其所有算子中最大的并行度。

Stream 在算子之间传输数据的形式可以是 one-to-one(forwarding)的模式也可以是 redistributing 的模式，具体是哪一种形式，取决于算子的种类 

分区上的一系列算子操作叫做子任务。

One-to-one： stream(比如在 source 和 map operator 之间)维护着分区以及元素的顺序。那意味着 map 算子的子任务看到的元素的个数以及顺序跟 source 算子的子任务生产的元素的个数、顺序相同， map、 fliter、 flatMap 等算子都是 one-to-one 的对应关系。 多个一对一的算子可以合并操作形成一个操作链条，任务链之间可以并行执行，每一个操作连或者单个算子叫做任务，task下面可以有多个子任务，**类似于 spark 中的窄依赖 **

Redistributing： stream(map()跟 keyBy/window 之间或者 keyBy/window 跟 sink之间)的分区会发生改变。每一个算子的子任务依据所选择的 transformation 发送数据到不同的目标任务。例如， keyBy() 基于 hashCode 重分区、 broadcast 和 rebalance会随机重新分区，这些算子都会引起 redistribute 过程，而 redistribute 过程就类似于Spark 中的 shuffle 过程，**类似于 spark 中的宽依赖**

![1621656757723](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/121346-275584.png) 

### 任务链（Operator Chains） 

相同并行度的 one to one 操作， Flink 这样相连的算子链接在一起形成一个 task，原来的算子成为里面的一部分。将算子链接成 task 是非常有效的优化：它能减少线程之间的切换和基于缓存区的数据交换，在减少时延的同时提升吞吐量。链接的行为可以在编程 API 中进行指定。 

![1614392790996](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/131819-860063.png)

![1621656757723](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/121346-275584.png)

客户端在提交任务的时候会对Operator进行优化操作，能进行合并的Operator会被合并为一个Operator，

合并后的Operator称为Operator chain，实际上就是一个执行链，每个执行链会在TaskManager上一个独立的线程中执行--就是SubTask。

### TaskManager 和 Slots

Flink 中每一个 worker(TaskManager)都是一个 JVM 进程，它可能会在独立的线程上执行一个或多个 subtask。为了控制一个 worker 能接收多少个 task， worker 通过 task slot 来进行控制（一个 worker 至少有一个 task slot）。 

每个 task slot 表示 TaskManager 拥有资源的一个固定大小的子集。假如一个TaskManager 有三个 slot，那么它会将其管理的内存分成三份给各个 slot。资源 slot化意味着一个 subtask 将不需要跟来自其他 job 的 subtask 竞争被管理的内存，取而代之的是它将拥有一定数量的内存储备。需要注意的是，这里不会涉及到 CPU 的隔离， slot 目前仅仅用来隔离 task 的受管理的内存。 

通过调整 task slot 的数量，允许用户定义 subtask 之间如何互相隔离。如果一个TaskManager 一个 slot，那将意味着每个 task group 运行在独立的 JVM 中（该 JVM可能是通过一个特定的容器启动的），而一个 TaskManager 多个 slot （线程）意味着更多的subtask 可以共享同一个 JVM。而在同一个 JVM 进程中的 task 将共享 TCP 连接（基于多路复用）和心跳消息。它们也可能共享数据集和数据结构，因此这减少了每个task 的负载。 

![1621657225993](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/122027-990485.png)

- Flink 中每一个 TaskManager 都是一个JVM进程，它可能会在独立的线程上执行一个或多个子任务，其中slot的个数就代表并发线程执行的个数。
- 为了控制一个 TaskManager 能接收多少个 task， TaskManager 通过 taskslot 来进行控制（一个 TaskManager 至少有一个 slot）
- 也就是说每一个线程都要单独分配slot资源去执行

![1614386765807](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/084627-278044.png)

### Sharing Slot

![1621657450129](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/122412-10545.png)

草共享可以避免线程的创建和销毁带来的开销，可以看做是线程池。

- 默认情况下，Flink 允许子任务共享 slot，即使它们是不同任务的子任务。 这样的结果是，一个 slot 可以保存作业的整个管道。相同的任务的子任务不能放在同一个slot中执行，必须是不同的任务的子任务可以共享slot,相同组的线程可以占用同一个slot,不同组的线程占据不同的slot.
- 推荐根据cpu的核心数设置slot的个数
- Task Slot 是静态的概念，是指 TaskManager 具有的并发执行能力
- 不同的slot共享组占据不同的slot。

Task Slot 是静态的概念，是指 TaskManager 具有的并发执行能力，可以通过参数 taskmanager.numberOfTaskSlots 进行配置； 而并行度 parallelism 是动态概念，即 TaskManager 运行程序时实际使用的并发能力，可以通过参数 parallelism.default进行配置。

也就是说，假设一共有 3 个 TaskManager，每一个 TaskManager 中的分配 3 个TaskSlot，也就是每个 TaskManager 可以接收 3 个 task，一共 9 个 TaskSlot，如果我们设置 parallelism.default=1，即运行程序默认的并行度为 1， 9 个 TaskSlot 只用了 1个，有 8 个空闲，因此，设置合适的并行度才能提高效率。

**允许插槽共享有两个主要好处：**

- 资源分配更加公平，如果有比较空闲的slot可以将更多的任务分配给它。

- 有了任务槽共享，可以提高资源的利用率。

**注意:**

- slot是静态的概念，是指taskmanager具有的并发执行能力

- parallelism是动态的概念，是指程序运行时实际使用的并发能  

### 并行子任务的分配

一个 TaskManager 允许同时执行多个任务。这些任务可以属于同一个算子(数据并行) ，也可以是不同算子(任务并行) ，甚至还可以来自不同的应用(作业并行) 0 TaskManager 通过提供固定数量的处理槽来控制可以并行执行的任务数。 每个处理槽可以执行应用的一部分，即算子的一个并行任务。图 3-2展示了TaskManager、处理槽、任务以及算子之间的关系。 

![1614694340976](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/221358-388017.png)

左侧的 JobGraph (应用的非并行化表示)包含了 5 个算子，其中算子 A 和 C是数据橱，算子 E 是数据汇。算子 C 和 E 的并行度为 2 ，其余算子的并行度为 4 。由于算子最大并行度是 4 ，因此应用若要执行则至少需要 4 个处理槽。如果每个 TaskManager 内有两个处理槽，则运行两个 TaskManager 即可满足该需求。
JobManager 将 JobGraph "展开成" ExecutionGraph 并把任务分配到 4 个空闲处理槽。对于井行度为 4 的算子，其任务会每个处理槽分配一个。其余两个算子 C 和 E 的任务会分别放到处理槽1.1、 2. 1 和处理槽 1.2 、 2 .2 中。将任务以切片的形式调度至处理槽中有一个好处: TaskManager 中的多个任务可以在同一进程 内高效地执行数据交换而无须访问网络。然而，任务过于集中也会使TaskManager 负载变高，继而可能导致性能下降。 

###  程序与数据流（DataFlow）

![1614390712367](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/095152-707756.png)

- 所有的 Flink 程序都是由三部分组成的： Source 、 Transformation 和 Sink。 
- Source 负责读取数据源， Transformation 利用各种算子进行处理加工， Sink 负责输出。 
- 在运行时， Flink 上运行的程序会被映射成“逻辑数据流”（ dataflows） ，它包含了这三部分。 每一个 dataflow 以一个或多个 sources 开始以一个或多个 sinks 结束。 dataflow 类似于任意的有向无环图（ DAG）。在大部分情况下，程序中的转换运算（ transformations） 跟 dataflow 中的算子（ operator） 是一一对应的关系，但有时候，一个 transformation 可能对应多个 operator。

![1614392321655](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/101842-896076.png)

### 执行图（ExecutionGraph） 

由 Flink 程序直接映射成的数据流图是 StreamGraph，也被称为逻辑流图，因为它们表示的是计算逻辑的高级视图。为了执行一个流处理程序， Flink 需要将逻辑流图转换为物理数据流图（也叫执行图） ，详细说明程序的执行方式。 

Flink 中的执行图可以分成四层： StreamGraph -> JobGraph -> ExecutionGraph ->物理执行图。 

**执行图生成过程**

![1621657844935](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/123047-672135.png)

![1621662703581](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/135542-374085.png)

**原理介绍**

- Flink执行executor会自动根据程序代码生成DAG数据流图
- Flink 中的执行图可以分成四层：StreamGraph -> JobGraph -> ExecutionGraph -> 物理执行图。
-  **StreamGraph**：是根据用户通过 Stream API 编写的代码生成的最初的图。表示程序的拓扑结构。
-  **JobGraph**：StreamGraph经过优化后生成了 JobGraph，提交给 JobManager 的数据结构。主要的优化为，将多个符合条件的节点 chain 在一起作为一个节点，这样可以减少数据在节点之间流动所需要的序列化/反序列化/传输消耗。
- **ExecutionGraph**：JobManager 根据 JobGraph 生成ExecutionGraph。ExecutionGraph是JobGraph的并行化版本，是调度层最核心的数据结构。
- **物理执行图**：JobManager 根据 ExecutionGraph 对 Job 进行调度后，在各个TaskManager 上部署 Task 后形成的“图”，并不是一个具体的数据结构。

**简单理解：**

- StreamGraph：最初的程序执行逻辑流程，也就是算子之间的前后顺序--在Client上生成

- JobGraph：将OneToOne的Operator合并为OperatorChain--在Client上生成

- ExecutionGraph：将JobGraph根据代码中设置的并行度和请求的资源进行并行化规划!--在JobManager上生成

- 物理执行图：将ExecutionGraph的并行计划,落实到具体的TaskManager上，将具体的SubTask落实到具体的TaskSlot内进行运行。

## Flink流处理API

Flink中的流一般分为有边界的流和无边界的流，有边界的流就是批处理。

### 概述

**API层次结构**

![1621574171278](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/135540-53159.png)

- Table api一般是面向对象进行编程

- DataStream是面向流和批的处理，DataSet api是面向批处理的，现在已经被淘汰。

入门案例使用dataSet面向批处理写，后面使用流处理和批处理一体的DataStream

**语法说明**

![1621574518154](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/132214-880288.png)

**flink编程模型**

![1621574558280](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/132241-692408.png)

- Data Source是加载数据
- Transformations是处理数据
- Data Sink是输出结果

**编程模型说明**

![1621574665781](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/132428-919301.png)

### DataSet介绍

**批处理执行环境类ExecutionEnvironment**

~~~ java
@Public
public class ExecutionEnvironment {}
//ExecutionEnvironment是一个工具类，是操作DataSet批处理的一个工具类
~~~

**DataSet版本wordcount**

~~~ java

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.*;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.util.Collector;

public class Test01 {

    public static void main(String[] args) throws Exception {

//        DataSet api使用
//        1 准备环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

//        2 加载数据
        DataSet<String> dataSource = env.readTextFile("D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt");
//        3 处理数据
        FlatMapOperator<String, String> flatmapData = dataSource.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
//               value表示每一行数据。现在对每一行数据进行切割
                String[] s = value.split(" ");
//                遍历切割好的单词，返回单词表
                for (String str : s) {
                    out.collect(str);
                }
            }
        });
//        转换结构
        MapOperator<String, Tuple2<String, Integer>> map = flatmapData.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return new Tuple2<>(value, 2);
            }
        });

//        统计单词
        UnsortedGrouping<Tuple2<String, Integer>> grouped = map.groupBy(0);

//        对最终的结果进行聚合操作
        AggregateOperator<Tuple2<String, Integer>> sum = grouped.sum(1);

//        4 输出结果
        sum.print();

    }
}
~~~

**DataSet的api继承结构**

![1621576716798](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/135839-951899.png)

### 如何写Flink程序

下面的程序全部是基于DataStream的API。

构建一个典型的 Flink 流式应用需要以下几步: 

1. 设置执行环境。
2. 从数据源中读取一 条或多条流。**source**
3. 通过一 系列流式转换来实现应用逻辑。**transform**
4. 选择性地将结果输出到 一个或多个数据汇中。**sink**
5. 执行程序。 

**DataStream流处理类图**

![1614430440646](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/205401-717813.png)

继承结构图

![1614474745877](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/28/091227-963313.png)

**流处理api的分类--Flink程序的构成**

![1614392944297](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/213317-534420.png)

**Flink程序的执行**

- Flink 程序都是通过**延迟计算** (lazily execute) 的方式执行。也就是说，那些创建数据掘和转换操作的 API 调用不会立即触发数据处理，而只会在执行环境中构建一个执行计划。计划中包含了从环境创建的流式数据源以及应用于这些数据源之上的一系列转换。只有在调用 execute ()方法时，系统才会触发程序执行。
- 构建完的计划会被转成 JobGraph 并提交至 JobManager 执行。根据执行环境类型的不同，系统可能需要将 JobGraph 发送到作为本地线程启动的JobManager 上(本地执行环境) ，也可能会将其发送到远程 JobManager 上。如果是后者，除 JobGraph 之外，我们还要同时提供包含应用所需全部类和依赖的 JAR 包。 

#### 流处理wordcount案例

~~~ java
public class Test02 {

    /**
     * DatasTREAM既支持流处理，也支持批处理，如何区分流处理和批处理？
     * @param args
     */
    public static void main(String[] args) throws Exception {

//        1 获取执行环境,获取的是流处理的env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(2);

//        如果想做批处理，可以进行下面的设置,RuntimeExecutionMode是一个枚举类，里面有三个值
        env.setRuntimeMode(RuntimeExecutionMode.BATCH);//使用批处理
//        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);//根据数据源自动进行选择
//        env.setRuntimeMode(RuntimeExecutionMode.STREAMING);//流处理

//      读取数据
        DataStreamSource<String> data = env.readTextFile("D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt");
//        操作数据
        SingleOutputStreamOperator<String> flatmapData = data.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
                String[] s = value.split(" ");
                for (String str:s) {
                    out.collect(str);
                }
            }
        });

        SingleOutputStreamOperator<Tuple2<String, Integer>> mapData = flatmapData.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return  Tuple2.of(value,1);
            }
        });

//        数据聚合操作
//        KeyedStream<Tuple2<String, Integer>, Tuple> sum = mapData.keyBy(0);

//        第一个参数表示输入的类型，第二个参数表示提取key 的类型
        KeyedStream<Tuple2<String, Integer>, String> sum = mapData.keyBy(new KeySelector<Tuple2<String, Integer>, String>() {
            @Override
            public String getKey(Tuple2<String, Integer> value) throws Exception {
                return value.f0;
            }
        });

//        SingleOutputStreamOperator<Tuple2<String, Integer>> sum1 = sum.sum(1);

        sum.print();
//        启动程序并且等待结束
        env.execute();
//        在后续的开发中，把一切的数据源都看做是流数据

    }
}
~~~

> DataStream是流处理和批处理一体的api，原来的dataSet批处理api已经废弃，可以通过控制流处理api的参数进行批处理操作。

### Environment 

##### getExecutionEnvironment 

创建一个执行环境，表示当前执行程序的上下文。 **如果程序是独立调用的，则此方法返回本地执行环境；如果从命令行客户端调用程序以提交到集群，则此方法返回此集群的执行环境**，也就是说， getExecutionEnvironment 会根据查询运行的方式决定返回什么样的运行环境，是最常用的一种创建执行环境的方式。 

**批处理执行环境ExecutionEnvironment**

~~~ java
public class ExecutionEnvironment {}
~~~

![1614393681958](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/104122-239154.png)

**流处理执行环境StreamExecutionEnvironment**

~~~ java
@Public
public class StreamExecutionEnvironment {}
//StreamExecutionEnvironment是流处理的执行环境
~~~

![1614393750522](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/104443-618574.png)

**案例**

~~~ java
public class EnvTest {

    public static void main(String[] args) {
//        获取批处理执行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
//        获取流处理的执行环境
        StreamExecutionEnvironment senv = StreamExecutionEnvironment.getExecutionEnvironment();
    }
}
~~~

##### createLocalEnvironment 

返回本地流式执行环境，需要在调用时指定默认的并行度。 

~~~ java
public class CLENV {

    public static void main(String[] args) {

//        设置并行度是2，获取的是流处理的本地执行环境
        LocalStreamEnvironment lsenv = StreamExecutionEnvironment.createLocalEnvironment(2);
    }
}
~~~

##### createRemoteEnvironment 

返回集群执行环境，远程流式执行环境，将 Jar 提交到远程服务器。需要在调用时指定 JobManager的 IP 和端口号，并指定要在集群中运行的 Jar 包。 

~~~ java
public class RMENV {

    public static void main(String[] args) {

//        创建集群环境，设置主机，端口号和并行度，获取的是流处理的远程执行环境
        StreamExecutionEnvironment rmenv = StreamExecutionEnvironment.createRemoteEnvironment("hadoop100", 888, 2);
    }
}
~~~

从上面我们可以看到，本地环境类，集群环境类都是StreamExecutionEnvironment的子类。

### Source 

##### 从集合读取数据 

~~~ java
public class ReadFromCollection {

    public static void main(String[] args) throws Exception{

//        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
//        设置全局的并行度 ,设置1的话就是单线程执行
        env.setParallelism(1);

//        从集合中读取数据
        DataStream<SensorReading> ds = env.fromCollection(Arrays.asList(
                new SensorReading("sensor_1", 1547718199L, 35.8),
                new SensorReading("sensor_6", 1547718201L, 15.4),
                new SensorReading("sensor_7", 1547718202L, 6.7),
                new SensorReading("sensor_10", 1547718205L, 38.1)
        ));

//        从元素中读取数据
        DataStreamSource<Integer> ids = env.fromElements(1, 2, 3, 4, 5);

//        打印输出得到的结果
        ds.print("data");


//        如果在这里设置并行度是1，那么就会按照顺序输出，这里设置的并行度只是设置当前任务的并行度，也可以设置全局的并行度
//        ids.print("int").setParallelism(1);
        ids.print("int");
//        执行任务
        env.execute("作业一");

    }
}
~~~

**案例2**

~~~ java
public class Test03 {

    public static void main(String[] args) throws Exception {

//        TODO env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
//        如果没有做设置的话，默认进行的是流处理

//        TODO source
        /**
         * 基于集合的source
         */
//        fromElements():可变参数
        DataStreamSource<String> ds = env.fromElements("spark", "hadoop", "flink");

//        基于集合创建source
//        fromCollection()：参数是一个集合
        DataStreamSource<String> dsc = env.fromCollection(Arrays.asList("spark", "hadoop", "flink"));

//         基于1-100生成一个集合
        DataStreamSource<Long> dsg = env.generateSequence(1, 100);
        DataStreamSource<Long> dsg1 = env.fromSequence(1, 100);

//        TODO sink
        ds.print();
        dsc.print();
        dsg.print();
        dsg1.print();

//        TODO execute()
        env.execute();
    }
}
~~~

##### 从文件中读取数据

~~~ java
public class ReadFromFile {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

//        打印输出
        sdss.print();

//        执行程序
        env.execute();

    }
}
~~~

**案例二**

~~~ java
public class Test04 {

    public static void main(String[] args) throws Exception {
        /**
         * 基于文件或者文件夹读取
          */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStream<String> dsf = env.readTextFile("D:\\soft\\idea\\work\\work08\\hmlen\\src\\main\\resources\\words.txt");
//        读取文件夹下面的所有文件
        DataStream<String> dsFile = env.readTextFile("D:\\soft\\idea\\work");
        env.execute();

    }
}
~~~

##### 以 kafka 消息队列的数据作为来源 

需要引入 kafka 连接器的依赖 

~~~ java
  <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka-0.11_2.12</artifactId>
            <version>1.10.1</version>- //flink的版本
        </dependency>
// 0.11是kafka的版本，2.12是scala的版本
~~~

**案例**

~~~ java
public class ReadFromKafka {

    public static void main(String[] args) throws Exception {

        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();


        // kafka 配置项
        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "localhost:9092");
        properties.setProperty("group.id", "consumer-group");
        properties.setProperty("key.deserializer",
                "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer",
                "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("auto.offset.reset", "latest");

//        添加数据源

        DataStreamSource<String> sdss = env.addSource(new FlinkKafkaConsumer011<String>("sensor", new SimpleStringSchema(), properties));
//sensor代表kafka组
        
        sdss.print();

//        执行程序
        env.execute();

    }
}

//案例er
public class Test14 {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(2);

//        添加kafka数据源
//        准备配置文件
        Properties props = new Properties();
        props.setProperty("bootstrap.servers", "node1:9092");//生产者组的ip
        props.setProperty("group.id", "flink");//消费者组
        props.setProperty("auto.offset.reset","latest");//设置offset,如果有offset记录，那么就从offset记录开始消费，没有的话从最新的消息消费
        props.setProperty("flink.partition-discovery.interval-millis","5000");//会开启一个后台线程每隔5s检测一下Kafka的分区情况，动态分区的检测
        props.setProperty("enable.auto.commit", "true");//自动提交
        props.setProperty("auto.commit.interval.ms", "2000");//自动提交时间间隔，会提交到默认的主题
//        创建kafka source
        FlinkKafkaConsumer<String> flink_kafka = new FlinkKafkaConsumer<>("flink_kafka", new SimpleStringSchema(), props);
//      使用source
        DataStreamSource<String> source = env.addSource(flink_kafka);

        env.execute();

    }
}
~~~

##### socket数据源

- 启动socket：nc -lk port

~~~ java
public class Test05 {

    public static void main(String[] args) throws Exception {
        /**
         * 基于socket读取文件
          */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        基于socket读取数据
        DataStreamSource<String> socket = env.socketTextStream("DESKTOP-56SGKD3", 9999);

        SingleOutputStreamOperator<String> words = socket.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
                String[] s = value.split(" ");
                for (String str : s) {
                    out.collect(str);
                }
            }
        });

        SingleOutputStreamOperator<Tuple2<String, Integer>> mapS = words.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return new Tuple2<>(value, 1);
            }
        });

        KeyedStream<Tuple2<String, Integer>, Tuple> keyByDtat = mapS.keyBy(1);

        keyByDtat.print();

        env.execute();
    }
}
public class Test02 {

    /**
     * DatasTREAM既支持流处理，也支持批处理，如何区分流处理和批处理？
     * @param args
     */
    public static void main(String[] args) throws Exception {

//        1 获取执行环境,获取的是流处理的env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

//        如果想做批处理，可以进行下面的设置,RuntimeExecutionMode是一个枚举类，里面有三个值
        env.setRuntimeMode(RuntimeExecutionMode.BATCH);//使用批处理
//        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);//根据数据源自动进行选择
//        env.setRuntimeMode(RuntimeExecutionMode.STREAMING);//流处理

//      读取数据
        DataStreamSource<String> data = env.readTextFile("D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt");
//        操作数据
        SingleOutputStreamOperator<String> flatmapData = data.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
                String[] s = value.split(" ");
                for (String str:s) {
                    out.collect(str);
                }
            }
        });

        SingleOutputStreamOperator<Tuple2<String, Integer>> mapData = flatmapData.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return  Tuple2.of(value,1);
            }
        });

//        数据聚合操作
//        KeyedStream<Tuple2<String, Integer>, Tuple> sum = mapData.keyBy(0);

//        第一个参数表示输入的类型，第二个参数表示提取key 的类型,根据返回的key对数据进行分组
        KeyedStream<Tuple2<String, Integer>, String> sum = mapData.keyBy(new KeySelector<Tuple2<String, Integer>, String>() {
            @Override
            public String getKey(Tuple2<String, Integer> value) throws Exception {
                return value.f0;
            }
        });

//        SingleOutputStreamOperator<Tuple2<String, Integer>> sum1 = sum.sum(1);

        sum.print();
//        启动程序并且等待结束
        env.execute();
//        在后续的开发中，把一切的数据源都看做是流数据

    }
}

~~~

##### 自定义 Source 

可以添加多种数据源

![1621927806658](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/153010-218291.png)

除了以上的 source 数据来源，我们还可以自定义 source。需要做的，只是传入一个 SourceFunction 就可以。具体调用如下： 

~~~ java
//我们可以添加自己定义的数据源
DataStream<SensorReading> dataStream = env.addSource( new MySensor());
~~~

**自定义数据源**

![1621752852591](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/152941-131078.png)

**代码演示**

~~~ java
//自定义数据源，需要去实现SourceFunction接口，接口中的泛型表示产生的数据类型
//此接口有两个方法，
public static class MySeneorFun implements SourceFunction<SensorReading>{
//定义一个标志位，控制数据的产生
        private boolean running=true;


        public void run(SourceContext<SensorReading> ctx) throws Exception {

//            定义随机数发生器
            Random random = new Random();

//            设置10个传感器温度的初始值
            HashMap<String, Double> temp = new HashMap<String,Double>();

            for (int i = 0; i <10; i++) {
//                nextGaussian按照高斯分布生成随机数，也就是正泰分布
                temp.put("sensor_"+(i+1),random.nextGaussian()*20);
            }

//            产生数据
            while (running){
//                使用collect方法生成我们的额数据
                for (String id:temp.keySet()) {
//                    使得温度分布更加随机
                    Double ranTem=temp.get(id)+random.nextGaussian();
                    temp.put(id,ranTem);
                    ctx.collect(new SensorReading(id,System.currentTimeMillis(),temp.get(id)));
                }
//              控制输出的频率
                Thread.sleep(3);
            }

        }

        public void cancel() {

            running=false;
        }
    }
~~~

**案例**

~~~ java
public class MyUDF {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        //env.setParallelism(1);

//        自定义从源中读取数据
//        只要自定义的source执行起来之后，就会调用run方法
        DataStreamSource<SensorReading> sdss = env.addSource(new MySeneorFun());

//        打印输出
        sdss.print();

//        执行程序
        env.execute();

    }

    public static class MySeneorFun implements SourceFunction<SensorReading>{
//定义一个标志位，控制数据的产生
        private boolean running=true;

        public void run(SourceContext<SensorReading> ctx) throws Exception {

//            定义随机数发生器
            Random random = new Random();

//            设置10个传感器温度的初始值
            HashMap<String, Double> temp = new HashMap<String,Double>();

            for (int i = 0; i <10; i++) {
//                nextGaussian按照高斯分布生成随机数，也就是正泰分布
                temp.put("sensor_"+(i+1),random.nextGaussian()*20);
            }

//            产生数据
            while (running){
//                使用collect方法生成我们的额数据
                for (String id:temp.keySet()) {
//                    使得温度分布更加随机
                    Double ranTem=temp.get(id)+random.nextGaussian();
                    temp.put(id,ranTem);
                    ctx.collect(new SensorReading(id,System.currentTimeMillis(),temp.get(id)));
                }
//              控制输出的频率
                Thread.sleep(3);
            }

        }

        public void cancel() {

            running=false;
        }
    }
}
~~~

**案例二**

~~~ java
public class Test06 {
    /**
     * Author itcast
     * Desc
     * 需求
     * 每隔1秒随机生成一条订单信息(订单ID、用户ID、订单金额、时间戳)
     * 要求:
     * - 随机生成订单ID(UUID)
     * - 随机生成用户ID(0-2)
     * - 随机生成订单金额(0-100)
     * - 时间戳为当前系统时间
     * <p>
     * API
     * 一般用于学习测试,模拟生成一些数据
     * Flink还提供了数据源接口,我们实现该接口就可以实现自定义数据源，不同的接口有不同的功能，分类如下：
     * SourceFunction:非并行数据源(并行度只能=1)
     * RichSourceFunction:多功能非并行数据源(并行度只能=1)
     * ParallelSourceFunction:并行数据源(并行度能够>=1)
     * RichParallelSourceFunction:多功能并行数据源(并行度能够>=1)--后续学习的Kafka数据源使用的就是该接口
     */


    public static void main(String[] args) throws Exception {
        /**
         * 基于socket读取文件
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);
        //2.Source
        DataStream<Order> orderDS = env
                .addSource(new MyOrderSource())//添加自定义的数据源
                .setParallelism(2);//可以自己设置并行度

        //3.Transformation

        //4.Sink
        orderDS.print();//每一次打印两条数据，因为并行度是2
        //5.execute
        env.execute();
    }
//    Data注释可以直接使用get set方法
    @Data
    @AllArgsConstructor //添加全参构造器
    @NoArgsConstructor //添加空参数构造器
    public static class Order {
        private String id;
        private Integer userId;
        private Integer money;
        private Long createTime;
    }

    public static class MyOrderSource extends RichParallelSourceFunction<Order> {
        private Boolean flag = true;

//        执行并且生成数据
        @Override
        public void run(SourceContext<Order> ctx) throws Exception {
            Random random = new Random();
            while (flag) {
                Thread.sleep(1000);
//                订单id
                String id = UUID.randomUUID().toString();
//                用户id
                int userId = random.nextInt(3);
                int money = random.nextInt(101);
                long createTime = System.currentTimeMillis();
                ctx.collect(new Order(id, userId, money, createTime));
            }
        }

        //取消任务/执行cancle命令的时候执行
        @Override
        public void cancel() {
            flag = false;
        }
    }
}
~~~

**lombok的使用**

添加依赖

~~~ java
 <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.2</version>
            <scope>provided</scope>
</dependency>
~~~

安装插件

![1621754220104](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621754220104.png)

**SourceFunction接口**

~~~ java
public interface SourceFunction<T> extends Function, Serializable {

	
  //用来产生我们的数据的方法，我们可以在这里写产生数据的方法
	void run(SourceContext<T> ctx) throws Exception;

  //控制产生数据的方法
	void cancel();

	@Public // Interface might be extended in the future with additional methods.
	interface SourceContext<T> {

		void collect(T element);

		@PublicEvolving
		void collectWithTimestamp(T element, long timestamp);

		@PublicEvolving
		void emitWatermark(Watermark mark);

		@PublicEvolving
		void markAsTemporarilyIdle();

		Object getCheckpointLock();

		void close();
	}
}

~~~

**类图**

![1621754486366](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/152129-113162.png)

自定义source的时候有多重source，可以根据自己的选择继承使用。

##### Mysql数据源

- 需求:

实际开发中,经常会实时接收一些数据,要和MySQL中存储的一些规则进行匹配,那么这时候就可以使用Flink自定义数据源从MySQL中读取数据

那么现在先完成一个简单的需求:

从MySQL中实时加载数据

要求MySQL中的数据有变化,也能被实时加载出来

~~~ java
public class Test07 {
    /**
     * Desc
     * 需求:
     * 实际开发中,经常会实时接收一些数据,要和MySQL中存储的一些规则进行匹配,那么这时候就可以使用Flink自定义数据源从MySQL中读取数据
     * 那么现在先完成一个简单的需求:
     * 从MySQL中实时加载数据
     * 要求MySQL中的数据有变化,也能被实时加载出来
     */


    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        自定义数据源，并且设置并行度是1
        DataStreamSource<Student> data = env.addSource(new MySqlSource()).setParallelism(1);

        data.print();

        env.execute();

    }


    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class Student{
        private Integer id;
        private String name;
        private Integer age;
    }

    public static class MySqlSource extends RichParallelSourceFunction<Student>{

        private boolean flag=true;
        private Connection connection=null;
        private PreparedStatement ps=null;
        private ResultSet rs=null;

        @Override
        public void open(Configuration parameters) throws Exception {
//            这个方法只执行一次，适合开启资源，比如这里打开数据库连接
//            /获取数据库连接地址
            connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "root");
            String sql="select * from t_student";
//            执行sql语句
            ps = connection.prepareStatement(sql);
        }

        @Override
        public void run(SourceContext<Student> ctx) throws Exception {
            while (flag){
//                每5秒钟进行一次查询
                rs=ps.executeQuery();
                while (rs.next()){
                    int id = rs.getInt("id");
                    String name = rs.getString("name");
                    int age = rs.getInt("age");
                    ctx.collect(new Student(id,name,age));
                }
                TimeUnit.SECONDS.sleep(5);
            }

        }

        @Override
        public void cancel() {
//            接受到cancel命令时候，取消数据的生成
            flag=false;
        }

        @Override
        public void close() throws Exception {
//            关闭资源
            if(connection != null)
                connection.close();
            if(ps != null)
                ps.close();
            if(rs != null){
                rs.close();
            }
        }
    }
}
~~~

**测试数据**

~~~ java
CREATE TABLE `t_student` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(255) DEFAULT NULL,
    `age` int(11) DEFAULT NULL,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;

INSERT INTO `t_student` VALUES ('1', 'jack', '18');
INSERT INTO `t_student` VALUES ('2', 'tom', '19');
INSERT INTO `t_student` VALUES ('3', 'rose', '20');
INSERT INTO `t_student` VALUES ('4', 'tom', '19');
INSERT INTO `t_student` VALUES ('5', 'jack', '18');
INSERT INTO `t_student` VALUES ('6', 'rose', '20');
~~~

### Transform （转换）

流式转换以一个或多个数据流为输入，井将它们转换成一个或多个输出流。完成一个 DataStream API 程序在本质

上可以归结为:**通过组合不同的转换来创建一个满足应用逻辑的 Dataflow 图**。 

大多数流式转换都是基于用户自定义函数来完成的。这些函数封装了用户应用逻辑，指定了输入流的元素将如何转

换为输出流的元素。函数可以通过实现某个特定转换的接口类来定义 ，函数接口规定了用户需要实现的转换方法

#### 算子分类

DataStream API的转换分为四类: 

1. 作用于单个事件的基本转换。 

2. 针对相同键值事件的 KeyedStream 转换。

3. 将多条数据流合并为一 条或将一条数据流拆分成多条流的转换。

4. 对流中的事件进行重新组织的分发转换 。

![1621852286978](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/183132-651854.png)

**操作概览**

![1621852330281](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/183220-69874.png)

整体来说，流式数据上的操作可以分为四类。

- 第一类是对于单条记录的操作，比如筛除掉不符合要求的记录（Filter 操作），或者将每条记录都做一个转换（Map 操作）

- 第二类是对多条记录的操作。比如说统计一个小时内的订单总成交量，就需要将一个小时内的所有订单记录的成交量加到一起。为了支持这种类型的操作，就得通过 Window 将需要的记录关联到一起进行处理

- 第三类是对多个流进行操作并转换为单个流。例如，多个流可以通过 Union、Join 或 Connect 等操作合到一起。这些操作合并的逻辑不同，但是它们最终都会产生了一个新的统一的流，从而可以进行一些跨流的操作。

- 最后， DataStream 还支持与合并对称的拆分操作，即把一个流按一定规则拆分为多个流（Split 操作），每个流是之前流的一个子集，这样我们就可以对不同的流作不同的处理。

#### 基本转换

基本转换会单独处理每个事件 ，这意味着每条输出记录都由单条输入记录所生成。常见的基本转换函数有:简单的值转换，记录拆分或过滤等 ，map()，flatMap()，Filter()，这三个是简单的转换算子

##### map()

通过调用 DataStream.map() 方法可以指定 map 转换产生 一 个新的DataStream。该转换将每个到来的事件传给一个用户自定义的映射器( user defined mapper) ，后者针对每个输入只会返回 一个(可能类型发生改变的)输出事件。图所示的 map 转换会将每个方形输入转换为圆形。 

![1614424509615](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/213417-791672.png)

**MapFunction接口**

~~~ java
//mapFunction是一个接口，只需要实现其中的方法就可以
//MapFunction 的两个类型参数分别是输入事件和输出事件的类型 ，它们可以通过 MapFuncti on 接口来指定。 i亥接口的 map() 方怯将每个输入事件转换为一个输出事件:
public interface MapFunction<T, O> extends Function, Serializable {
    O map(T var1) throws Exception;
}
//map()方法是针对每一个元素进行的操作
~~~

**案例**

~~~ java
public class TransformBase01 {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");


//        对读进来的数据做基本的转换
//     1， map():把传进来的数据转换为长度输出
//        MapFunction的泛型，String表示输入时候的泛型，Integer表示输出时候的泛型
        DataStream<Integer>mapStream=sdss.map(new MapFunction<String, Integer>() {
//            使用一个匿名类
//            重写map方法进行转换
            public Integer map(String s) throws Exception {
//                在这里做具体的转换,对输出的每一条数据都做转换
                return s.length();
            }
        });

//        打印输出
        //sdss.print();

//        mapStream.print();
//        执行程序
        env.execute();

    }
}
~~~

##### **flatMap **

- flatMap 转换类似于 map ，但它可以对每个输入事件产生零个、 一个或多个输出事件 。事实上， flatMap 转换可以看做是 filter 和 map 的泛化，它能够实现后两者的操作。 flatMap 转换会针对每个到来事件应用一个函数 。 
- 对应 的Fl atMapFunction 定义了 flatMap() 方法，你可以在其中通过向 Collector 对象传递数据的方式返回零个、一个或多个事件作为结果 :。
- flatMap:将集合中的每个元素变成一个或多个元素,并返回扁平化之后的结果

![1621852739453](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/183901-464396.png)

**Fl atMapFunction 接口**

~~~ java
@FunctionalInterface
@Public
// T:输入元素类型
// O:输入元素类型
// Collector：用于返回多个结果
public interface FlatMapFunction<T, O> extends Function, Serializable {
    void flatMap(T var1, Collector<O> var2) throws Exception;
}
//对每一个集合进行扁平化处理后返回结果
~~~

**案例**

~~~ java
public class TransformBase01 {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");


//        对读进来的数据做基本的转换
//      2 flatMap()转换
//                按照逗号对字符串进行切分,输入是String类型，输出是String类型
        DataStream <String>mapFlatStream=sdss.flatMap(new FlatMapFunction<String, String>() {
//            collector是用来输出数据
            public void flatMap(String s, Collector<String> collector) throws Exception {
                String[] split = s.split(",");
                for (String value:split) {
                    collector.collect(value);
                }
            }
        });

//        打印输出
        //sdss.print();

//        mapStream.print();
//            mapFlatStream.print();
        mapFliter.print();
//        执行程序
        env.execute();

    }
}
~~~

##### **Filter **

- filter 转换利用一个作用在流中每条输入事件上的布尔条件来决定事件的去留:如果返回值为 true ，那么它会保留输入事件并将其转发到输出，否则它会把事件丢弃。通过调用 DataStream.filter() 方住可以指定 filter 转换产生一个数据类型不变的 DataStream。 

- 可以利用 FilterFunction 接口或 Lambda 函数来实现定义布尔条件的函数。FilterFunction 接口的类型为输入流的类型，它的 filter() 方法会接收一个输入事件，返回一个布尔值: 对数据做过滤操作。

**FilterFunction接口**

~~~ java
@FunctionalInterface
@Public
//T:输入元素的类型
public interface FilterFunction<T> extends Function, Serializable {
    boolean filter(T var1) throws Exception;
}
//满足过滤条件返回true，就保留数据
~~~

下面filter 操作仅保留了白色方块。 

![1614424678962](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/220449-508125.png)

~~~ java
public class TransformBase01 {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

//        对读进来的数据做基本的转换
//        3 filter()过滤操作
        DataStream <String>mapFliter=sdss.filter(new FilterFunction<String>() {
            public boolean filter(String s) throws Exception {
//                返回false说明过滤掉，返回true说明保留
                return s.startsWith("sensor_1");
            }
        });

//        打印输出
        //sdss.print();

//        mapStream.print();
//            mapFlatStream.print();
        mapFliter.print();
//        执行程序
        env.execute();

    }
}
~~~

##### sum()

sum:按照指定的字段对集合中的元素进行求和

~~~ java
public class Test02 {

    /**
     * DatasTREAM既支持流处理，也支持批处理，如何区分流处理和批处理？
     * @param args
     */
    public static void main(String[] args) throws Exception {

//        1 获取执行环境,获取的是流处理的env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

//        如果想做批处理，可以进行下面的设置,RuntimeExecutionMode是一个枚举类，里面有三个值
        env.setRuntimeMode(RuntimeExecutionMode.BATCH);//使用批处理
//        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);//根据数据源自动进行选择
//        env.setRuntimeMode(RuntimeExecutionMode.STREAMING);//流处理

//      读取数据
        DataStreamSource<String> data = env.readTextFile("D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt");
//        操作数据
        SingleOutputStreamOperator<String> flatmapData = data.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
                String[] s = value.split(" ");
                for (String str:s) {
                    out.collect(str);
                }
            }
        });

        SingleOutputStreamOperator<Tuple2<String, Integer>> mapData = flatmapData.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return  Tuple2.of(value,1);
            }
        });

//        数据聚合操作
//        KeyedStream<Tuple2<String, Integer>, Tuple> sum = mapData.keyBy(0);

//        第一个参数表示输入的类型，第二个参数表示提取key 的类型,根据返回的key对数据进行分组
        KeyedStream<Tuple2<String, Integer>, String> sum = mapData.keyBy(new KeySelector<Tuple2<String, Integer>, String>() {
            @Override
            public String getKey(Tuple2<String, Integer> value) throws Exception {
                return value.f0;
            }
        });
    //      sum()函数做聚合操作
        SingleOutputStreamOperator<Tuple2<String, Integer>> sum1 = sum.sum(1);

        sum.print();
//        启动程序并且等待结束
        env.execute();
//        在后续的开发中，把一切的数据源都看做是流数据

    }
}
~~~

##### reduce()

reduce:对集合中的元素进行聚合

![1621853250817](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/185045-98488.png)

**案例**

~~~ java
public class Test02 {

    /**
     * DatasTREAM既支持流处理，也支持批处理，如何区分流处理和批处理？
     * @param args
     */
    public static void main(String[] args) throws Exception {

//        1 获取执行环境,获取的是流处理的env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

//        如果想做批处理，可以进行下面的设置,RuntimeExecutionMode是一个枚举类，里面有三个值
        env.setRuntimeMode(RuntimeExecutionMode.BATCH);//使用批处理
//        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);//根据数据源自动进行选择
//        env.setRuntimeMode(RuntimeExecutionMode.STREAMING);//流处理

//      读取数据
        DataStreamSource<String> data = env.readTextFile("D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt");
//        操作数据
        SingleOutputStreamOperator<String> flatmapData = data.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
                String[] s = value.split(" ");
                for (String str:s) {
                    out.collect(str);
                }
            }
        });

        SingleOutputStreamOperator<Tuple2<String, Integer>> mapData = flatmapData.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return  Tuple2.of(value,1);
            }
        });

//        数据聚合操作
//        KeyedStream<Tuple2<String, Integer>, Tuple> sum = mapData.keyBy(0);

//        第一个参数表示输入的类型，第二个参数表示提取key 的类型,根据返回的key对数据进行分组
        KeyedStream<Tuple2<String, Integer>, String> sum = mapData.keyBy(new KeySelector<Tuple2<String, Integer>, String>() {
            @Override
            public String getKey(Tuple2<String, Integer> value) throws Exception {
                return value.f0;
            }
        });
//          sum()函数做聚合操作
//        SingleOutputStreamOperator<Tuple2<String, Integer>> sum1 = sum.sum(1);


//        使用reduce对最后的结果进行聚合操作
        SingleOutputStreamOperator<Tuple2<String, Integer>> res = sum.reduce(new ReduceFunction<Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> reduce(Tuple2<String, Integer> value1, Tuple2<String, Integer> value2) throws Exception {
                return Tuple2.of(value1.f0,value1.f1+value2.f1);
            }
        });
        res.print();
//        启动程序并且等待结束
        env.execute();
//        在后续的开发中，把一切的数据源都看做是流数据

    }
}
~~~

##### 综合案例

~~~ java
public class Test08 {

    public static void main(String[] args) throws Exception {

//        DataSet api使用
//        1 准备环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

//        2 加载数据
        DataSet<String> dataSource = env.readTextFile("D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt");
//        3 处理数据
        FlatMapOperator<String, String> flatmapData = dataSource.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
//               value表示每一行数据。现在对每一行数据进行切割
                String[] s = value.split(" ");
//                遍历切割好的单词，返回单词表
                for (String str : s) {
                    out.collect(str);
                }
            }
        });
//        对数据进行过滤操作
        FilterOperator<String> filterData = flatmapData.filter(new FilterFunction<String>() {
            @Override
            public boolean filter(String value) throws Exception {
                return !value.equals("TMD");//如果是TMD那么就返回false，表示过滤掉
            }
        });


//        转换结构
        MapOperator<String, Tuple2<String, Integer>> map = filterData.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return new Tuple2<>(value, 1);
            }
        });

//        统计单词
        UnsortedGrouping<Tuple2<String, Integer>> groupData = map.groupBy(new KeySelector<Tuple2<String, Integer>, String>() {
            @Override
            public String getKey(Tuple2<String, Integer> value) throws Exception {
                return value.f0;
            }
        });

//        对最终的结果进行聚合操作
//        AggregateOperator<Tuple2<String, Integer>> res = groupData.sum(1);

//        也可以使用reduce进行聚合操作
        ReduceOperator<Tuple2<String, Integer>> result = groupData.reduce(new ReduceFunction<Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> reduce(Tuple2<String, Integer> value1, Tuple2<String, Integer> value2) throws Exception {
                return Tuple2.of(value1.f0, value1.f1 + value2.f1);
            }
        });

//        4 输出结果
        result.print();

    }
}
~~~

#### 基于 KeyedStream 的转换 

- 很多应用需要将事件按照某个属性**分组**后再进行处理。作为 DataStream API中 一类特殊的 DataStream ， KeyedStream 抽象可以从逻辑上将事件按照键值分配到多条独立的子流 中 ，其继承于DataStream。

![1614435119343](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/221221-910361.png)

~~~ java
//可以看到，KeyedStream还是继承于DataStream
public class KeyedStream<T, KEY> extends DataStream<T> { }
~~~

- **在流中没有直接的聚合操作，必须先进行分组操作，然后在做聚合操作。**

- 作用于 KeyedStream 的状态化转换可以对当前处理事件的键值所对应上下文中的状态进行读写。这意味着所有键值相同的事件可以访问相同的状态，因此它们可以被一并处理 。 

- KeyedStream 也支持使用你之见看到过的 map 、f1atMap 和 filter 等转换进行处理 

##### KeyBy 

- keyBy 转换通过指定键值的方式将一个 DataStream 转化为 KeyedStream。**流中的事件会根据各自键值被分到不同的分区**，这样一来，有着相同键值的事件一定会在后续算子的同一个任务上处理。虽然键值不同的事件也可能会在同一个任务上处理，但任务函数所能访问的键值分区状态始终会被约束在当前事件键值的范围内。 
- keyBy () 方法接收一个用来指定分区键值(可以是多个)的参数，返回 一个KeyedStream。 
- 流处理中没有groupBy,而是keyBy

**KeySelector接口**

~~~ java
@Public
@FunctionalInterface
public interface KeySelector<IN, KEY> extends Function, Serializable {
//返回分区的id
	KEY getKey(IN value) throws Exception;
}
~~~

如下图将黑色时间划分到一个分区中，其他时间划分到另一个分区中。

![1614425447441](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/220624-639789.png)

DataStream → KeyedStream：逻辑地将一个流拆分成不相交的分区，**每个分区包含具有相同 key 的元素，在内部以 hash 的形式实现的**。一个分区内也可能有不同的key,因为是按照hash进行映射的。

**代码说明**

~~~ java
public class TransformBase_keyby {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(4);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

//        转换为sensorreading类型
        DataStream<SensorReading> map = sdss.map(new MapFunction<String, SensorReading>() {
            public SensorReading map(String s) throws Exception {

                String[] s1 = s.split(",");

                return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));
            }
        });

//        按照第一个位置处的元素进行分区，在这里是按照id对数据进行分区操作
        KeyedStream<SensorReading, Tuple> resgb = map.keyBy("id");
        resgb.writeAsText("D:\\soft\\idea\\work\\work08\\data2");


//        执行程序
        env.execute();
    }
}
//会按照指定的id进行分组操作
~~~

##### 滚动聚合算子（Rolling Aggregation） 

滚动聚合转换作用于 KeyedStream 上，它将生成一个包含聚合结果(例如求和、最小值、最大值等)的 DataStream。滚动聚合算子会对每一个遇到过的键值保存一个聚合结果。每当有新事件到来，该算子都会更新相应的聚合结果， 并将其 以事件的形式发送出 去。 滚动聚合虽然不需要用户自定义函数，但需要接收一个用于指定聚合目标字段的参数。 

这下面这些算子可以针对 KeyedStream 的每一个支流做滚动聚合。

- sum()：滚动计算输入流中指定宇段的和。 
- max()：滚动计算输入流中指定字段的最大值。 
- min()：滚动计算输入流中指定字段的最小值。 
- minBy()：滚动计算输入流中迄今为止最小值，返回该值所在事件 。 
- maxBy()：滚动计算输入流中迄今为止最大值，返回该值所在事件。 

> 无法将多个滚动聚合方法组合使用， 每次只能计算一个。 

**案例**

~~~ java
public class Transform_Rolling_Aggregation {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

//        转换为sensorreading类型
//        DataStream<SensorReading> map = sdss.map(new MapFunction<String, SensorReading>() {
//            public SensorReading map(String s) throws Exception {
//
//                String[] s1 = s.split(",");
//
//                return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));
//            }
//        });

//        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));
        });

//       按照bean的字段进行分组
//        public class KeyedStream<T, KEY> extends DataStream<T>：<T, KEY>T表示输入类型，KEY表示分组字段的类型
//        但是返回的key类型是tuple类型，这里和源码有关
//        public KeyedStream<T, Tuple> keyBy(String... fields) {
//            return keyBy(new Keys.ExpressionKeys<>(fields, getType()));
//        }
        KeyedStream<SensorReading, Tuple> keyedStream = map.keyBy("id");
//        传入的匿名函数，最后返回String类型
//        KeyedStream<SensorReading, String> sensorReadingObjectKeyedStream = map.keyBy(data ->
//            data.getId()
//        );
//        分组聚合，取当前最大的温度值，按照字段，取某一个字段的最大值
//        SingleOutputStreamOperator<SensorReading> resultStream = keyedStream.max("temperature");
//        滚动聚合，获取某一个最大值
        SingleOutputStreamOperator<SensorReading> resultStream = keyedStream.maxBy("temperature");

//        打印输出
       // keyedStream.print();

        resultStream.print();
//        执行程序
        env.execute();
    }
}
//输出结果
SensorReading{id='sensor_1', tempStamp=1547718199, temperature=35.8}
SensorReading{id='sensor_6', tempStamp=1547718201, temperature=15.4}
SensorReading{id='sensor_7', tempStamp=1547718202, temperature=6.7}
SensorReading{id='sensor_10', tempStamp=1547718205, temperature=38.1}
SensorReading{id='sensor_1', tempStamp=1547718198, temperature=36.8}
SensorReading{id='sensor_1', tempStamp=1547718198, temperature=36.8}
SensorReading{id='sensor_1', tempStamp=1547718209, temperature=39.8}
//可以看到，时间戳没有进行更新
~~~

> 滚动聚合算子会为每个处理过的键值维持一个状态。由干这些状态不会被自动清理，所以该算子只能用于键值域有限的流。 

##### Reduce 

reduce 转换是滚动聚合转换的泛化。它将一个 ReduceFunction 应用在一个 KeyedStream 上，每个到 来事件都会和 reduce 结果进行一次组合，从而产生 一个新的 DataStream  ，reduce 转换不会改变数据类型， 因此输出流的类型会 永远和输入流保持一致。 

KeyedStream → DataStream：一个分组数据流的聚合操作，合并当前的元素和上次聚合的结果，产生一个新的值，返回的流中包含每一次聚合的结果，而不是只返回最后一次聚合的最终结果。 

**ReduceFunction接口**

~~~ JAVA
@Public
@FunctionalInterface
public interface ReduceFunction<T> extends Function, Serializable {
//只有一个泛型类型，也就是说数据类型不能够变，和滚动聚合一样
	/**
	 * The core method of ReduceFunction, combining two values into one value of the same type.
	 * The reduce function is consecutively applied to all values of a group until only a single value remains.
	 *
	 * @param value1 The first value to combine.
	 * @param value2 The second value to combine.
	 * @return The combined value of both input values.
	 *
	 * @throws Exception This method may throw exceptions. Throwing an exception will cause the operation
	 *                   to fail and may trigger recovery.
	 */
  //value1是之前聚合出来的状态，value2是当前的最新的值
	T reduce(T value1, T value2) throws Exception;
}
~~~

**案例**

~~~ java
public class Transform_Rolling_Reducer {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

//        转换为sensorreading类型
//        DataStream<SensorReading> map = sdss.map(new MapFunction<String, SensorReading>() {
//            public SensorReading map(String s) throws Exception {
//
//                String[] s1 = s.split(",");
//
//                return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));
//            }
//        });

//        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));
        });
        KeyedStream<SensorReading, Tuple> keyedStream = map.keyBy("id");

//        reduce聚合操作，取最大的温度值和最新的时间戳
       DataStream resultStream= keyedStream.reduce(new ReduceFunction<SensorReading>() {
            @Override
            public SensorReading reduce(SensorReading value1, SensorReading value2) throws Exception {

                return new SensorReading(value1.getId(),value2.getTempStamp(),Math.max(value1.getTemperature(),value2.getTemperature()));

            }
        });
       resultStream.print();



//        执行程序
        env.execute();
    }
}
//输出结果
SensorReading{id='sensor_1', tempStamp=1547718199, temperature=35.8}
SensorReading{id='sensor_6', tempStamp=1547718201, temperature=15.4}
SensorReading{id='sensor_7', tempStamp=1547718202, temperature=6.7}
SensorReading{id='sensor_10', tempStamp=1547718205, temperature=38.1}
SensorReading{id='sensor_1', tempStamp=1547718198, temperature=36.8}
SensorReading{id='sensor_1', tempStamp=1547718183, temperature=36.8}
SensorReading{id='sensor_1', tempStamp=1547718209, temperature=39.8}
//和上面的区别是时间戳进行更新操作
~~~

map,flatMap,filter,reducer都是操作一条数据流

下面的算子可以操作多条数据流=>多流转换算子

##### Split 和 Select 

split 转换是 union 转换的逆操作。它将输入流分割成两条或多条类型和输入流相同的输出流。每一个到来的事件都可以被发往零个、 一个或多个输出流。因此， split 也可以用来过滤或复制事件。 

DataStream.split() 方法接收一个 OutputSelector ，它用来定义如何将数据流的元素分配到不同的命名输出 ( named outpu t) 中。 OutputSelector 中定 义的 select() 方也会在每个输入事件到来时被调用，并随即返回 一个 java.lang.lterable[String] 对象 。 针对某记录所返回的一 系列 String 值指定了该记录需要被发往哪些输出流。 

DataStream.split() 方法会返回 一个 SplitStream 对象，它提供的 select()方法可以让我们通过指定输出名称的方式从 SplitStream 中选择一条或多条流 。 

按照一定的特征，把数据做一个划分，split和select要配合使用。

**Split**

![1614471868219](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614471868219.png)

DataStream → SplitStream：根据某些特征把一个 DataStream 拆分成两个或者多个 DataStream。 

**Select**

![1614471898093](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614471898093.png)

SplitStream→ DataStream：从一个 SplitStream 中获取一个或者多个DataStream。 

**OutputSelector接口**

~~~ java
@PublicEvolving
public interface OutputSelector<OUT> extends Serializable {
	/**
	 * Method for selecting output names for the emitted objects when using the
	 * {@link SingleOutputStreamOperator#split} method. The values will be
	 * emitted only to output names which are contained in the returned
	 * iterable.
	 *
	 * @param value
	 *            Output object for which the output selection should be made.
	 */
	Iterable<String> select(OUT value);
}
~~~

**需求**

需求： 传感器数据按照温度高低（以 30 度为界），拆分成两个流。 

~~~ java
public class TransformBase_multTrans {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

//        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));
        });

//        分流操作，按照温度30为界限，分成两条流
//        参数表示选择器
//        上层api已经被弃用，所以要使用底层的api
        SplitStream<SensorReading> split = map.split(new OutputSelector<SensorReading>() {
//            方法返回一个可以迭代类型的集合
            @Override
            public Iterable<String> select(SensorReading value) {
//                给每一条记录打标签
                return (value.getTemperature()>30)? Collections.singletonList("high"):Collections.singletonList("low");
            }
        });

//        按照标签筛选
//        DataStream<SensorReading> high = split.select("low");
//        筛选多个标签
        DataStream<SensorReading> high = split.select("low","high");

        high.print();

//        执行程序
        env.execute();
    }
}
~~~

##### 拆分和选择

- Split就是将一个流分成多个流

- Select就是获取分流后对应的数据

> 注意：split函数已过期并移除

- Side Outputs：可以使用process方法对流中数据进行处理，并针对不同的处理结果将数据收集到不同的OutputTag中

 对流中的数据按照奇数和偶数进行分流，并获取分流后的数据

~~~ java
public class Test09 {

    public static void main(String[] args) throws Exception {

//        DataSet api使用
//        1 准备环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //2.Source
        DataStreamSource<Integer> data = env.fromElements(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        OutputTag<Integer> oddTag = new OutputTag<Integer>("奇数", TypeInformation.of(Integer.class));
        OutputTag<Integer> even = new OutputTag<Integer>("偶数",TypeInformation.of(Integer.class));

        SingleOutputStreamOperator<Integer> res = data.process(new ProcessFunction<Integer, Integer>() {
            @Override
            public void processElement(Integer value, Context ctx, Collector<Integer> out) throws Exception {
//                out收集完毕的数据还是放在一起，ctx可以将数据放在不同的outputtag
                if(value %2 == 0){
                    ctx.output(even,value);
                }else {
                    ctx.output(oddTag,value);
                }
            }
        });
//        获取数据
        DataStream<Integer> oddData = res.getSideOutput(oddTag);
        DataStream<Integer> evenData = res.getSideOutput(even);
        oddData.print("奇数");
        evenData.print("偶数");
        env.execute();
    }
}
~~~



##### Connect 和 CoMap 

**connect**

DataStream.connect() 方法接收一个 DataStream 并返回 一个 ConnectedStream 对象，该对象表示两个联结起来 (connected) 的流:，ConnectedStreams 对象提供了 map() 和 flatMap ()方法，它们分别接收一个CoMapFunction 和一个 CoFlatMapFunction 作为参数，两个函数都是以两条输入流的类型外加输出流的类型作为其类型参数，它们为两条输入流定义了各自的处理方怯。 mapl() 和 flatMapl() 用来处理第一条输入流的事件， map2() 和 flatMap2() 用来处理第二条输入流的事件: 

![1614473695309](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/28/214907-132600.png)

DataStream,DataStream → ConnectedStreams：连接两个保持他们类型的数据流，两个数据流被 Connect 之后，只是被放在了一个同一个流中，内部依然保持各自的数据和形式不发生任何变化，两个流相互独立。 

**CoMap,CoFlatMap **

![1614473760250](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614473760250.png)

ConnectedStreams → DataStream：作用于 ConnectedStreams 上，功能与 map和 flatMap 一样，对 ConnectedStreams 中的每一个 Stream 分别进行 map 和 flatMap处理。 

**CoMapFunction接口**

~~~ java
@Public
public interface CoMapFunction<IN1, IN2, OUT> extends Function, Serializable {

	/**
	 * This method is called for each element in the first of the connected streams.
	 *
	 * @param value The stream element
	 * @return The resulting element
	 * @throws Exception The function may throw exceptions which cause the streaming program
	 *                   to fail and go into recovery.
	 */
  //第一条流的逻辑
	OUT map1(IN1 value) throws Exception;

	/**
	 * This method is called for each element in the second of the connected streams.
	 *
	 * @param value The stream element
	 * @return The resulting element
	 * @throws Exception The function may throw exceptions which cause the streaming program
	 *                   to fail and go into recovery.
	 */
  //第二条流的逻辑
	OUT map2(IN2 value) throws Exception;
}
//上面两条流的逻辑相互独立
~~~

**合流**

~~~ java
public class TransformBase_multTrans_connect {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

//        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));
        });

//        分流操作，按照温度30为界限，分成两条流
//        参数表示选择器
//        上层api已经被弃用，所以要使用底层的api
        SplitStream<SensorReading> split = map.split(new OutputSelector<SensorReading>() {
//            方法返回一个可以迭代类型的集合
            @Override
            public Iterable<String> select(SensorReading value) {
//                给每一条记录打标签
                return (value.getTemperature()>30)? Collections.singletonList("high"):Collections.singletonList("low");
            }
        });

//        按照标签筛选
//        DataStream<SensorReading> high = split.select("low");
//        筛选多个标签
        DataStream<SensorReading> high =split.select("high");
        DataStream<SensorReading> low =split.select("low");

//        使用connect，将高温流转换为元祖类型，与低温流合并之后，输出状态信息
        SingleOutputStreamOperator<Tuple2<String, Double>> dataStream = high.map(new MapFunction<SensorReading, Tuple2<String, Double>>() {
            @Override
            public Tuple2<String, Double> map(SensorReading value) throws Exception {
                return new Tuple2<>(value.getId(), value.getTemperature());
            }
        });

//        接下来做链接操作,基于dataStream流和高温流做链接操作
//        public <R> ConnectedStreams<T, R> connect(DataStream<R> dataStream)
//        T是调用的流的类型，R是被调用流的类型
        ConnectedStreams<Tuple2<String, Double>, SensorReading> connectStreams = dataStream.connect(low);

//        三个参数代表的含义
//        Tuple2<String, Double>:高温流的输入类型
//        SensorReading:低温流的输入类型
//        Object：最终输出的流的类型
        SingleOutputStreamOperator<Object> result = connectStreams.map(new CoMapFunction<Tuple2<String, Double>, SensorReading, Object>() {

//            因为返回类型是object,所以两个方法返回的类型要兼容

            @Override
            public Object map1(Tuple2<String, Double> value) throws Exception {

//                对高温流，返回一个三元组
//                元祖里面的元素叫做f0,f1,f2
                return new Tuple3<>(value.f0, value.f1, "high tem warning");
            }

            @Override
            public Object map2(SensorReading value) throws Exception {
//                对于低温流，返回正常信息
                return new Tuple2<>(value.getId(), "normal");
            }
        });

//        high.print();
            result.print();

//        执行程序
        env.execute();
    }
}
//输出结果
(sensor_1,35.8,high tem warning)
(sensor_6,normal)
(sensor_1,36.8,high tem warning)
(sensor_7,normal)
(sensor_1,36.2,high tem warning)
(sensor_10,normal)
(sensor_1,39.8,high tem warning)

~~~

上面的缺点是不可以链接多条流操作。

> CoMapFunction 函数无法选择从哪条流读取数据 和 CoFlatMapFunction 内方法的调用顺序无法控制 。一旦对应流中有事件到来，系统就需要调用相应的方泣。 

##### Union 

- DataStream.union() 方法可以合并两条或多条类型相同的 DataStream ，生成一个新的类型相同的 DataStream。这样后续的转换操作就可以对所有输入流中的元素统一处理。 

- union可以进行多条流的合并，**但是多条流的数据类型必须一样**，也就是union只可以合并同类型的流，connect只能合并两条流，但是流的类型可以不一样。也就是说connent可以合并一样类型的流和不一样类型的流。

- 可以发现，到目前为止，所有的转换全部是基于DataStream流进行的，因为基础的数据结构是DataStream类型。

![1614474499237](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/28/210810-646817.png)

union 执行过程中，来自两条流的事件会以 FIFO (先进先出)的方式合井，其顺序无法得到任何保证。此外， union **算子不会对数据进行去重**，每个输入消息都会被发往下游算子。 

DataStream → DataStream：对两个或者两个以上的 DataStream 进行 union 操作，产生一个包含所有 DataStream 元素的新 DataStream。 

~~~ java
public class TransformBase_multTrans_connect {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

//        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));
        });

//        分流操作，按照温度30为界限，分成两条流
//        参数表示选择器
//        上层api已经被弃用，所以要使用底层的api
        SplitStream<SensorReading> split = map.split(new OutputSelector<SensorReading>() {
//            方法返回一个可以迭代类型的集合
            @Override
            public Iterable<String> select(SensorReading value) {
//                给每一条记录打标签
                return (value.getTemperature()>30)? Collections.singletonList("high"):Collections.singletonList("low");
            }
        });

//        按照标签筛选
//        DataStream<SensorReading> high = split.select("low");
//        筛选多个标签
        DataStream<SensorReading> high =split.select("high");
        DataStream<SensorReading> low =split.select("low");

//        使用connect，将高温流转换为元祖类型，与低温流合并之后，输出状态信息
        SingleOutputStreamOperator<Tuple2<String, Double>> dataStream = high.map(new MapFunction<SensorReading, Tuple2<String, Double>>() {
            @Override
            public Tuple2<String, Double> map(SensorReading value) throws Exception {
                return new Tuple2<>(value.getId(), value.getTemperature());
            }
        });

//        接下来做链接操作,基于dataStream流和高温流做链接操作
//        public <R> ConnectedStreams<T, R> connect(DataStream<R> dataStream)
//        T是调用的流的类型，R是被调用流的类型
        ConnectedStreams<Tuple2<String, Double>, SensorReading> connectStreams = dataStream.connect(low);

//        三个参数代表的含义
//        Tuple2<String, Double>:高温流的输入类型
//        SensorReading:低温流的输入类型
//        Object：最终输出的流的类型
        SingleOutputStreamOperator<Object> result = connectStreams.map(new CoMapFunction<Tuple2<String, Double>, SensorReading, Object>() {

//            因为返回类型是object,所以两个方法返回的类型要兼容

            @Override
            public Object map1(Tuple2<String, Double> value) throws Exception {

//                对高温流，返回一个三元组
//                元祖里面的元素叫做f0,f1,f2
                return new Tuple3<>(value.f0, value.f1, "high tem warning");
            }

            @Override
            public Object map2(SensorReading value) throws Exception {
//                对于低温流，返回正常信息
                return new Tuple2<>(value.getId(), "normal");
            }
        });

//        union可以连接多条流，但是流的类型必须一样，从底层源码可以看到public final DataStream<T> union(DataStream<T>... streams)
//        从源码可以看到，泛型都是T类型,union还可以联合多条流
        DataStream<SensorReading> unionStream = high.union(low);
        unionStream.print();

//        high.print();
//            result.print();

//        执行程序
        env.execute();
    }
}
~~~

union和connect的区别

1. Union 之前两个流的类型必须是一样， Connect 可以不一样，在之后的 coMap中再去调整成为一样的 
2. Connect 只能操作两个流， Union 可以操作多个 
3. 两个DataStream经过connect之后被转化为ConnectedStreams，ConnectedStreams会对两个流的数据应用不同的处理方法，且双流之间可以共享状态。

![1621845625755](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/144750-807202.png)

#### 重分区

类似于Spark中的repartition,但是功能更强大,可以直接解决数据倾斜

Flink也有数据倾斜的时候，比如当前有数据量大概10亿条数据需要处理，在处理过程中可能会发生如图所示的状况，出现了数据倾斜，其他3台机器执行完毕也要等待机器1执行完毕后才算整体将任务完成；

![1621850834398](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/144750-273094.png)

所以在实际的工作中，出现这种情况比较好的解决方案就是rebalance(内部使用round robin方法将数据均匀打散)

![1621850867161](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/144749-230543.png)

##### rebalance重平衡分区

~~~ java
public class Test10 {

    public static void main(String[] args) throws Exception {

//        DataSet api使用
//        1 准备环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //2.Source
        DataStreamSource<Long> source = env.fromSequence(0, 100);
        env.setParallelism(2);

//        下面操作相当于将数据进行重新随机的分配，有可能发生数据的倾斜
        SingleOutputStreamOperator<Long> filterData = source.filter(new FilterFunction<Long>() {
            @Override
            public boolean filter(Long value) throws Exception {
                return value > 10;
            }
        });

        SingleOutputStreamOperator<Tuple2<Integer, Integer>> mapData = filterData
                .rebalance()
                .map(new RichMapFunction<Long, Tuple2<Integer, Integer>>() {

            @Override
            public Tuple2<Integer, Integer> map(Long value) throws Exception {
                int indexOfThisSubtask = getRuntimeContext().getIndexOfThisSubtask();//获取子任务id，也就是分区编号
                return Tuple2.of(indexOfThisSubtask, 1);
            }
        });

//        查看每一个分区中有多少个元素
        SingleOutputStreamOperator<Tuple2<Integer, Integer>> countData = mapData.keyBy(item -> item.f0).sum(1);

        countData.print();

        env.execute();
    }
}
~~~

#####  其他分区

![1621925336453](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/144857-364414.png)

最后一个可以理解为自定义分区。

**案例演示**

~~~ java
public class Test11 {

    public static void main(String[] args) throws Exception {

//        DataSet api使用
//        1 准备环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

//        2 加载数据
        DataSet<String> linesDS = env.readTextFile("D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt");
//        3 处理数据
        FlatMapOperator<String, Tuple2<String, Integer>> tupleDS = linesDS.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                String[] words = value.split(" ");
                for (String word : words) {
                    out.collect(Tuple2.of(word, 1));
                }
            }
        });

        //3.Transformation
//        DataStream<Tuple2<String, Integer>> result1 = tupleDS.global();数据发送到第一个task
//        DataStream<Tuple2<String, Integer>> result2 = tupleDS.broadcast();把tupleDS中的数据广播到下游所有的task
//        DataStream<Tuple2<String, Integer>> result3 = tupleDS.forward();上下游并行度一样的时候，一个对一个的发送
//        DataStream<Tuple2<String, Integer>> result4 = tupleDS.shuffle();打乱随机轮流的发送
//        PartitionOperator<Tuple2<String, Integer>> result5 = tupleDS.rebalance();轮流分配，重平衡
//        DataStream<Tuple2<String, Integer>> result6 = tupleDS.rescale();本地轮流分配
//        下面是自定义分区器
        PartitionOperator<Tuple2<String, Integer>> result7 = tupleDS.partitionCustom(new Partitioner<String>() {
            @Override
            public int partition(String key, int numPartitions) {
                return key.equals("hello") ? 0 : 1;
            }
        }, t -> t.f0);

        //4.sink
        //result1.print();
        //result2.print();
        //result3.print();
        //result4.print();
        //result5.print();
        //result6.print();
        result7.print();

        //5.execute
        env.execute();

    }
}
~~~



### 并行度

每个算子都会产生一个或多个并行任务。每个任务负责处理算子的部分输入流。算子井行化任务的数目称为该算子的井行度。它决定了算子处理的井行化程度以及能够处理的数据规模。 

算子的并行度可以在执行环境级别或单个算子级别进行控制。默认情况下，应用内所有算子的并行度都会被设置为应用执行环境的井行度。而环境的并行度(即所有算子的默认井行度) 则会根据应用启动时所处的上下文自动初始化。如果应用是在一个本地执行环境中运行， 并行度会设置为 CPU 的线程数目。如果应用是提交到 Flink 集群运行，那么除非提交客户端明确指定 ,否则环境的并行度将设置为集群的默认并行度.

一般情况下，最好将算子并行度设置为随环境默认并行度变化的值。这样就可以通过提交客户端来轻易调整井行度，从而实现应用的扩缩容 。 你可以按照下面的示例来访问环境的默认并行度 : 

~~~ java
//设置环境的并行度
env.setParallelism(1);
~~~

你可以通过显式指定的方式来覆盖算子的默认并行度。下面的示例中，数据源算子会以环境默认并行度执行， map 转换的任务数是数据糠的两倍，数据汇操作固定以两个并行任务执行: 

~~~ java
val env = StreamExecutionEnvironment.getExecutionEnvironment
// 获取默认并行度
val defaultP = env.getParallelism
//数据源以默认并行度运行
val result: = env.addSource(new CustomSource)
//设置 map 的并行度为默认并行度的两倍
.map(new MyMapper).setParallelism(defaultP * 2)
// print 数据汇的并行度固定为 2
.print().setParallelism(2)
~~~







### 支持的数据类型

- Flink 流应用程序处理的是以**数据对象**表示的事件流。 所以在 Flink 内部， 我们需要能够处理这些对象。它们需要被序列化和反序列化，以便通过网络传送它们；或者从状态后端、检查点和保存点读取它们。为了有效地做到这一点， Flink 需要明确知道应用程序所处理的数据类型。 Flink 使用类型信息的概念来表示数据类型，并为每个数据类型生成特定的序列化器、反序列化器和比较器。 

- Flink 还具有一个类型提取系统，该系统分析函数的输入和返回类型，以自动获取类型信息，从而获得序列化器和反序列化器。但是，在某些情况下，例如 lambda函数或泛型类型，需要显式地提供类型信息，才能使应用程序正常工作或提高其性能 

- Flink 支持 Java 和 Scala 中所有常见数据类型。使用最广泛的类型有以下几种。 

#### 基础数据类型

Flink 支持所有的 Java 和 Scala 基础数据类型， Int, Double, Long, String, … 包括包装类

#### Java 和 Scala 元组（Tuples） 

java中的元组

![1614486838998](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/28/123400-213651.png)

#### Scala 样例类（case classes） 

~~~ java
 case class Person(name: String, age: Int)
        val persons: DataStream[Person] = env.fromElements(
            Person("Adam", 17),
            Person("Sarah", 23) )
        persons.filter(p => p.age > 18)
~~~

#### Java 简单对象（POJOs） 

~~~ java
public class Person {
  //注意：属性需要声明为public类型
            public String name;
            public int age;
  //还必须有一个空参数的构造方法
            public Person() {}
            public Person(String name, int age) {
                this.name = name;
                this.age = age;
            }
        }
        DataStream<Person> persons = env.fromElements(
                new Person("Alex", 42),
                new Person("Wendy", 23));
~~~

#### 其它（Arrays, Lists, Maps, Enums, 等等） 

Flink 对 Java 和 Scala 中的一些特殊目的的类型也都是支持的，比如 Java 的ArrayList， HashMap， Enum 等等。

 #### 为数据类型创建类型信息

Flink 类型系统的核心类是 Typelnformation ，它为系统生成序列化器和比
较器提供了必要的信息。 

~~~ java
@Public
public abstract class TypeInformation<T> implements Serializable {}
~~~

#### 定义键值和引用字段

1. 根据字段的位置引用
2. 使用字段表达式
3. 使用键值选择器

**使用键值选择器**

~~~ java
//        使用键值选择器，输入类型是sensorreading类型，输出的键值是元祖类型
        KeyedStream<SensorReading, List<String>> id = map.keyBy(new KeySelector<SensorReading, List<String>>() {
            @Override
            public List<String> getKey(SensorReading value) throws Exception {
                return Collections.singletonList(value.getId());
            }
        });
~~~

### 实现 UDF 函数——更细粒度的控制流 

#### 函数类（Function Classes） 

Flink 暴露了所有 udf 函数的接口(实现方式为接口或者抽象类)。例如MapFunction, FilterFunction, ProcessFunction 等等。 

下面例子实现了 FilterFunction 接口： 

~~~ java
DataStream<String> flinkTweets = tweets.filter(new FlinkFilter());
public static class FlinkFilter implements FilterFunction<String> {
    @Override
    public boolean filter(String value) throws Exception {
        return value.contains("flink");
    }
}
~~~

还可以将函数实现成匿名类 

~~~ java
DataStream<String> flinkTweets = tweets.filter(new FilterFunction<String>() {
@Override
public boolean filter(String value) throws Exception {
	return value.contains("flink");
	}
});
~~~

我们 filter 的字符串"flink"还可以当作参数传进去。 

~~~ java

    DataStream<String> tweets = env.readTextFile("INPUT_FILE ");
    DataStream<String> flinkTweets = tweets.filter(new KeyWordFilter("flink"));

public static class KeyWordFilter implements FilterFunction<String> {
    private String keyWord;

    KeyWordFilter(String keyWord) {
        this.keyWord = keyWord;
    }

    @Override
    public boolean filter(String value) throws Exception {
        return value.contains(this.keyWord);
    }
}
~~~

#### 匿名函数（Lambda Functions） 

~~~ java
DataStream<String> tweets = env.readTextFile("INPUT_FILE");
DataStream<String> flinkTweets = tweets.filter( tweet -> tweet.contains("flink") );
~~~

#### 富函数（Rich Functions） 

“富函数”是 DataStream API 提供的一个函数类的接口， 所有 Flink 函数类都有其 Rich 版本。 它与常规函数的不同在于，可以获取运行环境的上下文，并拥有一些生命周期方法，所以可以实现更复杂的功能，富函数的命名规则是以 Rich 开头，后面跟着普通转换函数的名字  

- RichMapFunction
-  RichFlatMapFunction
- RichFilterFunction 

Rich Function 有一个生命周期的概念。 典型的生命周期方法有： 

- open()方法是 rich function 的初始化方法，当一个算子例如 map 或者 filter被调用之前 open()会被调用。，open() :是富函数中的初始化方法。它在每个任务首次调用转换方能(如fi lter 或 m ap) 前调用 一 次。 open() 通常用于那些只需进行一 次的设置工作。 
- close () 作为函数的终止方洁，会在每个任务最后一次调用转换方法后调用一次。它通常用于清理和释放资源。  
- getRuntimeContext()方法提供了函数的 RuntimeContext 的一些信息，例如函数执行的并行度，任务的名字，以及 state 状态 

~~~ java
public class Transform_Rolling_RechFunction {

    public static void main(String[] args) throws Exception {

        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是4,open和close会输出四次，因为有4个分区，每一个分区会对应的有一个MyRichFunction类的实例对象
        env.setParallelism(4);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");
//        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line -> {
            String[] s1 = line.split(",");

            return new SensorReading(s1[0], new Long(s1[1]), new Double(s1[2]));
        });

//        DataStream<Tuple2<String, Integer>> resultStream = map.map(new MyMapper());
        DataStream<Tuple2<String, Integer>> resultStream = map.map(new MyRichFunction());

        resultStream.print();

//        执行程序
        env.execute();


    }

    public static class MyMapper implements MapFunction<SensorReading, Tuple2<String, Integer>> {

//        每一个数据到来后，都会调用map方法

        @Override
        public Tuple2<String, Integer> map(SensorReading value) throws Exception {
            return new Tuple2<>(value.getId(), value.getId().length());
        }
    }

    //    实现自定义的富函数类
    public static class MyRichFunction extends RichMapFunction<SensorReading, Tuple2<String, Integer>> {

        @Override
        public void open(Configuration parameters) throws Exception {
//            一般做一些初始化工作,定义状态，获取建立数据库的链接
            System.out.println("open method");

        }

        @Override
        public Tuple2<String, Integer> map(SensorReading value) throws Exception {
            return new Tuple2<>(value.getId(), value.getId().length());
        }

        @Override
        public void close() throws Exception {
//            一般是关闭链接，清空状态的收尾工作
            System.out.println("close");
        }
    }
}
~~~

37

### Sink 

Flink 没有类似于 spark 中 foreach 方法，让用户进行迭代的操作。虽有对外的输出操作都要利用 Sink 完成。最后通过类似如下方式完成整个任务最终输出操作。 

~~~ java
stream.addSink(new MySink(xxxx))
~~~

官方提供了一部分的框架的 sink。除此以外，需要用户自定义实现 sink。 

**官方提供的连接器**

![1614559803465](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/085237-536905.png)

**第三方提供的连接器**

![1614559978906](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/151149-524971.png)

#### 基于控制台的输出

- ds.print 直接输出到控制台

- ds.printToErr() 直接输出到控制台,用红色

- ds.writeAsText("本地/HDFS的path",WriteMode.OVERWRITE).setParallelism(1)

 **注意:**

- 在输出到path的时候,可以在前面设置并行度,如果
  - 并行度>1,则path为目录
  - 并行度=1,则path为文件名

**案例**

~~~ java
public class Test12 {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(2);

        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);
        DataStreamSource<String> dataSource = env.readTextFile("D:\\soft\\idea\\work\\work08\\hmlen\\src\\main\\resources\\words.txt");
        dataSource.print("输出标示");
        dataSource.print();
        dataSource.printToErr();//控制台上红颜色输出标示
        env.execute();
    }
}
~~~

#### Kafka 

**添加依赖包**

~~~ java
<dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka-0.11_2.12</artifactId>
            <version>1.10.1</version>
</dependency>
~~~

**案例**

~~~ java
public class Transform_sink {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");


//        使用lambda表达式
        DataStream<String> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2])).toString();
        });

//        new一个kafka的生产者
        map.addSink(new FlinkKafkaProducer011<String>("localhost:9092","sinktest",new SimpleStringSchema()));

//        执行程序
        env.execute();
    }
}
~~~

#### Rides

**依赖**

~~~ java
<dependency>
<groupId>org.apache.bahir</groupId>
<artifactId>flink-connector-redis_2.11</artifactId>
<version>1.0</version>
</dependency>
~~~

**案例**

使用建造者模式

~~~ java
public class SinkTestRides {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");


//        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });


//        定义jedis链接配置

        FlinkJedisPoolConfig config = new FlinkJedisPoolConfig.Builder().setHost("localhost").setPort(6379).build();
//        创建一个连接器
        map.addSink(new RedisSink<>(config,new MyRidesMapper()));
//        执行程序
        env.execute();

    }

    public static class MyRidesMapper implements RedisMapper<SensorReading> {

//定义数据保存到rides的命令，以哈希表形式存储,hset sensor_temp id temputer
//        SENSOR_TEMP:表名
        @Override
        public RedisCommandDescription getCommandDescription() {
            return new RedisCommandDescription(RedisCommand.HSET,"SENSOR_TEMP");
        }

        @Override
        public String getKeyFromData(SensorReading data) {
//写入数据库的键
            return data.getId();
        }

        @Override
        public String getValueFromData(SensorReading data) {
            
//            写入数据库的值
            double temperature = data.getTemperature();
            return temperature+"";
        }
    }
}
~~~

#### Elasticsearch 

**依赖**

~~~ java
<dependency>
<groupId>org.apache.flink</groupId>
<artifactId>flink-connector-elasticsearch6_2.12</artifactId>
<version>1.10.1</version>
</dependency>
~~~

**案例**

~~~ java
public class SinkTestES {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");


//        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

//        定义es的链接配置
        ArrayList<HttpHost> host=new ArrayList();
        host.add(new HttpHost("localhost",9200));

        map.addSink(new ElasticsearchSink.Builder<SensorReading>(host,new MyEsSinkFun()).build());


//        执行程序
        env.execute();

    }

    public static class MyEsSinkFun implements ElasticsearchSinkFunction<SensorReading>{

        @Override
        public void process(SensorReading sensorReading, RuntimeContext runtimeContext, RequestIndexer requestIndexer) {
//            实现写入数据的source
            HashMap datasource = new HashMap<String,String>();
            datasource.put("id",sensorReading.getId());
            datasource.put("temp",sensorReading.getTemperature());
            datasource.put("ts",sensorReading.getTempStamp());

//            创建请求作为向es发起的写入命令
            IndexRequest source = Requests.indexRequest().index("sensor").type("readingdata").source(datasource);

//            发送请求，使用indeser发发送
            requestIndexer.add(source);

        }
    }
}
~~~

#### JDBC 自定义 sink 

可以定义多种sink进行输出

![1621927643858](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/152727-856427.png)

**引入依赖**

~~~ java
<dependency>
<groupId>mysql</groupId>
<artifactId>mysql-connector-java</artifactId>
<version>5.1.44</version>
</dependency>
~~~

**自定义sink输出**

~~~ java
//    实现MyJDBCFun
    public static class MyJDBCFun extends RichSinkFunction<SensorReading>{
        Connection connection=null;

//        定义预编译器
    PreparedStatement insertStatement=null;
    PreparedStatement updatestat=null;

    @Override
    public void open(Configuration parameters) throws Exception {
//        声明周期方法，创建一个链接
//        获取一个链接
        connection= DriverManager.getConnection("jdbc:mysql://localhost:3306/test","root","root");
//        初始化
        insertStatement=connection.prepareStatement("insert into temp_sensor(id,temp) values(?,?)");
        updatestat=connection.prepareCall("update temp_sensor set temp=? where id=?");

    }

    @Override
    public void invoke(SensorReading value, Context context) throws Exception {
//        这个方法每来一条数据就会创建一个链接，所以这样很消耗资源，所以我们把这个链接放在open方法中
//        每来一条语句，就调用链接执行sql,设置占位符
        updatestat.setDouble(1,value.getTemperature());
        updatestat.setString(2,value.getId());
//        执行语句
        updatestat.execute();
//        如果没有更新成功，就执行插入操作
        if(updatestat.getUpdateCount() ==0){
            insertStatement.setString(1,value.getId());
            insertStatement.setDouble(2,value.getTemperature());
            insertStatement.execute();
        }
    }

    @Override
    public void close() throws Exception {
//        关闭所有的资源
        insertStatement.close();
        updatestat.close();
        connection.close();
    }
}
~~~

**案例**

~~~ java
public class SinkTestJDBC {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");


//        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

        map.addSink(new MyJDBCFun());
//        执行程序
        env.execute();

    }
}
~~~

#### Connectors

##### jdbc Connectors

~~~ java
public class Test13 {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<Stu> dataSource = env.fromElements(new Stu(2, "xiaorui", 25));

        dataSource.addSink(JdbcSink.sink("insert into t_student values (null,?,?)",
                (ps,value)->
                {
                    ps.setString(1,value.getName());
                    ps.setInt(2,value.getAge());
                }, new JdbcConnectionOptions.JdbcConnectionOptionsBuilder()
                .withUrl("jdbc:mysql://localhost:3306/test")
                .withUsername("root")
                .withPassword("root")
                .withDriverName("com.mysql.jdbc.Driver")
                .build()));

        env.execute();

    }
}

@Data
@AllArgsConstructor
@NoArgsConstructor
class Stu{
    private int id;
    private String name;
    private int age;
}
~~~

##### Kafka Producer

Flink 里已经提供了一些绑定的 Connector，例如 kafka source 和 sink，Es sink 等。读写kafka、es、rabbitMQ 时可以直接使用相应 connector 的 api 即可，虽然该部分是 Flink 项目源代码里的一部分，但是真正意义上不算作 Flink 引擎相关逻辑，并且该部分没有打包在二进制的发布包里面。所以在提交 Job 时候需要注意， job 代码
jar 包中一定要将相应的 connetor 相关类打包进去，否则在提交作业时就会失败，提示找不到相应的类，或初始化某些类异常。

**添加依赖**

~~~ java
 <dependency>
           <groupId>org.apache.flink</groupId>
           <artifactId>flink-connector-kafka_2.11</artifactId>
           <version>1.12.3</version>
</dependency>
~~~

**参数设置**

以下参数都必须/建议设置上

1. 订阅的主题

2. 反序列化规则

3. 消费者属性-集群地址

4. 消费者属性-消费者组id(如果不设置,会有默认的,但是默认的不方便管理)

5. 消费者属性-offset重置规则,如earliest/latest...

6. 动态分区检测(当kafka的分区数变化/增加时,Flink能够检测到!)

7. 如果没有设置Checkpoint,那么可以设置自动提交offset,后续学习了Checkpoint会把offset随着做Checkpoint的时候提交到Checkpoint和默认主题中

**参数设置**

![1621941296269](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/191501-976107.png)

![1621941335200](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/191548-220370.png)

![1621941395341](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/191636-455476.png)

实际的生产环境中可能有这样一些需求，比如：

- 场景一：有一个 Flink 作业需要将五份数据聚合到一起，五份数据对应五个 kafka topic，随着业务增长，新增一类数据，同时新增了一个 kafka topic，如何在不重启作业的情况下作业自动感知新的 topic。
- 场景二：作业从一个固定的 kafka topic 读数据，开始该 topic 有 10 个 partition，但随着业务的增长数据量变大，需要对 kafka partition 个数进行扩容，由 10 个扩容到 20。该情况下如何在不重启作业情况下动态感知新扩容的 partition？

针对上面的两种场景，首先需要在构建 FlinkKafkaConsumer 时的 properties 中设置 flink.partition-discovery.interval-millis 参数为非负值，表示开启动态发现的开关，以及设置的时间间隔。此时 FlinkKafkaConsumer 内部会启动一个单独的线程定期去 kafka 获取最新的 meta 信息。

针对场景一，还需在构建 FlinkKafkaConsumer 时，topic 的描述可以传一个正则表达式描述的 pattern。每次获取最新 kafka meta 时获取正则匹配的最新 topic 列表。

针对场景二，设置前面的动态发现参数，在定期获取 kafka 最新 meta 信息时会匹配新的 partition。为了保证数据的正确性，新发现的 partition 从最早的位置开始读取。

![1621943840230](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621943840230.png)

>  注意:
>
> 开启 checkpoint 时 offset 是 Flink 通过状态 state 管理和恢复的，并不是从 kafka 的 offset 位置恢复。在 checkpoint 机制下，作业从最近一次checkpoint 恢复，本身是会回放部分历史数据，导致部分数据重复消费，Flink 引擎仅保证计算状态的精准一次，要想做到端到端精准一次需要依赖一些幂等的存储系统或者事务操作。

**案例**

~~~ java
public class Test15 {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(2);

        //        添加kafka数据源
//        准备配置文件
        Properties props = new Properties();
        props.setProperty("bootstrap.servers", "node1:9092");//生产者组的ip
        props.setProperty("group.id", "flink");//消费者组
        props.setProperty("auto.offset.reset","latest");//设置offset,如果有offset记录，那么就从offset记录开始消费，没有的话从最新的消息消费
        props.setProperty("flink.partition-discovery.interval-millis","5000");//会开启一个后台线程每隔5s检测一下Kafka的分区情况，动态分区的检测
        props.setProperty("enable.auto.commit", "true");//自动提交
        props.setProperty("auto.commit.interval.ms", "2000");//自动提交时间间隔，会提交到默认的主题
//        创建kafka source
        FlinkKafkaConsumer<String> flink_kafka = new FlinkKafkaConsumer<>("flink_kafka", new SimpleStringSchema(), props);
//      使用source
        DataStreamSource<String> source = env.addSource(flink_kafka);
//        在这里可以基于kafka的数据源source做数据的清洗工作，然后在输出处理过后的数据

//          下面添加输出数据源
//        准备配置文件
        Properties properties = new Properties();
        props.setProperty("bootstrap.servers", "node1:9092");//生产者组的ip
//        创建kafka消费者
        FlinkKafkaProducer<String> flink_kafka_sink = new FlinkKafkaProducer<>("flink_kafka", new SimpleStringSchema(), properties);
//      添加kafka sink
        DataStreamSink<String> stringDataStreamSink = source.addSink(flink_kafka_sink);
      
        env.execute();

    }
}
~~~

## Flink四大基石

Flink之所以能这么流行，离不开它最重要的四个基石：Checkpoint、State、Time、Window。

![1622103858340](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/140526-259786.png)

**Checkpoint**

- 这是Flink最重要的一个特性。

- Flink基于Chandy-Lamport算法实现了一个分布式的一致性的快照，从而提供了一致性的语义。

- Chandy-Lamport算法实际上在1985年的时候已经被提出来，但并没有被很广泛的应用，而Flink则把这个算法发扬光大了。

- Spark最近在实现Continue streaming，Continue streaming的目的是为了降低处理的延时，其也需要提供这种一致性的语义，最终也采用了Chandy-Lamport这个算法，说明Chandy-Lamport算法在业界得到了一定的肯定。`https://zhuanlan.zhihu.com/p/53482103`

**State** 

- 提供了一致性的语义之后，Flink为了让用户在编程时能够更轻松、更容易地去管理状态，还提供了一套非常简单明了的State API，包括ValueState、ListState、MapState，BroadcastState。

**Time**

- 除此之外，Flink还实现了Watermark的机制，能够支持基于事件的时间的处理，能够容忍迟到/乱序的数据。

**Window**

- 另外流计算中一般在对流数据进行操作之前都会先进行开窗，即基于一个什么样的窗口上做这个计算。Flink提供了开箱即用的各种窗口，比如滑动窗口、滚动窗口、会话窗口以及非常灵活的自定义的窗口。

## Flink 中的 Window 

- 窗 口 是流式应用中 一类十分常见的操作 。它们可以在无限数据流上基于有界区间实现聚合等转换。通常情况下， 这些区间都是基于时间逻辑定义的 。窗口算子提供了一种基于有限大小的桶对事件进行分组， 并对这些桶中的有限内容进行计算的方法 。 

- streaming 流式计算是一种被设计用于处理无限数据集的数据处理引擎，而无限数据集是指一种不断增长的本质上无限的数据集，而 window 是一种切割无限数据为有限块进行处理的手段。Window 是无限数据流处理的核心， Window 将一个无限的 stream 拆分成有限大小的” buckets”桶，我们可以在这些桶上做计算操作。 

- 窗口算子可用在键值分区或非键值分区的数据流上。用于键值分区窗口的算子可以并行计算，而非键值分区窗口只能单线程处理。 

### 为什么需要Window

在流处理应用中，数据是连续不断的，有时我们需要做一些聚合类的处理，例如：在过去的1分钟内有多少用户点击了我们的网页。在这种情况下，我们必须定义一个窗口(window)，用来收集最近1分钟内的数据，并对这个窗口内的数据进行计算。

![1614567122089](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/105209-808469.png)

- 一般真实的流都是无界的，怎样处理无界的数据？
- 可以把无限的数据流进行切分，得到有限的数据集进行处理——也就是得到有界流
- 窗口（window）就是将无限流切割为有限流的一种方式，它会将流数据分发到有限大小的桶（bucket）中进行分析

### Window的分类

#### 按照time和count分类

time-window:时间窗口:根据时间划分窗口,如:每xx分钟统计最近xx分钟的数据

count-window:数量窗口:根据数量划分窗口,如:每xx个数据统计最近xx个数据

**图示**

![1622104270812](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/27/163118-133038.png)

#### 按照slide和size分类

窗口有两个重要的属性: 窗口大小size和滑动间隔slide,根据它们的大小关系可分为:

**滚动窗口（Tumbling Windows）**

滚动窗口:size=slide,如:每隔10s统计最近10s的数据

![1622104549235](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/27/163552-363085.png)

![1614568758764](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/111920-559392.png)

- 将数据依据固定的窗口长度对数据进行切分，窗口大小固定。
- 时间对齐，窗口长度固定，没有重叠
- 在时间节点上的数据可以自定义开闭区间
- 只需要定义窗口的大小即可
- DataStream  API 针对事件时间和处理时间的滚动窗口分别提供了对应的分配器TumblingEventTimeWindows 和 TumblingProcessingTimeWindows 
- 滚动窗口分配器只接受一个参数，以时间单元表示窗口的大小，他可以利用分配器的of(Time size)进行指定。

> 特点：时间对齐，窗口长度固定，没有重叠。 

**滑动窗口（Sliding Windows）**

滑动窗口:size>slide,如:每隔5s统计最近10s的数据

![1622104608127](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/27/163650-745134.png)

例如，你有 10 分钟的窗口和 5 分钟的滑动，那么每个窗口中 5 分钟的窗口里包含着上个 10 分钟产生的数据，如下图所示 

![1614568849756](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/112106-853588.png)

注意:当size<slide的时候,如每隔15s统计最近10s的数据,那么中间5s的数据会丢失,所有开发中不用

- 滑动窗口是固定窗口的更广义的一种形式，滑动窗口由固定的窗口长度和滑动间隔组成
- 窗口长度固定，可以有重叠
- 可以自定义滑动的间隔
- 滑动窗口分配器将元素分配给大小固定且按指定滑动间隔移动的窗口。 
- 对于滑动窗口而言，你需要指定窗口大小以及用于定义新窗口开始频率的滑动间隔。如果滑动间隔小于窗口大小，则窗口会出现重叠，此时元素会被分配给多个窗口:如果滑动间隔大于窗口大小，则一些元素可能不会分配给任何窗口，因此可能会被直接丢弃。 
- Data Stream API 提供了针对事件时间和处理时间的分配器以及相关的简写方怯 

> 特点：时间对齐，窗口长度固定， 可以有重叠 

**会话窗口（Session Windows）**

![1614568939743](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/27/162854-431280.png)

- 由一系列事件组合一个指定时间长度的timeout间隙组成，也就是一段时间没有接收到新数据就会生成新的窗口
- 特点：时间无对齐
- 可以指定回话间隔，也就是多长时间没有产生新的数据就开辟一个窗口
- 会话窗口将元素放入长度可变且不重叠的窗口中。会话窗口的边界由非活动间隔，即持续没有收到记录的时间间隔来定义 
- session 窗口分配器通过 session 活动来对元素进行分组， session 窗口跟滚动窗口和滑动窗口相比，不会有重叠和固定的开始时间和结束时间的情况，相反，当它在一个固定的时间周期内不再收到元素，即非活动间隔产生，那这个窗口就会关闭。一个 session 窗口通过一个 session 间隔来配置，这个 session 间隔定义了非活跃周期的长度，当这个非活跃周期产生，那么当前的 session 将关闭并且后续的元素将被分配到新的 session 窗口中去。 

> 特点：时间对齐，窗口长度固定， 可以有重叠 

**综上，窗口的划分如下**

- 时间窗口（Time Window）:按照时间来划分数据

  根据数据的移动规则进行划分

  - 滚动时间窗口，用的较多
  - 滑动时间窗口。用的较多
  - 会话窗口

- 计数窗口（Count Window）：按照数据的个数划分数据

  根据数据的移动规则进行划分

  - 滚动计数窗口，用的较少
  - 滑动计数窗口，用的较少

### Window Api

#### 总体概况

![1614587298004](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/162821-962794.png)

首先按照key进行分组，然后按照窗口对分组进行聚合操作。

Non-Keyed-Windows是直接对窗口中的数据进行聚合操作。

#### API类图

**api**

~~~ java
@PublicEvolving
public abstract class Window {

	/**
	 * Gets the largest timestamp that still belongs to this window.
	 *
	 * @return The largest timestamp that still belongs to this window.
	 */
	public abstract long maxTimestamp();
}
~~~

**继承结构**

![1614600461734](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/200745-895551.png)

**类图**

![1615716866515](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/14/181429-750953.png)

#### 窗口的创建

**新建窗口需要两个组件**

1. 一 个用于决定输入流中的元素该如何划分的窗口分配器( window assigner ) 。窗口分配器会产生一个WindowedStream (如果用在非键值分区的 DataStream 上则是 AIIWindowedStream ) 。 
2. 一 个作用于 WindowedStream (或 AIIWindowedStream ) 上，用于处理分配到窗口中元素的窗口函数。 

![1622876156662](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/145557-227196.png)

- 窗口分配器——window()方法
- 我们可以用.window()来定义一个窗口，然后基于这个window去做一些聚合或者其它处理操作。注意window ()方法必须在keyBy之后才能用。
- Flink提供了更加简单的.timeWindow和.countWindow方法，用于定义时间窗口和计数窗口。

![1614573542742](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/145605-88098.png)

##### 窗口分配器（window assigner）

- window()方法接收的输入参数是一个WindowAssigner
- WindowAssigner负责将每条输入的数据分发到正确的window中
- Flink提供了通用的WindowAssigner
  - 滚动窗口（tumbling window）
  - 滑动窗口（sliding window）
  - 会话窗口（session window）
  - 全局窗口（global window）

**分配器继承结构**

![1615717395768](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/14/182317-714546.png)

##### 创建不同类型的窗口

根据参数的个数来区分是滑动窗口还是滚动窗口。

- 滚动时间窗口（tumblingtime window）

Flink 默认的时间窗口根据 Processing Time 进行窗口的划分，将 Flink 获取到的数据根据进入 Flink 的时间划分到不同的窗口中 ，时间间隔可以通过 Time.milliseconds(x)， Time.seconds(x)， Time.minutes(x)等其中的一个来指定。 

~~~ java
//里面的参数是窗口分配器 
.window(TumblingProcessingTimeWindows.of(Time.seconds(15)));
.window(TumblingEventTimeWindows.of(Time.seconds(1)))
  //下面这种写法是上面两种方法的简写，通过参数的个数确定是滑动窗口还是滚动窗口
 .timeWindow(Time.seconds(15));
~~~

- 滑动时间窗口（sliding time window）

滑动窗口和滚动窗口的函数名是完全一致的，只是在传参数时需要传入两个参数，一个是 window_size，一个是 sliding_size。 时间间隔可以通过 Time.milliseconds(x)， Time.seconds(x)， Time.minutes(x)等其中的一个来指定 

~~~ java
//处理时间滑动窗口分配器，注意有两个参数
.window(SlidingProcessingTimeWindows.of(Time.hours(1), Time.seconds(15)))
//事件时间滑动窗口，注意两个参数
.window(SlidingEventTimeWindows.of(Time.hours(1), Time.seconds(15)))
//下面是简写，按照参数个数判断是时间窗口还是滑动窗口
.timeWindow(Time.seconds(12),Time.seconds(2));
~~~

- 会话窗口（session window）

~~~ JAVA
//事件时间会话窗口分配器
.window(EventTimeSessionWindows.withGap(Time.seconds(25)));
//处理时间会话窗口分配器
 .window(ProcessingTimeSessionWindows.withGap(Time.seconds(12)))
~~~

**计数窗口**

CountWindow 根据窗口中相同 key 元素的数量来触发执行，执行时只计算元素数量达到窗口大小的 key 对应的结果。 注意： CountWindow 的 window_size 指的是相同 Key 的元素的个数，不是输入的所有元素的总数。 

- 滚动计数窗口（tumblingcount window）

默认的 CountWindow 是一个滚动窗口，只需要指定窗口大小即可，当元素数量达到窗口大小时，就会触发窗口的执行。 

~~~ java
countWindow(10);
~~~

- 滑动计数窗口（sliding count window）

滑动窗口和滚动窗口的函数名是完全一致的，只是在传参数时需要传入两个参数，一个是 window_size，一个是 sliding_size。 下面代码中的 sliding_size 设置为了 2，也就是说，每收到两个相同 key 的数据
就计算一次，每一次计算的 window 范围是 10 个元素 

~~~ java
countWindow(10,2);
~~~

##### 窗口函数（window function）

**类图**

![1614607444753](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/220406-129546.png)

- window function定义了要对窗口中收集的数据做的计算操作
- 窗口函数定义了针对窗口内元素的计算逻辑。 

**函数分类**：两类

- 增量聚合函数（incremental aggregation functions）：类似于流处理的过程，每来一条数据，就在以前数据的基础上做聚合操作。
  - 它的应用场景是窗口内以状态形式存储某个值且需要根据每个加入窗口的元素对该值进行更新。此类函数通常会十分节省空间且最终会将聚合值作为单个结果发送出去。 
  - 每条数据到来就进行计算，保持一个简单的状态，
  - ReduceFunction, AggregateFunction
- 全窗口函数（full window functions）
  - 它会接收集窗口内的所有元素，并在执行计算时对它们进行遍历。虽然全量窗口函数通常需要占用更多空间，但它和增量聚合函数相比，支持更复杂的逻辑。 
  - 先把窗口所有数据收集起来，等到计算的时候会遍历所有数据。
  - ProcessWindowFunction，WindowFunction

###### **增量聚合函数**

**AggregateFunction接口**

~~~ java
@PublicEvolving
public interface AggregateFunction<IN, ACC, OUT> extends Function, Serializable {
// in输入类型，acc:累加操作，out:输出操作
	/**
	 * Creates a new accumulator, starting a new aggregate.
	 *
	 * <p>The new accumulator is typically meaningless unless a value is added
	 * via {@link #add(Object, Object)}.
	 *
	 * <p>The accumulator is the state of a running aggregation. When a program has multiple
	 * aggregates in progress (such as per key and window), the state (per key and window)
	 * is the size of the accumulator.
	 *
	 * @return A new accumulator, corresponding to an empty aggregate.
	 */
  ////创建一个累加器来启动聚合
	ACC createAccumulator();

	/**
	 * Adds the given input value to the given accumulator, returning the
	 * new accumulator value.
	 *
	 * <p>For efficiency, the input accumulator may be modified and returned.
	 *
	 * @param value The value to add
	 * @param accumulator The accumulator to add the value to
	 */
  //向 累加器中添加一个输入元素并返回累加器
	ACC add(IN value, ACC accumulator);

	/**
	 * Gets the result of the aggregation from the accumulator.
	 *
	 * @param accumulator The accumulator of the aggregation
	 * @return The final aggregation result.
	 */
  //根据累加器计算并返回结果
	OUT getResult(ACC accumulator);

	/**
	 * Merges two accumulators, returning an accumulator with the merged state.
	 *
	 * <p>This function may reuse any of the given accumulators as the target for the merge
	 * and return that. The assumption is that the given accumulators will not be used any
	 * more after having been passed to this function.
	 *
	 * @param a An accumulator to merge
	 * @param b Another accumulator to merge
	 *
	 * @return The accumulator with the merged state
	 */
  // 合并两个累加器并返回合并结果
	ACC merge(ACC a, ACC b);
}
//i去接口定 义 了输 入 类型 IN ， 累 加 器类型 ACC 以 及结果类型 OUT。它和ReduceFunction 不同的是中间数据类型 以及结果类型不再依赖输入类型。
//ReduceFunction 接收两个同类型的值井将它们组合生成一个类型不变的值。当被用在窗口化数据流上时， ReduceFunction 会对分配给窗口的元素进行增量聚合。窗口只需要存储当前聚合结果，一个和 ReduceFunction 的输入及输出类型都相同的值。每当收到一个新元素，算子都会以该元素和从窗口状态取出的当前聚合值为参数调用ReduceFunction ， 随后会用 ReduceFunction 的结果替换窗口状态。
@Public
@FunctionalInterface
public interface ReduceFunction<T> extends Function, Serializable {

	/**
	 * The core method of ReduceFunction, combining two values into one value of the same type.
	 * The reduce function is consecutively applied to all values of a group until only a single value remains.
	 *
	 * @param value1 The first value to combine.
	 * @param value2 The second value to combine.
	 * @return The combined value of both input values.
	 *
	 * @throws Exception This method may throw exceptions. Throwing an exception will cause the operation
	 *                   to fail and may trigger recovery.
	 */
	T reduce(T value1, T value2) throws Exception;
}
~~~

**案例**

~~~ java
public class TimeWindow {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

//        开窗测试
//        先做分组操作,也可以直接使用timewindow开窗口
        //                countWindow(10,2);

        //.window(EventTimeSessionWindows.withGap(Time.seconds(25)));
//                .timeWindow(Time.seconds(15));
//                window(TumblingProcessingTimeWindows.of(Time.seconds(15)));
//          做增量聚合操作
       /* map.keyBy("id").timeWindow(Time.seconds(15)).reduce(new ReduceFunction<SensorReading>() {
            @Override
            public SensorReading reduce(SensorReading value1, SensorReading value2) throws Exception {

            }
        });*/
//          使用AggregateFunction函数,实现count()计数功能
        DataStream<Integer> result = map.keyBy("id").timeWindow(Time.seconds(15)).aggregate(new AggregateFunction<SensorReading, Integer, Integer>() {
//      SensorReading:输入类型
//            integer:输出类型
//            Integer：输出类型

            @Override
            public Integer createAccumulator() {
//                创建一个累加器
                return 0;
            }

            @Override
            public Integer add(SensorReading value, Integer accumulator) {
//                做累加操作
                return accumulator + 1;
            }

            @Override
            public Integer getResult(Integer accumulator) {
//                获取结果
                return accumulator;
            }

            @Override
            public Integer merge(Integer a, Integer b) {
//                做分区合并操作
                return a + b;
            }
        });

        result.print();
//        map.print();
        env.execute();
    }
}
~~~

###### 全窗口函数

ReduceFunction 和 AggregateFunction 都是对分配到窗口的事件进行增量计算。 然而有些时候我们需要访问窗口内的所有元素来执行一些更加复杂的计算，例如计算窗口内数据的中值或出现频率最高的值。对于此类应用，ReduceFunction 和 AggregateFunction 都不适合。 FlinkDataStream API 提供的 ProcessWindowFunction 可以对窗口内容执行任意计算。 

**WindowFunction接口**

~~~ java
@Public
public interface WindowFunction<IN, OUT, KEY, W extends Window> extends Function, Serializable {
//in:输入 out:输出，key:键的类型，w:当前的window类型
  //key:默认是元祖类型
	/**
	 * Evaluates the window and outputs none or several elements.
	 *
	 * @param key The key for which this window is evaluated.
	 * @param window The window that is being evaluated.
	 * @param input The elements in the window being evaluated.
	 * @param out A collector for emitting elements.
	 * 
	 * @throws Exception The function may throw exceptions to fail the program and trigger recovery. 
	 */
	void apply(KEY key, W window, Iterable<IN> input, Collector<OUT> out) throws Exception;
}
//已经被ProcessWindowFunction取代
~~~

**ProcessWindowFunction接口**

~~~ java
/**
 * Base abstract class for functions that are evaluated over keyed (grouped) windows using a context
 * for retrieving extra information.
 *
 * @param <IN> The type of the input value.
 * @param <OUT> The type of the output value.
 * @param <KEY> The type of the key.
 * @param <W> The type of {@code Window} that this window function can be applied on.
 */
@PublicEvolving
public abstract class ProcessWindowFunction<IN, OUT, KEY, W extends Window> extends AbstractRichFunction {

	private static final long serialVersionUID = 1L;

	/**
	 * Evaluates the window and outputs none or several elements.
	 *
	 * @param key The key for which this window is evaluated.
	 * @param context The context in which the window is being evaluated.
	 * @param elements The elements in the window being evaluated.
	 * @param out A collector for emitting elements.
	 *
	 * @throws Exception The function may throw exceptions to fail the program and trigger recovery.
	 */
  // 对窗口执行计算
	public abstract void process(KEY key, Context context, Iterable<IN> elements, Collector<OUT> out) throws Exception;

	/**
	 * Deletes any state in the {@code Context} when the Window is purged.
	 *
	 * @param context The context to which the window is being evaluated
	 * @throws Exception The function may throw exceptions to fail the program and trigger recovery.
	 */
  // 在窗口清除时删除自定义的单个窗口状态
	public void clear(Context context) throws Exception {}

	/**
	 * The context holding window metadata.
	 */
  // 保存窗口无数据的上下文
	public abstract class Context implements java.io.Serializable {
		/**
		 * Returns the window that is being evaluated.
		 */
    // 返回窗口的元数据
		public abstract W window();

		/** Returns the current processing time. */
		public abstract long currentProcessingTime();

		/** Returns the current event-time watermark. */
    // 返回当前处理时间
		public abstract long currentWatermark();

		/**
		 * State accessor for per-key and per-window state.
		 *
		 * <p><b>NOTE:</b>If you use per-window state you have to ensure that you clean it up
		 * by implementing {@link ProcessWindowFunction#clear(Context)}.
		 */
		public abstract KeyedStateStore windowState();

		/**
		 * State accessor for per-key global state.
		 */
		public abstract KeyedStateStore globalState();

		/**
		 * Emits a record to the side output identified by the {@link OutputTag}.
		 *
		 * @param outputTag the {@code OutputTag} that identifies the side output to emit to.
		 * @param value The record to emit.
		 */
		public abstract <X> void output(OutputTag<X> outputTag, X value);
	}
}
~~~

**案例**

~~~ java
public class TimeWindow_fullwin {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });
//        全窗口
        SingleOutputStreamOperator<Tuple3<String,Long,Integer>> id = map.keyBy("id")
          //时间窗口
                .timeWindow(Time.seconds(15))
                .process(new ProcessWindowFunction<SensorReading, Object, Tuple, TimeWindow>() {
                    /**
                     *
                     * @param tuple
                     * @param context 上下文，里面包含window
                     * @param elements
                     * @param out
                     * @throws Exception
                     */
                    @Override
                    public void process(Tuple tuple, Context context, Iterable<SensorReading> elements, Collector<Object> out) throws Exception {
                      //处理逻辑
                    }
                })
                .apply(new WindowFunction<SensorReading, Tuple3<String,Long,Integer>, Tuple, TimeWindow>() {

                    /**
                     *和增量函数的区别就是，此函数使用Iterable吧数据全部拿来，然后在做处理，而增量是拿到一个数据，就处理一个数据
                     * @param tuple 当前的键
                     * @param window window类型
                     * @param input 当前的所有输入的数据
                     * @param out 输出数据，没有返回值类型，使用out输出
                     * @throws Exception
                     */
                    @Override
                    public void apply(Tuple tuple, TimeWindow window, Iterable<SensorReading> input, Collector<Tuple3<String,Long,Integer>> out) throws Exception {
                        String id=tuple.getField(0);//获取id，根据位置获取
                        Long windowAnd=window.getEnd();
                        int size = IteratorUtils.toList(input.iterator()).size();
                        out.collect(new Tuple3<>(id,windowAnd,size));
                    }
                });
        id.print();


        env.execute();
    }
}

//apply是windowFunction中的方法
//process是ProcessWindowFunction中的方法
~~~

###### 计数窗口

**接口**

~~~ java
@PublicEvolving
public interface AggregateFunction<IN, ACC, OUT> extends Function, Serializable {

	/**
	 * Creates a new accumulator, starting a new aggregate.
	 *
	 * <p>The new accumulator is typically meaningless unless a value is added
	 * via {@link #add(Object, Object)}.
	 *
	 * <p>The accumulator is the state of a running aggregation. When a program has multiple
	 * aggregates in progress (such as per key and window), the state (per key and window)
	 * is the size of the accumulator.
	 *
	 * @return A new accumulator, corresponding to an empty aggregate.
	 */
	ACC createAccumulator();

	/**
	 * Adds the given input value to the given accumulator, returning the
	 * new accumulator value.
	 *
	 * <p>For efficiency, the input accumulator may be modified and returned.
	 *
	 * @param value The value to add
	 * @param accumulator The accumulator to add the value to
	 */
	ACC add(IN value, ACC accumulator);

	/**
	 * Gets the result of the aggregation from the accumulator.
	 *
	 * @param accumulator The accumulator of the aggregation
	 * @return The final aggregation result.
	 */
	OUT getResult(ACC accumulator);

	/**
	 * Merges two accumulators, returning an accumulator with the merged state.
	 *
	 * <p>This function may reuse any of the given accumulators as the target for the merge
	 * and return that. The assumption is that the given accumulators will not be used any
	 * more after having been passed to this function.
	 *
	 * @param a An accumulator to merge
	 * @param b Another accumulator to merge
	 *
	 * @return The accumulator with the merged state
	 */
	ACC merge(ACC a, ACC b);
}
~~~

**案例**

~~~ java
public class CountWindow {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

//        开计数窗口测试
        SingleOutputStreamOperator<Double> result = map.keyBy("id")
                //统计5个数的平均值，每两个数滑动一次
                .countWindow(5, 2)
                .aggregate(new MyAvgTemp());
        result.print();


        env.execute();
    }

    public static class MyAvgTemp implements AggregateFunction<SensorReading, Tuple2<Double,Integer>,Double>{

        @Override
        public Tuple2<Double, Integer> createAccumulator() {
//            创建一个tuple2(),作为中间计算使用
            return new Tuple2<>(0.0,0);
        }

        @Override
        public Tuple2<Double, Integer> add(SensorReading value, Tuple2<Double, Integer> accumulator) {
//            加上当前记录的温度
            return new Tuple2<>(accumulator.f0+value.getTemperature(),accumulator.f1+1);
        }

        @Override
        public Double getResult(Tuple2<Double, Integer> accumulator) {
            return accumulator.f0/accumulator.f1;
        }

        @Override
        public Tuple2<Double, Integer> merge(Tuple2<Double, Integer> a, Tuple2<Double, Integer> b) {
            return new Tuple2<>(a.f0+b.f0,a.f1+b.f1);
        }
    }
}

~~~

##### 其他API

1. .trigger()——触发器：定义window什么时候关闭，触发计算并输出结果
2. .evictor()——移除器：定义移除某些数据的逻辑
3. .allowedLateness()——允许处理迟到的数据
4. .sideOutputLateData()——将迟到的数据放入侧输出流
5. .getSideOutput()——获取侧输出流

##### 案例

**基于时间的滚动和滑动窗口**

![1622878592124](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622878592124.png)

**代码演示**

~~~ java
//nc -lk 9999
public class Test17 {
    /*
    1,5
    2,5
    3,5
    4,5
     */
    public static void main(String[] args) throws Exception {
        /**
         * 基于事件的滚动和滑动窗口
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        基于socket读取数据
        DataStreamSource<String> socket = env.socketTextStream("hadoop100", 9999);

        SingleOutputStreamOperator<CarInfo> mapData = socket.map(new MapFunction<String, CarInfo>() {
            @Override
            public CarInfo map(String value) throws Exception {

                String[] s = value.split(" ");
                return new CarInfo(s[0], Integer.parseInt(s[1]));
            }
        });

//        每5秒钟统计一次，统计最近5秒内，各个红绿灯路口通过的汽车的数量，基于时间的滚动窗口
//        每5秒钟统计一次，统计最近10秒内，各个红绿灯路口通过的汽车的数量，基于时间的滑动窗口

//        需求中要的是各个路口/红绿灯的结果，所以需要先进行分组
        KeyedStream<CarInfo, String> keyedData = mapData.keyBy(item -> item.sensorId);

//        创建滑动窗口
        SingleOutputStreamOperator<CarInfo> res1 = keyedData.window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
                .sum("count");//做聚合操作

        SingleOutputStreamOperator<CarInfo> res2 = keyedData.window(SlidingProcessingTimeWindows.of(Time.seconds(10), Time.seconds(5)))
                .sum("count");


        res1.print();


        env.execute();
    }

    @NoArgsConstructor
    @AllArgsConstructor
    @Data
    public static class CarInfo{
        private String sensorId;//信号灯的id
        private Integer count;//通过该信号灯的车的数量
    }
}
~~~

**基于数量的滚动和滑动窗口**

![1622878655029](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622878655029.png)

**代码演示**

~~~ java
public class Test18 {
    /*
    1,5
    2,5
    3,5
    4,5
     */
    public static void main(String[] args) throws Exception {
        /**
         * 基于事件的滚动和滑动窗口
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        基于socket读取数据
        DataStreamSource<String> socket = env.socketTextStream("hadoop100", 9999);

        SingleOutputStreamOperator<CarInfo> mapData = socket.map(new MapFunction<String, CarInfo>() {
            @Override
            public CarInfo map(String value) throws Exception {

                String[] s = value.split(" ");
                return new CarInfo(s[0], Integer.parseInt(s[1]));
            }
        });

//        每5秒钟统计一次，统计最近5秒内，各个红绿灯路口通过的汽车的数量，基于时间的滚动窗口
//        每5秒钟统计一次，统计最近10秒内，各个红绿灯路口通过的汽车的数量，基于时间的滑动窗口

//        需求中要的是各个路口/红绿灯的结果，所以需要先进行分组
        KeyedStream<CarInfo, String> keyedData = mapData.keyBy(item -> item.sensorId);

        SingleOutputStreamOperator<CarInfo> res1 = keyedData.countWindow(5)
                .sum("count");

        SingleOutputStreamOperator<CarInfo> res2 = keyedData.countWindow(5, 3)
                .sum("count");

        res1.print();


        env.execute();
    }

    @NoArgsConstructor
    @AllArgsConstructor
    @Data
    public static class CarInfo{
        private String sensorId;//信号灯的id
        private Integer count;//通过该信号灯的车的数量
    }
}
~~~

**会话窗口**

![1622879583581](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/155446-57266.png)

**代码演示**

~~~ java
public class Test19 {
   
    public static void main(String[] args) throws Exception {
        /**
         * 基于事件的滚动和滑动窗口
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        基于socket读取数据
        DataStreamSource<String> socket = env.socketTextStream("hadoop100", 9999);

        SingleOutputStreamOperator<CarInfo> mapData = socket.map(new MapFunction<String, CarInfo>() {
            @Override
            public CarInfo map(String value) throws Exception {

                String[] s = value.split(" ");
                return new CarInfo(s[0], Integer.parseInt(s[1]));
            }
        });

//        每5秒钟统计一次，统计最近5秒内，各个红绿灯路口通过的汽车的数量，基于时间的滚动窗口
//        每5秒钟统计一次，统计最近10秒内，各个红绿灯路口通过的汽车的数量，基于时间的滑动窗口

//        需求中要的是各个路口/红绿灯的结果，所以需要先进行分组
        KeyedStream<CarInfo, String> keyedData = mapData.keyBy(item -> item.sensorId);

        SingleOutputStreamOperator<CarInfo> res = keyedData.window(ProcessingTimeSessionWindows.withGap(Time.seconds(10)))
                .sum("count");

        res.print();
        env.execute();
    }

    @NoArgsConstructor
    @AllArgsConstructor
    @Data
    public static class CarInfo{
        private String sensorId;//信号灯的id
        private Integer count;//通过该信号灯的车的数量
    }
}
~~~

##### Window API总览

![1614587298004](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/162821-962794.png)

## 时间语义

不做设置的话默认是处理时间语义。

### Flink 中的时间语义 

在 Flink 的流式处理中，会涉及到时间的不同概念，如下图所示： 

![1622880880702](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/161443-627644.png)

![1614644740170](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/17/105437-808486.png)

- Event Time：是事件创建的时间。它通常由事件中的时间戳描述，例如采集的日志数据中，每一条日志都会记录自己的生成时间， Flink 通过时间戳分配器访问事件时间戳。 指定算子根据数据自身包含的信息决定当前时间。每个事件时间都带有一个时间戳，而系统的逻辑时间是由水位线来定义。 

- Ingestion Time：是数据进入 Flink 的时间。指定每个接收的记录都把在数据源算子的处理时间作为事件时间的时间戳，并自动生成水位线。  

- Processing Time：是每一个执行基于时间操作的算子的**本地系统时间**，与机器相关，**默认的时间属性就是 Processing Time**。 指定算子根据处理机器的系统时钟决定数据流当前的时间。 处理时间窗口基于机器时间触发，它可以涵盖触发时间点之前到达算子的任意元素 

例如，一条日志进入 Flink 的时间为 2017-11-12 10:00:00.123，到达 Window 的系统时间为 2017-11-12 10:00:01.234，日志的内容如下： 

~~~ java
2017-11-02 18:37:15.624 INFO Fail over to rm2
~~~

对于业务来说，要统计 1min 内的故障日志个数，哪个时间是最有意义的？ ——eventTime，因为我们要根据日志的生成时间进行统计。 

### EventTime 的引入 

在 Flink 的流式处理中，绝大部分的业务都会使用 eventTime，一般只在eventTime 无法使用时，才会被迫使用 ProcessingTime 或者 IngestionTime。

如果要使用 EventTime，那么需要引入 EventTime 的时间属性，引入方式如下所示：  

~~~ java
public class CountWindow_eventtime {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        设置时间语义,事件时间语义
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        env.execute();
    }
}
~~~

### Watermark 

水位线用于告知算子不必再等那些时间戳小于或等于水位线的事件 。水位线的等待是什么意思，就是直接把时间调的慢多少分钟，这样就相当于等待了若干分钟。

![1622882370631](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/163935-652122.png)

#### 水位线（Watermark）

- 怎样避免乱序数据带来计算不正确？
- 遇到一个时间戳达到了窗口关闭时间，不应该立刻触发窗口计算，而是等待一段时间，等迟到的数据来了再关闭窗口
- Watermark是一种衡量Event Time进展的机制，可以设定延迟触发
- Watermark是用于处理乱序事件的，而正确的处理乱序事件，通常用Watermark机制结合window来实现；
- 数据流中的Watermark用于表示timestamp小于Watermark的数据，都已经到达了，因此，window的执行也是由Watermark触发的。
- watermark用来让程序自己平衡延迟和结果正确性

**作用**

![1622882548416](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/164231-609979.png)

#### 基本概念

我们知道，流处理从事件产生，到流经 source，再到 operator，中间是有一个过程和时间的，虽然大部分情况下，流到 operator 的数据都是按照事件产生的时间顺序来的，但是也不排除由于网络、分布式等原因，导致乱序的产生，所谓乱序，就是指 Flink 接收到的事件的先后顺序不是严格按照事件的 Event Time 顺序排的。 

![1614655834182](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/19/092749-699206.png)

在这里设置的水位线相当于时间调慢了一段时间，对于乱序到来的数据，比如对于事件时间是5的数据，设置延迟是3秒，那么事件时间是5的数据到来是只相当于当前进展到5-3=2秒，事件时间是6秒的数据到来时候只相当于当前进展到3秒。也就是对每一条到来的数据，统一延迟3秒钟。当事件时间是8的时间到来时候，8-3=5，此时说明前面0-5秒的数据已经全部到来，可以关闭窗口，因为这里设置窗口的大小是5。

watermark是用来处理乱序事件的，waterMark一般设置一个相对较小的统一的延迟时间。

那么此时出现一个问题，一旦出现乱序，如果只根据 eventTime 决定 window 的运行，我们不能明确数据是否全部到位，但又不能无限期的等下去，此时必须要有个机制来保证一个特定的时间后，必须触发 window 去进行计算了，这个特别的机制，就是 Watermark。 

- Watermark 是一种衡量 Event Time 进展的机制 
- Watermark 是用于处理乱序事件的，而正确的处理乱序事件，通常用Watermark 机制结合 window 来实现。 
- 数据流中的 Watermark 用于表示 timestamp 小于 Watermark 的数据，都已经到达了（如何理解这句话，比如说watermark允许延迟的时间是3，那么事件时间为5的数据到达时候，那么当前的watermarks是2，说明事件时间为2的数据以及之前的数据已经到齐了），因此， window 的执行也是由 Watermark 触发的。 
- Watermark 可以理解成一个延迟触发机制，我们可以设置 Watermark 的延时时长 t，每次系统会校验已经到达的数据中最大的 maxEventTime，然后认定 eventTime小于 maxEventTime - t 的所有数据都已经到达，如果有窗口的停止时间等于maxEventTime – t，那么这个窗口被触发执行。 

有序流的 Watermarker 如下图所示：（ Watermark 设置为 0） 

![1614656042731](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614656042731.png)

乱序流的 Watermarker 如下图所示：（ Watermark 设置为 2） 

![1614656067281](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/113429-644739.png)

当 Flink 接收到数据时， 会按照一定的规则去生成 Watermark，这条 Watermark就等于当前所有到达数据中的 maxEventTime - 延迟时长，也就是说， Watermark 是基于数据携带的时间戳生成的，一旦 Watermark 比当前未触发的窗口的停止时间要晚，那么就会触发相应窗口的执行。由于 event time 是由数据携带的，因此，如果运行过程中无法获取新的数据，那么没有被触发的窗口将永远都不被触发。 

上图中，我们设置的允许最大延迟到达时间为 2s，所以时间戳为 7s 的事件对应的 Watermark 是 5s，时间戳为 12s 的事件的 Watermark 是 10s，如果我们的窗口 1是 1s~5s，窗口 2 是 6s~10s，那么时间戳为 7s 的事件到达时的 Watermarker 恰好触发窗口 1，时间戳为 12s 的事件到达时的 Watermark 恰好触发窗口 2。

**Watermark 就是触发前一窗口的“关窗时间”，一旦触发关门那么以当前时刻为准在窗口范围内的所有所有数据都会收入窗中。**
只要没有达到水位那么不管现实中的时间推进了多久都不会触发关窗。 

![1622889433596](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/183717-675917.png)

#### watermark详解

![1623129086219](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/131127-526004.png)![1623129595750](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/131957-442663.png)

#### watermark的特点

![1614659939633](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/03/101819-143027.png)

watermark=2插入数据流，说明事件时间为2的数据已经全部到达，接下来、有事件时间为5的数据带来，所以插入watermark=5，也就是说事件时间为5以及之前的数据已经全部到达，但是后来又有事件时间为3的数据到达，但是现在watermark是5，说明事件时间是3的数据已经迟到。watermark的时间是根据数据的事件时间产生的。

-  watermark是一条特殊的数据记录

~~~ java
@PublicEvolving
public final class Watermark extends StreamElement {

	/** The watermark that signifies end-of-event-time. */
	public static final Watermark MAX_WATERMARK = new Watermark(Long.MAX_VALUE);

	// ------------------------------------------------------------------------

	/** The timestamp of the watermark in milliseconds. */
	private final long timestamp;  //watermark其实就是一个带着时间戳的数据，可以直接插入数据流中，代表时间的推移

	/**
	 * Creates a new watermark with the given timestamp in milliseconds.
	 */
	public Watermark(long timestamp) {
		this.timestamp = timestamp;
	}

	/**
	 * Returns the timestamp associated with this {@link Watermark} in milliseconds.
	 */
	public long getTimestamp() {
		return timestamp;
	}

	// ------------------------------------------------------------------------

	@Override
	public boolean equals(Object o) {
		return this == o ||
				o != null && o.getClass() == Watermark.class && ((Watermark) o).timestamp == this.timestamp;
	}

	@Override
	public int hashCode() {
		return (int) (timestamp ^ (timestamp >>> 32));
	}

	@Override
	public String toString() {
		return "Watermark @ " + timestamp;
	}
}
~~~

- watermark必须单调递增，以确保任务的事件时间时钟在向前推进，而不是在后退
- watermark与数据的时间戳相关

watermart就代表一个含义，其之前的数据已经全部到齐了，所以watermark的设置要保证其前面的数据全部到齐，假如说当前设置的延迟时间戳是s秒，延迟时间的设置主要看最大的迟到程度，每次使用当前到来的数据的时间戳减去延迟时间进行比较，如果结果小于零，或者小于当前的watermark的值，那么watermark还使用之前的值，否则人进行更新操作，那么窗口什么时候关闭呢？一直更新watermark直到其值等于窗口大小就关闭一次窗口操作，根据watermark的值判断是否需要关闭窗口。如果窗口关闭之后，还有迟到的数据，这个时候可以输出到测输出流中。

watermark表示的数据到齐与否是左闭右开的，设置延迟的时间应该根据最大的迟到时间差来。watermark的设置是当前数据的事件时间-设置的延迟时间

下面的图，同时可以存在多个桶，然后依次把数据分配到桶中，触发桶的执行是根据watermark进行的，当事件时间为8的数据到达后，8-3=5，此时说明前5秒的数据已经全部到达，注意这里不包含事件时间为5秒的数据，此时第一个桶，也就是1234所在的桶被触发执行。

时间语义的三种保证：

- watermark
- `.allowedLateness(Time.seconds(1))`允许迟到一段时间
- 使用侧输出流进行输出

**图解**

![1614661198149](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/03/101822-97793.png)

#### watermark的传递

上面考虑的是一条流，相当于一个分区，对于多个任务之间，水位线要以最小的那个值为准

![1614661794820](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/135527-780703.png)

#### Watermark 的引入 

![1614683588914](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/191311-558140.png)

Event Time的使用一定要指定数据源中的时间戳调用assignTimestampAndWatermarks方法，传入一个BoundedOutOfOrdernessTimestampExtractor，就可以指定watermark，在底层提取时间要求的是毫秒数。

水位线是根据时间戳生成的。

watermark 的引入很简单，对于乱序数据，最常见的引用方式如下： 

~~~ java
//                这种方式是周期性生成watermart,也可以随机生成
//BoundedOutOfOrdernessTimestampExtractor这个类的作用是提取时间戳
                .assignTimestampsAndWatermarks(new 
                   //设置watermark的延迟时间是2秒，表示最大的乱序程度                            BoundedOutOfOrdernessTimestampExtractor<SensorReading>(Time.seconds(2)) {

//            提取每一个数据的时间戳
            @Override
            public long extractTimestamp(SensorReading element) {
//                返回的时间戳是毫秒数
                return element.getTempStamp()*1000l;
            }
        });

//                如果数据已经知道有序，没有乱序发生，就不用设置延迟，不用设置延迟时间，不需要延迟触发，可以只指定时间戳就行了
//                .assignTimestampsAndWatermarks(new AscendingTimestampExtractor<SensorReading>() {
//                    @Override
//                    public long extractAscendingTimestamp(SensorReading element) {
//                        return element.getTempStamp()*1000l;
//                    }
//                })
~~~

**时间戳接口**

~~~ java
public interface AssignerWithPeriodicWatermarks<T> extends TimestampAssigner<T>{}
public interface AssignerWithPunctuatedWatermarks<T> extends TimestampAssigner<T> {}

//TimestampAssigner接口中必须指明如何提取时间戳
public interface TimestampAssigner<T> extends Function {

	/**
	 * Assigns a timestamp to an element, in milliseconds since the Epoch.
	 *
	 * <p>The method is passed the previously assigned timestamp of the element.
	 * That previous timestamp may have been assigned from a previous assigner,
	 * by ingestion time. If the element did not carry a timestamp before, this value is
	 * {@code Long.MIN_VALUE}.
	 *
	 * @param element The element that the timestamp will be assigned to.
	 * @param previousElementTimestamp The previous internal timestamp of the element,
	 *                                 or a negative value, if no timestamp has been assigned yet.
	 * @return The new timestamp.
	 */
	long extractTimestamp(T element, long previousElementTimestamp);
}


~~~

![1614662515762](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614662515762.png)

~~~ java
//BoundedOutOfOrdernessTimestampExtractor其实就是一个周期性生成时间戳的类
public abstract class BoundedOutOfOrdernessTimestampExtractor<T> implements AssignerWithPeriodicWatermarks<T> {

	private static final long serialVersionUID = 1L;

	/** The current maximum timestamp seen so far. */
  当前最大的时间戳
	private long currentMaxTimestamp;

	/** The timestamp of the last emitted watermark. */
  上一次发出的时间戳
	private long lastEmittedWatermark = Long.MIN_VALUE;

	/**
	 * The (fixed) interval between the maximum seen timestamp seen in the records
	 * and that of the watermark to be emitted.
	 */
  最大的延迟时间，也就是waternark的延迟时间 
	private final long maxOutOfOrderness;

	public BoundedOutOfOrdernessTimestampExtractor(Time maxOutOfOrderness) {
		if (maxOutOfOrderness.toMilliseconds() < 0) {
			throw new RuntimeException("Tried to set the maximum allowed " +
				"lateness to " + maxOutOfOrderness + ". This parameter cannot be negative.");
		}
    设置最大延迟时间
		this.maxOutOfOrderness = maxOutOfOrderness.toMilliseconds();
    最大的时间戳，加上延迟时间是因为后面还要减去最大的时间戳保证数据不会溢出
		this.currentMaxTimestamp = Long.MIN_VALUE + this.maxOutOfOrderness;
	}

	public long getMaxOutOfOrdernessInMillis() {
		return maxOutOfOrderness;
	}

	/**
	 * Extracts the timestamp from the given element.
	 *
	 * @param element The element that the timestamp is extracted from.
	 * @return The new timestamp.
	 */
	public abstract long extractTimestamp(T element);

	@Override
	public final Watermark getCurrentWatermark() {
		// this guarantees that the watermark never goes backwards.
    潜在的watermark=最大的时间戳减去最大的延迟时间
		long potentialWM = currentMaxTimestamp - maxOutOfOrderness;
    判断潜在的watermark是否大于上一次发出去的watermark，因为watermark是递增的一个时间
		if (potentialWM >= lastEmittedWatermark) {
      更新watermark时间
			lastEmittedWatermark = potentialWM;
		}
		return new Watermark(lastEmittedWatermark);
	}
//提取时间戳的方法
	@Override
	public final long extractTimestamp(T element, long previousElementTimestamp) {
		long timestamp = extractTimestamp(element);
		if (timestamp > currentMaxTimestamp) {
			currentMaxTimestamp = timestamp;
		}
		return timestamp;
	}
}
AscendingTimestampExtractor
相当于仅仅延迟一毫秒时间
@Override
	public final Watermark getCurrentWatermark() {
		return new Watermark(currentTimestamp == Long.MIN_VALUE ? Long.MIN_VALUE : currentTimestamp - 1);
	}
~~~

`AscendingTimestampExtractor`也是周期性生成

![1614662990403](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/133541-295517.png)

`AssignerWithPunctuatedWatermarks`没有被实现

![1614663059985](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/133103-949255.png)

Event Time 的使用一定要指定数据源中的时间戳。否则程序无法知道事件的事件时间是什么(数据源里的数据没有时间戳的话，就只能使用 Processing Time 了)。 

我们看到上面的例子中创建了一个看起来有点复杂的类，这个类实现的其实就是分配时间戳的接口。 Flink 暴露了 TimestampAssigner 接口供我们实现，使我们可以自定义如何从事件数据中抽取时间戳。

~~~ java
StreamExecutionEnvironment env =
StreamExecutionEnvironment.getExecutionEnvironment();
// 设置事件时间语义
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
DataStream<SensorReading> dataStream = env.addSource(new SensorSource())
.assignTimestampsAndWatermarks(new MyAssigner())
~~~

MyAssigner 有两种类型，都是继承 TimestampAssigner 

- AssignerWithPeriodicWatermarks
  - 周期性的生成watermark：系统会周期性的将watermark插入到流中
  - 默认周期是200毫秒，可以使用ExecutionConfig.setAutoWatermarkInterval()方法进行设置
  - 升序和前面乱序的处理BoundedOutOfOrdernessTimestampExtractor，都是基于周期性watermark的。
-  AssignerWithPunctuatedWatermarks
  - 没有时间周期规律，可打断的生成watermark

以上两个接口都继承自 TimestampAssigner。 

##### Assigner with periodic watermarks 

周期性的生成 watermark：系统会周期性的将 watermark 插入到流中(水位线也是一种特殊的事件!)。默认周期是 200 毫秒。可以使用ExecutionConfig.setAutoWatermarkInterval()方法进行设置。 

~~~ java
// 每隔 5 秒产生一个 watermark
env.getConfig.setAutoWatermarkInterval(5000);
~~~

产生 watermark 的逻辑：每隔 5 秒钟， Flink 会调用AssignerWithPeriodicWatermarks 的 getCurrentWatermark()方法。如果方法返回一个 时间戳大于之前水位的时间戳，新的 watermark 会被插入到流中。这个检查保证了水位线是单调递增的。如果方法返回的时间戳小于等于之前水位的时间戳，则不会产生新的 watermark。 

例子，自定义一个周期性的时间戳抽取： 

~~~ java
// 自定义周期性时间戳分配器
public static class MyPeriodicAssigner implements
        AssignerWithPeriodicWatermarks<SensorReading>{
    private Long bound = 60 * 1000L; // 延迟一分钟
    private Long maxTs = Long.MIN_VALUE; // 当前最大时间戳
    @Nullable
    @Override
    public Watermark getCurrentWatermark() {
        return new Watermark(maxTs - bound);
    } @
            Override
    public long extractTimestamp(SensorReading element, long previousElementTimestamp)
    {
        maxTs = Math.max(maxTs, element.getTimestamp());
        return element.getTimestamp();
    }
}
~~~

一种简单的特殊情况是， 如果我们事先得知数据流的时间戳是单调递增的，也就是说没有乱序， 那我们可以使用 AscendingTimestampExtractor， 这个类会直接使用数据的时间戳生成 watermark。 

~~~ java
DataStream<SensorReading> dataStream = …
        dataStream.assignTimestampsAndWatermarks(
        new AscendingTimestampExtractor<SensorReading>() {
@Override
public long extractAscendingTimestamp(SensorReading element) {
        return element.getTimestamp() * 1000;
        }
});
~~~

而对于乱序数据流， 如果我们能大致估算出数据流中的事件的最大延迟时间，就可以使用如下代码： 

~~~ java
DataStream<SensorReading> dataStream = …
dataStream.assignTimestampsAndWatermarks(
  //       水位线BoundedOutOfOrdernessTimestampExtractor参数是最大的延迟时间
new BoundedOutOfOrdernessTimestampExtractor<SensorReading>(Time.seconds(1)) {
@Override
public long extractTimestamp(SensorReading element) {
return element.getTimestamp() * 1000L;
}
});
~~~

##### Assigner with punctuated watermarks 

间断式地生成 watermark。和周期性生成的方式不同，这种方式不是固定时间的，而是可以根据需要对每条数据进行筛选和处理。 直接上代码来举个例子， 我们只给sensor_1 的传感器的数据流插入 watermark： 

~~~ java
public static class MyPunctuatedAssigner implements
        AssignerWithPunctuatedWatermarks<SensorReading>{
    private Long bound = 60 * 1000L; // 延迟一分钟
    @Nullable
    @Override
    public Watermark checkAndGetNextWatermark(SensorReading lastElement, long
            extractedTimestamp) {
        if(lastElement.getId().equals("sensor_1"))
            return new Watermark(extractedTimestamp - bound);
        else
            return null;
    }
    @Override
    public long extractTimestamp(SensorReading element, long previousElementTimestamp)
    {
        return element.getTimestamp();
    }
}
~~~

**对于迟到的数据，写入到测输出流中**

~~~ java
public class CountWindow_eventtime_ {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        //env.setParallelism(1);

//        设置时间语义,事件时间语义
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        env.getConfig().setAutoWatermarkInterval(100);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

//       水位线BoundedOutOfOrdernessTimestampExtractor参数是最大的延迟时间
        map
//                这种方式是周期性生成watermart,也可以随机生成
                .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<SensorReading>(Time.seconds(2)) {

//            提取每一个数据的时间戳
            @Override
            public long extractTimestamp(SensorReading element) {
//                返回的时间戳是毫秒数
                return element.getTempStamp()*1000l;
            }
        });


        OutputTag<SensorReading> late = new OutputTag<>("late");
//        基于事件时间的开窗工作
//        统计3秒内温度的最小值
        SingleOutputStreamOperator<SensorReading> minTemp = map.keyBy("id")
                .timeWindow(Time.seconds(2))//2秒滚动一个窗口
                .allowedLateness(Time.seconds(5))//允许迟到的时间
                .sideOutputLateData(late)//迟到的数据，放进测输出流中
                .minBy("temperature");

        minTemp.print();

//        输出测输出流中的数据
        minTemp.getSideOutput(late).print("late");

        env.execute();
    }

}
~~~

#### watermark的设定

- 在Flink中，watermark由应用程序开发人员生成，这通常需要对相应的领域有一定的了解
- 如果watermark设置的延迟太久，收到结果的速度可能就会很慢，解决办法是在水位线到达之前输出一个近似结果
- 而如果watermark到达得太早，则可能收到错误结果，不过Flink处理迟到数据的机制可以解决这个问题

**周期性生成中周期的设定**

~~~ java
@PublicEvolving
	public void setStreamTimeCharacteristic(TimeCharacteristic characteristic) {
		this.timeCharacteristic = Preconditions.checkNotNull(characteristic);
		if (characteristic == TimeCharacteristic.ProcessingTime) {
      //如果是处理时间，设置时间是0
			getConfig().setAutoWatermarkInterval(0);
		} else {
      //事件时间设置的周期是200毫秒
			getConfig().setAutoWatermarkInterval(200);
		}
	}

//初始窗口的设定
@Override
	public Collection<TimeWindow> assignWindows(Object element, long timestamp, WindowAssignerContext context) {
		if (timestamp > Long.MIN_VALUE) {
			// Long.MIN_VALUE is currently assigned when no timestamp is present
			long start = TimeWindow.getWindowStartWithOffset(timestamp, offset, size);
			return Collections.singletonList(new TimeWindow(start, start + size));
		} else {
			throw new RuntimeException("Record has Long.MIN_VALUE timestamp (= no timestamp marker). " +
					"Is the time characteristic set to 'ProcessingTime', or did you forget to call " +
					"'DataStream.assignTimestampsAndWatermarks(...)'?");
		}
	}

/**
	 * Method to get the window start for a timestamp.
	 *
	 * @param timestamp epoch millisecond to get the window start.
	 * @param offset The offset which window start would be shifted by.
	 * @param windowSize The size of the generated windows.
	 * @return window start
	 */
offset是一个偏移量，也就是起始点的位置时间发生一点偏移
	public static long getWindowStartWithOffset(long timestamp, long offset, long windowSize) {
		return timestamp - (timestamp - offset + windowSize) % windowSize;
	}
也就是使用初始的时间戳-初试时间戳对窗口的大小取模，第一个窗口确定，后面所有窗口都确定

//在watermark之后设置数据可以迟到多长时间
.allowedLateness(Time.seconds(1))
  
//三条线保证数据不会丢失
   .keyBy(ApacheLogEvent::getUrl)//按照url进行分组操作
//                窗口的大小是5，窗口滑动的步长是10
                .timeWindow(Time.minutes(5), Time.minutes(10))
//                在窗口中设置允许迟到数据,上面设置watermark可以等待一秒，在窗口这里在允许等待一分钟
                .allowedLateness(Time.seconds(1))
//                在这里设置侧输出流
                .sideOutputLateData(outputTag)
~~~

#### 并行度问题

63集

假如说输入的数据流有多条，也就是有多个source，那么多条数据流的watermark之间不会相互影响，但是如果只有一条输入流，但是每一个算子的并行度不同，那么数据在多个任务之间的传输，watermark会遵循木桶原则，也就是根据最小的waternark值设置自己的watermark的值。

![1614737979924](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/03/101942-178134.png)

在map和kb之间，map的并行度是4，map之后的每一个任务都会广播自己的watermark值给后边的kb任务，kb任务会选取所有的watermark的最小值来设置自己的watermark值。

~~~ java

public class CountWindow_eventtime_ {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1，并行度不是1的情况
        //env.setParallelism(1);

//        设置时间语义,事件时间语义
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        env.getConfig().setAutoWatermarkInterval(100);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

//       水位线BoundedOutOfOrdernessTimestampExtractor参数是最大的延迟时间
        map
//                这种方式是周期性生成watermart,也可以随机生成
                .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<SensorReading>(Time.seconds(2)) {

//            提取每一个数据的时间戳
            @Override
            public long extractTimestamp(SensorReading element) {
//                返回的时间戳是毫秒数
                return element.getTempStamp()*1000l;
            }
        });


        OutputTag<SensorReading> late = new OutputTag<>("late");
//        基于事件时间的开窗工作
//        统计3秒内温度的最小值
        SingleOutputStreamOperator<SensorReading> minTemp = map.keyBy("id")
                .timeWindow(Time.seconds(2))//2秒滚动一个窗口
                .allowedLateness(Time.seconds(5))//允许迟到的时间
                .sideOutputLateData(late)//迟到的数据，放进测输出流中
                .minBy("temperature");

        minTemp.print();

//        输出测输出流中的数据
        minTemp.getSideOutput(late).print("late");

        env.execute();
    }

}
~~~

#### 案例

![1623131796430](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623131796430.png)

~~~ java
public class Test20 {

    public static void main(String[] args) throws Exception {
        /**
         * 基于事件的滚动和滑动窗口
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        基于socket读取数据
//        DataStreamSource<String> socket = env.socketTextStream("hadoop100", 9999);

        DataStreamSource<Order> orderDataStreamSource = env.addSource(new SourceFunction<Order>() {
            private boolean flag=true;
            @Override
            public void run(SourceContext<Order> ctx) throws Exception {
                Random random = new Random();
                while (flag){
//                    产生订单的id
                    String id = UUID.randomUUID().toString();
//                    产生用户id
                    int userId = random.nextInt(2);
                    int money = random.nextInt(101);//0--100
//                    产生随机的延迟时间
                    long evenTime=System.currentTimeMillis()-random.nextInt(5)* 1000;
                    ctx.collect(new Order(id,userId,money,evenTime));
                    Thread.sleep(2000);
                }
            }
            @Override
            public void cancel() {
                flag=false;
            }
        });

//        基于事件时间的窗口计算+watermark
//        求每一个用户订单的总金额，最近5秒钟
//        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
//        在新版本中默认使用的是事件时间
//        设置水位线
        SingleOutputStreamOperator<Order> orderSingleOutputStreamOperator = orderDataStreamSource.assignTimestampsAndWatermarks(WatermarkStrategy.
                <Order>forBoundedOutOfOrderness(Duration.ofSeconds(3))//设置延迟时间，也即是最大的乱序事件
                .withTimestampAssigner((order, timestamp) -> order.evevTime));//设置事件时间


        SingleOutputStreamOperator<Order> money = orderSingleOutputStreamOperator.keyBy(order -> order.getOrderId())
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))
                .sum("money");

        money.print();
        env.execute();
    }

    @NoArgsConstructor
    @AllArgsConstructor
    @Data
    public static class Order {
        private String orderId;
        private Integer userId;
        private Integer money;
        private Long evevTime;
    }

}
~~~

#### 源码说明

~~~ java
SingleOutputStreamOperator<Order> orderSingleOutputStreamOperator = orderDataStreamSource.assignTimestampsAndWatermarks(WatermarkStrategy.
                <Order>forBoundedOutOfOrderness(Duration.ofSeconds(3))//设置延迟时间，也即是最大的乱序事件
                .withTimestampAssigner((order, timestamp) -> order.evevTime));//设置事件时间

static <T> WatermarkStrategy<T> forBoundedOutOfOrderness(Duration maxOutOfOrderness) {
		return (ctx) -> new BoundedOutOfOrdernessWatermarks<>(maxOutOfOrderness);
	}

public BoundedOutOfOrdernessWatermarks(Duration maxOutOfOrderness) {
		checkNotNull(maxOutOfOrderness, "maxOutOfOrderness");
		checkArgument(!maxOutOfOrderness.isNegative(), "maxOutOfOrderness cannot be negative");

		this.outOfOrdernessMillis = maxOutOfOrderness.toMillis();

		// start so that our lowest watermark would be Long.MIN_VALUE.
  //时间戳的初始值
		this.maxTimestamp = Long.MIN_VALUE + outOfOrdernessMillis + 1;
	}

@Override
	public void onEvent(T event, long eventTimestamp, WatermarkOutput output) {
   //获取当前进来的数据的最大时间戳
		maxTimestamp = Math.max(maxTimestamp, eventTimestamp);
	}

@Override
	public void onPeriodicEmit(WatermarkOutput output) {
    //当前最大时间戳-允许延迟的时间
		output.emitWatermark(new Watermark(maxTimestamp - outOfOrdernessMillis - 1));
	}
~~~

#### 数据丢失问题

![1623132921218](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623132921218.png)

在这个例子中，在D数据到来时候，会触发计算，所以会导致数据A丢失问题，所以需要引入侧输出 流机制。

![1623133116787](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623133116787.png)

~~~ java
public class Test21 {

    public static void main(String[] args) throws Exception {
        /**
         * 基于事件的滚动和滑动窗口
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        基于socket读取数据
//        DataStreamSource<String> socket = env.socketTextStream("hadoop100", 9999);

        DataStreamSource<Order> orderDataStreamSource = env.addSource(new SourceFunction<Order>() {
            private boolean flag=true;
            @Override
            public void run(SourceContext<Order> ctx) throws Exception {
                Random random = new Random();
                while (flag){
//                    产生订单的id
                    String id = UUID.randomUUID().toString();
//                    产生用户id
                    int userId = random.nextInt(2);
                    int money = random.nextInt(101);//0--100
//                    产生随机的延迟时间,有可能导致数据丢失
                    long evenTime=System.currentTimeMillis()-random.nextInt(5)* 1000;
                    ctx.collect(new Order(id,userId,money,evenTime));
                    Thread.sleep(2000);
                }
            }
            @Override
            public void cancel() {
                flag=false;
            }
        });

//        用来存放迟到的数据
        OutputTag<Order> tag = new OutputTag<>("data", TypeInformation.of(Order.class));

//        基于事件时间的窗口计算+watermark
//        求每一个用户订单的总金额，最近5秒钟
//        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
//        在新版本中默认使用的是事件时间
//        设置水位线
        SingleOutputStreamOperator<Order> orderSingleOutputStreamOperator = orderDataStreamSource.assignTimestampsAndWatermarks(WatermarkStrategy.
                <Order>forBoundedOutOfOrderness(Duration.ofSeconds(3))//设置延迟时间，也即是最大的乱序事件
                .withTimestampAssigner((order, timestamp) -> order.evevTime));//设置事件时间


        SingleOutputStreamOperator<Order> money = orderSingleOutputStreamOperator.keyBy(order -> order.getOrderId())
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))
                .allowedLateness(Time.seconds(3))//允许延迟
                .sideOutputLateData(tag)//延迟的数据存放的位置
                .sum("money");

//        打印正常到来的数据
        money.print();
//        打印侧输出流中的结果
        System.out.println(money.getSideOutput(tag));
        env.execute();
    }

    @NoArgsConstructor
    @AllArgsConstructor
    @Data
    public static class Order {
        private String orderId;
        private Integer userId;
        private Integer money;
        private Long evevTime;
    }
}
~~~

## Flink状态管理

目前Flink已经可以做到状态的自动管理。

#### 无状态计算

![1623139219723](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623139219723.png)

#### 有状态计算

![1623139250027](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623139250027.png)

#### Flink中的状态分类

- State
  - ManagerState-开发中推荐使用,Flink自动管理和优化，支持多种数据结构和类型
    - KeyState：只能在keyedStream上面使用，支持多种数据结构
    - operatorState：可以在所有数据流中使用，支持liststate结构
  - Raw State，完全由用户自己管理，只支持byte[]数组，只能在自定义operator中使用。
    - operatorState

![1623139879988](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/161125-490915.png)

开发中大多使用自动管理机制。

![1623140791468](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623140791468.png)

#### Flink中的状态

![1614671349533](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/03/101838-752218.png)

- **由一个任务维护，并且用来计算某个结果的所有数据，都属于这个任务的状态**
- 可以认为状态就是一个本地变量，可以被任务的业务逻辑访问
- Flink会进行状态管理，包括状态一致性、故障处理以及高效存储和访问，以便开发人员可以专注于应用程序的逻辑
- Flink中的状态是和任务绑定在一起的，可以认为是任务的一个**本地变量**。
- 在Flink中，状态始终与特定算子相关联
- 为了使运行时的Flink了解算子的状态，算子需要预先注册其状态

**有两种类型的状态：**

- 算子状态（Operator State）
  - 算子状态的作用范围限定为算子任务
- 键控状态（Keyed State）
  - 根据输入数据流中定义的键（key）来维护和访问，也就是说只有当前key对应的数据才可以访问当前的状态

#### 算子状态（Operator State）

![1614673703139](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/03/101844-455784.png)

- 算子状态的作用范围限定为算子任务，由同一并行任务所处理的所有数据都可以访问到相同的状态
- 状态对于同一子任务而言是共享的
- 算子状态不能由相同或不同算子的另一个子任务访问

##### 算子状态数据结构

- 列表状态（List state）
  - 将状态表示为一组数据的列表
- 联合列表状态（Union list state）
  - 也将状态表示为数据的列表。它与常规列表状态的区别在于，在发生故障时，或者从保存点（savepoint）启动应用程序时如何恢复
- 广播状态（Broadcast state）
  - 如果一个算子有多项任务，而它的每项任务状态又都相同，那么这种特殊情况最适合应用广播状态。

**案例**

~~~ java
public class StateTest {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

//        定义一个有状态的map操作，统计当前有多少个温度值
//        统计当前分区数据的个数
        SingleOutputStreamOperator<Integer> result = map.map(new MyMapMapper());

        result.print();


        env.execute();
    }

    public static class MyMapMapper implements MapFunction<SensorReading,Integer>, ListCheckpointed<Integer> {

//        定义一个本地变量，作为算子状态
        private Integer count=0;

        @Override
        public Integer map(SensorReading value) throws Exception {
            return count++;
        }

//        对状态做快照
        @Override
        public List<Integer> snapshotState(long checkpointId, long timestamp) throws Exception {
            return Collections.singletonList(count);
        }

//        恢复快照
        @Override
        public void restoreState(List<Integer> state) throws Exception {
            for (Integer num:state) {
//                恢复count的值
                count+=num;
            }
        }
    }
}

~~~

#### 键控状态（Keyed State）

![1614735541965](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/17/111456-631534.png)

1. 键控状态是根据输入数据流中定义的键（key）来维护和访问的，根据键分组后，每一组键都有对应的状态。
2. Flink为每个key维护一个状态实例，并将具有相同键的所有数据，都分区到同一个算子任务中，这个任务会维护和处理这个key对应的状态
3. 当任务处理一条数据时，它会自动将状态的访问范围限定为当前数据的key

##### 键控状态数据结构

- 值状态（Valuestate）
  - 将状态表示为单个的值
- 列表状态（List state）
  - 将状态表示为一组数据的列表
- 映射状态（Mapstate）
  - 将状态表示为一组Key-Value对
- 聚合状态（Reducing state & Aggregating State）
  - 将状态表示为一个用于聚合操作的列表

##### 键控状态的使用

声明一个键控状态

![1614736400065](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614736400065.png)

读取状态

![1614736439159](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614736439159.png)

对状态赋值

![1614736465644](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614736465644.png)

**案例**

~~~ java
public class KeyedState {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

        SingleOutputStreamOperator<Integer> result = map
                .keyBy("id")
                .map(new MyKeyCountMapper());
        result.print();


        env.execute();
    }

    public static class MyKeyCountMapper extends RichMapFunction<SensorReading,Integer>{

//        new ValueStateDescriptor<Integer>("keycount",Integer.class):参数表示状态的名字和状态的类型
//        在环境所需要的实例全部都创建出来后才可以拿到运行时上下文环境，必须在open()后才可以做初始化操作
        private ValueState<Integer>keyCountState;

//        其他类型的状态声明
        private ListState<String> listState;

//        map状态
        private MapState<String,Double> mapState;

        @Override
        public void open(Configuration parameters) throws Exception {
            keyCountState=getRuntimeContext().getState(new ValueStateDescriptor<Integer>("keycount",Integer.class,0));
//            这里没有对count进行初始化，报错
            //keyCountState=getRuntimeContext().getState(new ValueStateDescriptor<Integer>("keycount",Integer.class));

//            初始化列表状态
            listState=getRuntimeContext().getListState(new ListStateDescriptor<String>("list_state",String.class));

            mapState=getRuntimeContext().getMapState((new MapStateDescriptor<String, Double>("map-state",String.class,Double.class)));
        }

        @Override
        public Integer map(SensorReading value) throws Exception {

            //            存入一个状态
            mapState.put("1",32.5);
//            向map状态中获取一个状态
            Double aDouble = mapState.get("1");

            //            获取list状态的值
            Iterable<String> strings = listState.get();
            for (String str:strings) {
                System.out.println(str);

            }

//            所有的状态都有clear()方法，清空所有的状态
            mapState.clear();
//            向列表中添加一个新的状态
            listState.add("state-1");

//            对单个状态变量的值做操作
            Integer count = keyCountState.value();
            count++;
            keyCountState.update(count);
            return count;

        }
    }
}

~~~

##### 练习

~~~ java
public class ApplicationCase {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

//        定义flatmap检测温度的跳变，输出报警信息
//        注意，这里对数据进行了分组，相同组中的数据温度差值大于10才会报警，不同组之间相差10不会报警
        SingleOutputStreamOperator<Tuple3<String, Double, Double>> result = map
                .keyBy("id")
                .flatMap(new MychangeFlatMap(10.0));

        result.print();

        env.execute();
    }

    public static class MychangeFlatMap extends RichFlatMapFunction<SensorReading, Tuple3<String,Double,Double>>
    {
        private Double threshold;

//        定义一个状态，保存上一次的温度值
        private ValueState<Double> lastTemp;

        public MychangeFlatMap(Double threshold) {
            this.threshold = threshold;
        }

//        在open方法中对状态变量做初始化


        @Override
        public void open(Configuration parameters) throws Exception {
            lastTemp=getRuntimeContext().getState(new ValueStateDescriptor<Double>("temp",Double.class,1.0));

        }

        @Override
        public void flatMap(SensorReading value, Collector<Tuple3<String, Double, Double>> out) throws Exception {

//            获取状态，也就是上一次的温度值
            Double value1 = lastTemp.value();
            if(lastTemp!=null){
                Double diff=Math.abs(value.getTemperature()-value1);
                if(diff >= threshold){
                    out.collect(new Tuple3<String,Double,Double>(value.getId(),value1,value.getTemperature()));

                }

            }
                //            更新状态信息
            lastTemp.update(value.getTemperature());
        }

//        最后还要做清理工作

        @Override
        public void close() throws Exception {
            lastTemp.clear();
        }
    }
}
~~~

#### 状态后端（State Backends）

![1623389350664](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/132913-888517.png)

- 每传入一条数据，有状态的算子任务都会读取和更新状态
- 由于有效的状态访问对于处理数据的低延迟至关重要，因此每个并行任务都会在本地维护其状态，以确保快速的状态访问
- 状态的存储、访问以及维护，由一个可插入的组件决定，这个组件就叫做状态后端（state backend）
- 状态后端主要负责两件事：本地的状态管理（也就是内存中的状态管理），以及将检查点（checkpoint）状态写入远程存储。容错性的保证，备份状态。

##### 状态后端分类

**MemoryStateBackend**

![1623389433434](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/133035-909589.png)

- 内存级的状态后端，会将键控状态作为内存中的对象进行管理，将它们存储在TaskManager的JVM堆上，而将checkpoint存储在JobManager的内存中
- 特点：快速、低延迟，但不稳定

**FsStateBackend**

![1623389471575](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/133113-305891.png)

- 将checkpoint存到远程的持久化文件系统（FileSystem）上，而对于本地状态，跟MemoryStateBackend一样，也会存在TaskManager的JVM堆上
- 同时拥有内存级的本地访问速度，和更好的容错保证

**RocksDBStateBackend**

![1623391739132](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/140902-198202.png)

将所有状态序列化后，存入本地的RocksDB中存储。

##### 状态后端配置

~~~ java
public class StateBackend {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        状态后端的配置
//        内存状态后端
        env.setStateBackend(new MemoryStateBackend());
//        文件系统状态后端
        env.setStateBackend(new FsStateBackend("path"));
//        RocksDBStateBackend需要引入依赖
        env.setStateBackend(new RocksDBStateBackend("path"));

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

        map.print();

        env.execute();
    }

}

~~~

**引入依赖**

~~~ java
 <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-statebackend-rocksdb_2.12</artifactId>scala版本号
            <version>1.10.1</version>flink版本号
        </dependency>
~~~

#### 案例说明

![1623143547852](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623143547852.png)

**代码演示**

~~~ java
public class Test22 {

    public static void main(String[] args) throws Exception {
        /**
         * 基于事件的滚动和滑动窗口
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        DataStreamSource<Tuple2<String, Integer>> source = env.fromElements(
                Tuple2.of("beijing", 2),
                Tuple2.of("shanghai", 5),
                Tuple2.of("shenzhen", 8),
                Tuple2.of("guangzhou", 1),
                Tuple2.of("xian", 19),
                Tuple2.of("dalian", 9),
                Tuple2.of("tianjin", 18)
        );

//        求value的最大值
//        source.keyBy(t -> t.f0)
//                .maxBy(1)
//                .print();

        SingleOutputStreamOperator<Tuple3<String, Integer, Integer>> map = source.map(new RichMapFunction<Tuple2<String, Integer>, Tuple3<String, Integer, Integer>>() {
//            定义一个状态，存放最大值
            private ValueState<Integer> max;
//            对状态进行初始化工作,open()方法只会调用一次
            @Override
            public void open(Configuration parameters) throws Exception {
                ValueStateDescriptor<Integer> integerValueStateDescriptor = new ValueStateDescriptor<Integer>("max",Integer.class);
                max=getRuntimeContext().getState(integerValueStateDescriptor);
            }

//            使用状态
            @Override
            public Tuple3<String, Integer, Integer> map(Tuple2<String, Integer> value) throws Exception {

//                获取当前的值
                Integer currentValue = value.f1;
//                获取状态
                Integer state = max.value();

                if(state == null || currentValue > state){
                    max.update(currentValue);
                    return Tuple3.of(value.f0,currentValue,currentValue);
                }
                return Tuple3.of(value.f0,currentValue,currentValue);
            }
        });

        map.print();


        env.execute();
    }
}
~~~

**案例二**

![1623388204192](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/131025-894336.png)

~~~ java
public class Test23 {

    public static void main(String[] args) throws Exception {
        /**
         * 基于事件的滚动和滑动窗口
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
//        设置并行度为1，方便观察
        env.setParallelism(1);
        DataStreamSource dataStreamSource = env.addSource(new MyKakfa());

        dataStreamSource.print();


        env.execute();
    }

     static class MyKakfa extends RichParallelSourceFunction implements CheckpointedFunction {

//        因为并行度设置为1，所以只有一个分区
        private ListState<Long> offsetState=null;
//        如果有多个分区，那么List状态里面存储的是分区对应的状态
//        private ListState<Map<partition,Offset>> offsetState=null;
        private Long offset=0l;
        private boolean flag=true;


//          持久化state
//        该方法会定时执行将state存放到checkPoint（磁盘目录）中
        @Override
        public void snapshotState(FunctionSnapshotContext context) throws Exception {
//            清理内存中的数据，并且存放到checkPoint中
            offsetState.clear();
            offsetState.add(offset);

        }
        //        初始化
        @Override
        public void initializeState(FunctionInitializationContext context) throws Exception {

            ListStateDescriptor<Long> descriptor = new ListStateDescriptor<>("offsetState", Long.class);
            ListState<Long> listState = context.getOperatorStateStore().getListState(descriptor);
            offsetState=listState;
        }

        @Override
        public void run(SourceContext ctx) throws Exception {

           while (flag){
               Iterator<Long> iterator = offsetState.get().iterator();
               if(iterator.hasNext()){
                   offset=iterator.next();
               }
               offset+=1;
               int subtaskId=getRuntimeContext().getIndexOfThisSubtask();
               ctx.collect("subtaskId="+subtaskId+"   offset="+offset);
               Thread.sleep(2000);
           }
        }

        @Override
        public void cancel() {
            flag=false;
        }
    }
}
~~~

## 容错机制

state状态一般存储在内存中，而checkpoint一般存储在磁盘中。

CheckPoint和State的关系

![1623388709313](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/131831-48171.png)

### 状态一致性

当在分布式系统中引入状态时，自然也引入了一致性问题。一致性实际上是"正确性级别"的另一种说法， 也就是说在成功处理故障并恢复之后得到的结果，与没有发生任何故障时得到的结果相比，前者到底有多正确？举例来说，假设要对最近一小时登录的用户计数。在系统经历故障之后，计数结果是多少？ 如果有偏差，是有漏掉的计数还是重复计数？ 

#### 什么是状态一致性

- 有状态的流处理，内部每个算子任务都可以有自己的状态
- 对于流处理器内部来说，所谓的状态一致性，其实就是我们所说的计算结果要保证准确。
- 一条数据不应该丢失，也不应该重复计算
- 在遇到故障时可以恢复状态，恢复以后的重新计算，结果应该也是完全正确的。

#### 一致性级别

在流处理中，一致性可以分为 3 个级别：

- at-most-once: 这其实是没有正确性保障的委婉说法——故障发生之后，计数结果可能丢失。同样的还有 udp。当任务故障时，**最简单的做法是什么都不干，既不恢复丢失的状态，也不重播丢失的数据**。At-most-once语义的含义是最多处理一次事件。保证处理速度很快，性能达到最优，但是缺点就是会发生丢失数据。
- at-least-once: 这表示计数结果可能大于正确值，但绝不会小于正确值。也就是说，计数程序在发生故障后可能多算，但是绝不会少算。在大多数的真实应用场景，我们希望不丢失事件。这种类型的保障称为at-least-once，意思是所有的事件都得到了处理，而一些事件还可能被处理多次。
- exactly-once: 这指的是系统保证在发生故障后得到的计数结果与正确值一致。恰好处理一次是最严格的保证，也是最难实现的。恰好处理一次语义不仅仅意味着没有事件丢失，还意味着针对每一个数据，内部状态仅仅更新一次。 

**一致性检查点（Checkpoints）**

- Flink使用了一种轻量级快照机制——检查点（checkpoint）来保证exactly-once语义，这个快照选取的是刚好所有的数据都处理完成后的那个时间点保存快照，另外在source端还要保存检查点数据的偏移量信息。
- 有状态流应用的一致检查点，其实就是：所有任务的状态，在某个时间点的一份拷贝（一份快照）。而这个时间点，应该是所有任务都恰好处理完一个相同的输入数据的时候。
- 应用状态的一致检查点，是Flink故障恢复机制的核心

![1616373943517](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616373943517.png)

曾经， at-least-once 非常流行。第一代流处理器(如 Storm 和 Samza)刚问世时只保证 at-least-once，原因有二。 

- 保证 exactly-once 的系统实现起来更复杂。这在基础架构层(决定什么代表正确，以及 exactly-once 的范围是什么)和实现层都很有挑战性。
- 流处理系统的早期用户愿意接受框架的局限性，并在应用层想办法弥补(例如使应用程序具有幂等性，或者用批量计算层再做一遍计算)。 

最先保证 exactly-once 的系统(Storm Trident 和 Spark Streaming)在性能和表现力这两个方面付出了很大的代价。为了保证 exactly-once，这些系统无法单独地对每条记录运用应用逻辑，而是同时处理多条(一批)记录，保证对每一批的处理要么全部成功，要么全部失败。这就导致在得到结果前，必须等待一批记录处理结束。因此，用户经常不得不使用两个流处理框架(一个用来保证 exactly-once，另一个用来对每个元素做低延迟处理)，结果使基础设施更加复杂。曾经，用户不得不在保证exactly-once 与获得低延迟和效率之间权衡利弊。 Flink 避免了这种权衡。 

Flink 的一个重大价值在于， 它既保证了 exactly-once，也具有低延迟和高吞吐的处理能力 

从根本上说， Flink 通过使自身满足所有需求来避免权衡，它是业界的一次意义重大的技术飞跃。尽管这在外行看来很神奇，但是一旦了解，就会恍然大悟。 

#### 端到端（end-to-end）状态一致性 

目前我们看到的一致性保证都是由流处理器实现的，也就是说都是在 Flink 流处理器内部保证的；而在真实应用中，流处理应用除了流处理器以外还包含了数据源（例如 Kafka）和输出到持久化系统。 

端到端的一致性保证，意味着结果的正确性贯穿了整个流处理应用的始终；每一个组件都保证了它自己的一致性， 整个端到端的一致性级别取决于所有组件中一致性最弱的组件。具体可以划分如下： 

- 内部保证 —— 依赖 checkpoint
- source 端 —— 需要外部源可重设数据的读取位置
- sink 端 —— 需要保证从故障恢复时，数据不会重复写入外部系统 
  - 而对于 sink 端，又有两种具体的实现方式：幂等（ Idempotent）写入和事务性（ Transactional）写入。 

**幂等写入**
所谓幂等操作，是说一个操作，可以重复执行很多次，但只导致一次结果更改，也就是说，后面再重复执行就不起作用了。 幂等写入可能存在数据的重复写入，因为是进行追加操作，会重复写入检查点之后的数据。有短暂的数据不一致，这里的写入指的是写入外部系统。

可以想象为集合中的hashmap

![1616374043747](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/22/084726-924873.png)

**事务写入**

事务（Transaction）

- 应用程序中一系列严密的操作，所有操作必须成功完成，否则在每个操作中所作的所有更改都会被撤消
- 具有原子性：一个事务中的一系列的操作要么全部成功，要么一个都不做

需要构建事务来写入外部系统，构建的事务对应着 checkpoint，等到 checkpoint真正完成的时候，才把所有对应的结果写入 sink 系统中。 （和外部系统构建一个事务，提交的时候是基于事务的提交）

对于事务性写入，具体又有两种实现方式：预写日志（ WAL）和两阶段提交（ 2PC）。 DataStream API 提供了 GenericWriteAheadSink 模板类和TwoPhaseCommitSinkFunction 接口，可以方便地实现这两种方式的事务性写入。 

- 预写日志（Write-Ahead-Log，WAL）
  - 把结果数据先当成状态保存，然后在收到checkpoint完成的通知时，一次性写入sink系统
  - 简单易于实现，由于数据提前在状态后端中做了缓存，所以无论什么sink系统，都能用这种方式一批搞定，这样写入的时候就是批处理，会有延迟时间。但是也有问题，如果写入数据的时候，只能写入一半的话，那么就会发生故障，所以不能严格意义上说是流处理。
  - DataStreamAPI提供了一个模板类：GenericWriteAheadSink，来实现这种事务性sink
- 两阶段提交（Two-Phase-Commit，2PC）
  - 对于每个checkpoint，sink任务会启动一个事务，并将接下来所有接收的数据添加到事务里
  - 然后将这些数据写入外部sink系统，但不提交它们——这时只是“预提交”
  - 当它收到checkpoint完成的通知时，它才正式提交事务，实现结果的真正写入
  - 这种方式真正实现了exactly-once，它需要一个提供事务支持的外部sink系统。Flink提供了TwoPhaseCommitSinkFunction接口。
  - 简单来说实现精确一次端到端的传输是用：checkpoint和事务的提交结合
- 2PC对外部sink系统的要求
  - 外部sink系统必须提供事务支持，或者sink任务必须能够模拟外部系统上的事务
  - 在checkpoint的间隔期间里，必须能够开启一个事务并接受数据写入
  - 在收到checkpoint完成的通知之前，事务必须是“等待提交”的状态。
  - 在故障恢复的情况下，这可能需要一些时间。如果这个时候sink系统关闭事务（例如超时了），那么未提交的数据就会丢失
  - sink任务必须能够在进程失败后恢复事务
  - 提交事务必须是幂等操作

不同 Source 和 Sink 的一致性保证可以用下表说明： 

如果source端不可以重置数据的偏移量，那么后面就会丢失数据，

如果source端可以重置数据的偏移量，那么至少可以保证at-least-once

![1616301435004](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/22/081650-483061.png)

#### Flink+Kafka端到端状态一致性的保证

- Flink内部——利用checkpoint机制，把状态存盘，发生故障的时候可以恢复，保证内部的状态一致性。
- source——kafka consumer作为source，可以将偏移量保存下来，如果后续任务出现了故障，恢复的时候可以由连接器重置偏移量，重新消费数据，保证一致性
- sink——kafka producer作为sink，采用两阶段提交sink，需要实现一个TwoPhaseCommitSinkFunction

**Exactly-once两阶段提交**

![1616376244725](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/14/201912-125712.png)

- JobManager协调各个TaskManager进行checkpoint存储
- checkpoint保存在StateBackend中，默认StateBackend是内存级的，也可以改为文件级的进行持久化保存

![1616376310417](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616376310417.png)

- 当checkpoint启动时，JobManager会将检查点分界线（barrier）注入数据流
- 当每一个算子接收到数据流中的barrier的时候，会进行checkpoint处理，保存当前算子的数据状态。
- barrier会在算子间传递下去

![1616376350174](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616376350174.png)

- 每个算子会对当前的状态做个快照，保存到状态后端
- checkpoint机制可以保证内部的状态一致性

![1616376396040](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616376396040.png)

- 每个内部的transform任务遇到barrier时，都会把状态存到checkpoint里
- 什么时候开启一个新的事务？当sink端接受到barrier数据的时候，第一件做的事情是sink执行checkpoint操作，就是保存自己的数据，但是此时整个checkpoint并没有完成，可能还有其他的算子checkpoint没有执行完毕，而sink这个时候会开启一个新的事务，因为barrier是一个分界线，以barrier为界限，之前的数据和之后的数据分别属于不同的checkpoint，在做两阶段提交的时候，要把事务和checkpoint绑定在一起，所以barrier之后带来的数据状态的改变，不应该在前一个checkpoint里面，后面数据状态的改变，应该在下一个checkpoint和事务中。当所有的算子的checkpoint全部完成的时候，向jobmanager报告checkpoint完成，此时jobmanager通知sink正式提交前一个事务，写入数据。
- sink任务首先把数据写入外部kafka，这些数据都属于预提交的事务；遇到barrier时，把状态保存到状态后端，并开启新的预提交事务

![1616376443799](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616376443799.png)

- 当所有算子任务的快照完成，也就是这次的checkpoint完成时，JobManager会向所有任务发通知，确认这次checkpoint完成
- sink任务收到确认通知，正式提交之前的事务，kafka中未确认数据改为“已确认”

**Exactly-once两阶段提交步骤**

- 第一条数据来了之后，开启一个kafka的事务（transaction），正常写入kafka分区日志但标记为未提交，这就是“预提交”
- jobmanager触发checkpoint操作，barrier从source开始向下传递，遇到barrier的算子将状态存入状态后端，并通知jobmanager
- sink连接器收到barrier，保存当前状态，存入checkpoint，通知jobmanager，并开启下一阶段的事务，用于提交下个检查点的数据
- jobmanager收到所有任务的通知，发出确认信息，表示checkpoint完成
- sink任务收到jobmanager的确认信息，正式提交这段时间的数据
- 外部kafka关闭事务，提交的数据可以正常消费了。

#### 状态的恢复和重启策略

![1623392860474](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/142743-100875.png)

##### 默认的重启策略：

当程序出现异常的时候，不会抛出异常，而是执行重启程序。默认重启策略可以无限重启。

- 在配置了checkpoint的情况下，不做任何配置，默认是无限重启自动恢复。可以解决小异常，但是可能隐藏真正的错误。
- 如果配置了Checkpoint,而没有配置重启策略,那么代码中出现了非致命错误时,程序会无限重启

##### 单独配置无重启策略

~~~ java
//Job直接失败，不会尝试进行重启
 //设置方式1:
 restart-strategy: none
 
 //设置方式2:
 //无重启策略也可以在程序中设置
 val env = ExecutionEnvironment.getExecutionEnvironment()
 env.setRestartStrategy(RestartStrategies.noRestart())
~~~

#### 固定延迟重启策略--开发中使用

~~~ java
设置方式1:
 重启策略可以配置flink-conf.yaml的下面配置参数来启用，作为默认的重启策略:
 例子:
 restart-strategy: fixed-delay
 restart-strategy.fixed-delay.attempts: 3
 restart-strategy.fixed-delay.delay: 10 s
 
 设置方式2:
 也可以在程序中设置:
 val env = ExecutionEnvironment.getExecutionEnvironment()
 env.setRestartStrategy(RestartStrategies.fixedDelayRestart(
   3, // 最多重启3次数
   Time.of(10, TimeUnit.SECONDS) // 重启时间间隔
 ))
 上面的设置表示:如果job失败,重启3次, 每次间隔10

~~~

#### 失败率重启策略--开发偶尔使用

~~~ java
设置方式1:
 失败率重启策略可以在flink-conf.yaml中设置下面的配置参数来启用:
 例子:
 restart-strategy:failure-rate
 restart-strategy.failure-rate.max-failures-per-interval: 3
 restart-strategy.failure-rate.failure-rate-interval: 5 min
 restart-strategy.failure-rate.delay: 10 s
 
 设置方式2:
 失败率重启策略也可以在程序中设置:
 val env = ExecutionEnvironment.getExecutionEnvironment()
 env.setRestartStrategy(RestartStrategies.failureRateRestart(
   3, // 每个测量时间间隔最大失败次数
   Time.of(5, TimeUnit.MINUTES), //失败率测量的时间间隔
   Time.of(10, TimeUnit.SECONDS) // 两次连续重启的时间间隔
 ))
//上面的设置表示:如果5分钟内job失败不超过三次,自动重启, 每次间隔10s (如果5分钟内程序失败超过3次,则程序退出)
~~~

### 检查点（ checkpoint）

Flink 具体如何保证 exactly-once 呢? 它使用一种被称为"检查点"（ checkpoint）的特性，在出现故障时将系统重置回正确状态。下面通过简单的类比来解释检查点的作用。 

假设你和两位朋友正在数项链上有多少颗珠子，如下图所示。你捏住珠子，边数边拨，每拨过一颗珠子就给总数加一。你的朋友也这样数他们手中的珠子。当你分神忘记数到哪里时，怎么办呢? 如果项链上有很多珠子，你显然不想从头再数一遍，尤其是当三人的速度不一样却又试图合作的时候，更是如此(比如想记录前一分钟三人一共数了多少颗珠子，回想一下一分钟滚动窗口)。 

![1616301615060](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616301615060.png)

于是，你想了一个更好的办法: 在项链上每隔一段就松松地系上一根有色皮筋，将珠子分隔开; 当珠子被拨动的时候，皮筋也可以被拨动; 然后，你安排一个助手，让他在你和朋友拨到皮筋时记录总数。用这种方法，当有人数错时，就不必从头开始数。相反，你向其他人发出错误警示，然后你们都从上一根皮筋处开始重数，助
手则会告诉每个人重数时的起始数值，例如在粉色皮筋处的数值是多少。 

Flink 检查点的作用就类似于皮筋标记。数珠子这个类比的关键点是: 对于指定的皮筋而言，珠子的相对位置是确定的; 这让皮筋成为重新计数的参考点。总状态(珠子的总数)在每颗珠子被拨动之后更新一次，助手则会保存与每根皮筋对应的检查点状态，如当遇到粉色皮筋时一共数了多少珠子，当遇到橙色皮筋时又是多少。当问题出现时，这种方法使得重新计数变得简单。 

#### Flink 的检查点算法 

~~~ java
val stream: DataStream[(String, Int)] = ...
val counts: DataStream[(String, Int)] = stream
.keyBy(record => record._1)
.mapWithState( (in: (String, Int), state: Option[Int]) =>
              state match {
case Some(c) => ( (in._1, c + in._2), Some(c + in._2) )
case None => ( (in._1, in._2), Some(in._2) )
})
~~~

该程序有两个算子: keyBy 算子用来将记录按照第一个元素(一个字符串)进行分组，根据该 key 将数据进行重新分区，然后将记录再发送给下一个算子: 有状态的map 算子(mapWithState)。 map 算子在接收到每个元素后，将输入记录的第二个字段的数据加到现有总数中，再将更新过的元素发射出去。下图表示程序的初始状态: 输入流中的 6 条记录被检查点分割线(checkpoint barrier)隔开，所有的 map 算子状态均为 0(计数还未开始)。所有 key 为 a 的记录将被顶层的 map 算子处理，所有 key 为 b的记录将被中间层的 map 算子处理，所有 key 为 c 的记录则将被底层的 map 算子处理。 

![1616302124351](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616302124351.png)

上图是程序的初始状态。注意， a、 b、 c 三组的初始计数状态都是 0，即三个圆柱上的值。 ckpt 表示检查点分割线（ checkpoint barriers） 。每条记录在处理顺序上严格地遵守在检查点之前或之后的规定，例如["b",2]在检查点之前被处理， ["a",2]则在检查点之后被处理。 

当该程序处理输入流中的 6 条记录时，涉及的操作遍布 3 个并行实例(节点、 CPU内核等)。那么，检查点该如何保证 exactly-once 呢? 

检查点分割线和普通数据记录类似。它们由算子处理，但并不参与计算，而是会触发与检查点相关的行为。当读取输入流的数据源(在本例中与 keyBy 算子内联)遇到检查点屏障时，它将其在输入流中的位置保存到持久化存储中。如果输入流来自消息传输系统(Kafka)，这个位置就是偏移量。 Flink 的存储机制是插件化的，持久
化存储可以是分布式文件系统，如 HDFS。下图展示了这个过程 

![1616302376919](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/21/125258-782428.png)

当 Flink 数据源(在本例中与 keyBy 算子内联)遇到检查点分界线（ barrier） 时，它会将其在输入流中的位置保存到持久化存储中。这让 Flink 可以根据该位置重启。 

检查点像普通数据记录一样在算子之间流动。当 map 算子处理完前 3 条数据并收到检查点分界线时，它们会将状态以异步的方式写入持久化存储，如下图所示。 

![1616302487062](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616302487062.png)

位于检查点之前的所有记录(["b",2]、 ["b",3]和["c",1])被 map 算子处理之后的情况。此时，持久化存储已经备份了检查点分界线在输入流中的位置(备份操作发生在barrier 被输入算子处理的时候)。 map 算子接着开始处理检查点分界线，并触发将状态异步备份到稳定存储中这个动作。 

当 map 算子的状态备份和检查点分界线的位置备份被确认之后，该检查点操作就可以被标记为完成，如下图所示。我们在无须停止或者阻断计算的条件下，在一个逻辑时间点(对应检查点屏障在输入流中的位置)为计算状态拍了快照。通过确保备份的状态和位置指向同一个逻辑时间点，后文将解释如何基于备份恢复计算，从而保证 exactly-once。值得注意的是，当没有出现故障时， Flink 检查点的开销极小，检查点操作的速度由持久化存储的可用带宽决定。回顾数珠子的例子: 除了因为数错而需要用到皮筋之外，皮筋会被很快地拨过。 

![1616302642963](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616302642963.png)

检查点操作完成，状态和位置均已备份到稳定存储中。输入流中的所有数据记录都已处理完成。值得注意的是，备份的状态值与实际的状态值是不同的。备份反映的是检查点的状态。 

如果检查点操作失败， Flink 可以丢弃该检查点并继续正常执行，因为之后的某一个检查点可能会成功。虽然恢复时间可能更长，但是对于状态的保证依旧很有力。只有在一系列连续的检查点操作失败之后， Flink 才会抛出错误，因为这通常预示着发生了严重且持久的错误 

现在来看看下图所示的情况: 检查点操作已经完成，但故障紧随其后。 

![1616302748623](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616302748623.png)

在这种情况下， Flink 会重新拓扑(可能会获取新的执行资源)，将输入流倒回到上一个检查点，然后恢复状态值并从该处开始继续计算。在本例中， ["a",2]、 ["a",2]和["c",2]这几条记录将被重播。

下图展示了这一重新处理过程。从上一个检查点开始重新计算，可以保证在剩下的记录被处理之后，得到的 map 算子的状态值与没有发生故障时的状态值一致。 

 ![1616302900084](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/21/130141-647076.png)

Flink 将输入流倒回到上一个检查点屏障的位置，同时恢复 map 算子的状态值。然后， Flink 从此处开始重新处理。这样做保证了在记录被处理之后， map 算子的状态值与没有发生故障时的一致。 

Flink 检查点算法的正式名称是异步分界线快照(asynchronous barrier snapshotting)。该算法大致基于 Chandy-Lamport 分布式快照算法。
检查点是 Flink 最有价值的创新之一，因为它使 Flink 可以保证 exactly-once，并且不需要牺牲性能。 

#### 一致性检查点

![1616309570252](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/131309-394748.png)

- Flink故障恢复机制的核心，就是应用状态的一致性检查点
- 有状态流应用的一致检查点，其实就是所有任务的状态，在某个时间点的一份拷贝（一份快照）；这个时间点，应该是所有任务都恰好处理完一个相同的输入数据的时候

#### 从检查点中恢复

- 在执行流应用程序期间，Flink会定期保存状态的一致检查点
- 如果发生故障，Flink将会使用最近的检查点来一致恢复应用程序的状态，并重新启动处理流程

![1616309793406](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/21/145636-187637.png)

遇到故障之后，第一步就是重启应用

![1616309846884](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616309846884.png)

第二步是从checkpoint中读取状态，将状态重置
从检查点重新启动应用程序后，其内部状态与检查点完成时的状态完全相同

![1616309885786](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616309885786.png)

第三步：开始消费并处理检查点到发生故障之间的所有数据
这种检查点的保存和恢复机制可以为应用程序状态提供“精确一次”（exactly-once）的一致性，因为所有算子都会保存检查点并恢复其所有状态，这样一来所有的输入流就都会被重置到检查点完成时的位置

#### 检查点算法的实现

- 一种简单的想法

  - ——暂停应用，保存状态到检查点，再重新恢复应用
    •Flink的改进实现

- 基于Chandy-Lamport算法的分布式快照
  - 将检查点的保存和数据处理分离开，不暂停整个应用

#### Flink检查点算法

**Checkpoint过程**

![1623388870170](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/132216-647596.png)

![1623388940699](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/132223-943696.png)

![1623389107026](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/132509-633562.png)

在进行写快照的过程中是异步执行的。简单来说checkpoint就是按照固定的时间间隔把state中的状态进行持久化处理。

**原理**

Flink的检查点算怯中会用到一类名为检查点分隔符 (checkpoint barrier) 的特殊记录。和水位线类似，这些检查点分隔符会通过数据源算子注入到常规的记录流中。相对其他记录，它们在流中的位置无怯提前或延后。为了标识所属的检查点，每个检查点分隔符都会带有一个检查点编号，这样就把一条数据流从逻辑上分成了两个部分。所有先于分隔符的记录所引起的状态更改都会被包含在分隔符所对应的检查点之中;而所有晚于分隔符的记录所引起的状态更改者H会被纳入之后的检查点中。 

barrier代表的数据就是把一条数据流分隔开。检查点存储的是数据的偏移量

![1616310125285](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/21/150224-406403.png)

现在是一个有两个输入流的应用程序，用并行的两个Source任务来读取

JobManager会向每个source任务发送一条带有新检查点ID的消息，通过这种方式来启动检查点

![1616310546745](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/21/150932-895539.png)

当 一个数据源任务收到消息后，会暂停发出记录 ， 利用状态后端触发生成本 地状态的检查点，并把该检查点分隔符连同检查点编号广播至所有传出的数据流分区。状态后端会在状态存为检查点完成后通知任务，随后任务会给jobManager发送确认消息，在将所有分隔符发出后，数据源将恢复正常工作。 通过向输出流中注入分隔符，数据源函数定义了需要在流中哪些位置生成检 查点 。 下图展示了流式应用为数据源任务的本地状态生成检查点并且发出检查分隔符。 

![1616310510607](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/21/150901-244841.png)

每一个算子会等到他的上游所有发送的barrier都到齐后然后在执行check point工作。

数据源任务发出的检查点分隔符会传输到与之相连的任务 。 和水位线类似，检查点分隔符总是以广播形式发送 ，从而可以确保每个任务能从它们 的每个 输入都收到一个分隔符。当任务收到一个新检查点的分隔符时，会继续等待所有其他输入分区也发来这个检查点的分隔符。在等待过程中，它会继续处理那些从还未提供分隔符的分区发来的数据。对于已经提供分隔符的分区，它们新到来的记录会被缓冲起来，不能处理。这个等待所有分隔符到达的过程称为分隔符对齐，

![1616311171773](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/21/151946-831349.png)

- 分界线对齐：barrier向下游传递，sum任务会等待所有输入分区的barrier到达
- 对于barrier已经到达的分区，继续到达的数据会被缓存
- 而barrier尚未到达的分区，数据会被正常处理 

![1616311234484](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616311234484.png)

当收到所有输入分区的barrier时，任务就将其状态保存到状态后端的检查点中，然后将barrier继续向下游转发

![1616311309579](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616311309579.png)

向下游转发检查点barrier后，任务继续正常的数据处理

![1616311344797](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616311344797.png)

Sink任务向JobManager确认状态保存到checkpoint完毕

当所有任务都确认已成功将状态保存到检查点时，检查点就真正完成了

#### 保存点（Savepoints）

savepoint其实就是手动的checkpoint 。

![1623394260702](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/145146-795710.png)

- Flink还提供了可以自定义的镜像保存功能，就是保存点（savepoints）
- 原则上，创建保存点使用的算法与检查点完全相同，因此保存点可以认为就是具有一些额外元数据的检查点
- Flink不会自动创建保存点，因此用户（或者外部调度程序）必须明确地触发创建操作
- 保存点是一个强大的功能。除了故障恢复外，保存点可以用于：有计划的手动备份，更新应用程序，版本迁移，暂停和重启应用，等等

#### savepoint和checkpoint

![1623394409089](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623394409089.png)

## ProcessFunction API（ 底层 API） 

我们之前学习的转换算子是无法访问事件的时间戳信息和水位线信息的。而这在一些应用场景下，极为重要。例如 MapFunction 这样的 map 转换算子就无法访问时间戳或者当前事件的事件时间。 

基于此， DataStream API 提供了一系列的 Low-Level 转换算子。可以访问时间戳、 watermark 以及注册定时事件。还可以输出特定的一些事件，例如超时事件等。Process Function 用来构建事件驱动的应用以及实现自定义的业务逻辑(使用之前的window 函数和转换算子无法实现)。例如， Flink SQL 就是使用 Process Function 实现的。 

**Flink 提供了 8 个 Process Function： **

- ProcessFunction
- KeyedProcessFunction
- CoProcessFunction
- ProcessJoinFunction
- BroadcastProcessFunction
- KeyedBroadcastProcessFunction
- ProcessWindowFunction
- ProcessAllWindowFunction 

**继承关系**

![1615952074611](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/17/113437-161134.png)

### KeyedProcessFunction 

这里我们重点介绍 KeyedProcessFunction。
KeyedProcessFunction 用来操作 KeyedStream。 KeyedProcessFunction 会处理流的每一个元素，输出为 0 个、 1 个或者多个元素。所有的 Process Function 都继承自RichFunction 接口，所以都有 open()、close()和 getRuntimeContext()等方法。而KeyedProcessFunction<K, I, O>还额外提供了两个方法: 

- processElement(I value, Context ctx, Collector<O> out), 流中的每一个元素都会调用这个方法，调用结果将会放在 Collector 数据类型中输出。 Context 可以访问元素的时间戳，元素的 key，以及 TimerService 时间服务。 Context 还可以将结果输出到别的流(side outputs)。 
- onTimer(long timestamp, OnTimerContext ctx, Collector<O> out) 是一个回调函数。当之前注册的定时器触发时调用。参数 timestamp 为定时器所设定的触发的时间戳。 Collector 为输出结果的集合。 OnTimerContext 和processElement 的 Context 参数一样，提供了上下文的一些信息，例如定时器
  触发的时间信息(事件时间或者处理时间)。 

**案例**

~~~ java
public class ProcessFunTest {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

        //        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

//        先分组。然后自定义处理
//        sdss.keyBy("id").process(new MyProcess()).print();


        env.execute();
    }

    /**
     * 实现自定义的处理类
     */
    public static class MyProcess extends KeyedProcessFunction<Tuple,SensorReading,Integer>{

        ValueState<Long> tsTimer;

        @Override
        public void open(Configuration parameters) throws Exception {
            tsTimer=getRuntimeContext().getState(new ValueStateDescriptor<Long>("ts-timer",Long.class));
        }

      //处理每一个元素
        @Override
        public void processElement(SensorReading value, Context ctx, Collector<Integer> out) throws Exception {

//            输出id的长度
            out.collect(value.getId().length());

//            从context获取时间戳
            ctx.timestamp();
//            获取当前数据的键
            ctx.getCurrentKey();
//            测输出流的输出
//            ctx.output();
//          获取定时服务,注册处理时间定时器
//            ctx.timerService().registerProcessingTimeTimer();
//            事件时间定时器,根据当前的时间戳注册
            ctx.timerService().registerEventTimeTimer(value.getTempStamp()+10*1000);
//            更新时间,也就是保存时间戳
            tsTimer.update(value.getTempStamp()+10*1000);
//            删除时间定时器

//            ctx.timerService().deleteEventTimeTimer(1000);
//            如果现在想清空事件，可以直接使用时间戳
            ctx.timerService().deleteEventTimeTimer(tsTimer.value());
        }

        @Override
        public void onTimer(long timestamp, OnTimerContext ctx, Collector<Integer> out) throws Exception {

//            在这里写定时器时间到的时候做的触发操作
            System.out.println(timestamp+"定时器时间到");

        }

        @Override
        public void close() throws Exception {
            tsTimer.clear();
        }
    }
}
~~~

### TimerService 和 定时器（ Timers） 

Context 和 OnTimerContext 所持有的 TimerService 对象拥有以下方法: 

- long currentProcessingTime()：返回当前处理时间 
- long currentWatermark()：返回当前 watermark 的时间戳 
- void registerProcessingTimeTimer(long timestamp) 会注册当前 key 的processing time 的定时器。当 processing time 到达定时时间时，触发 timer。 
- void registerEventTimeTimer(long timestamp) 会注册当前 key 的 event time 定时器。当水位线大于等于定时器注册的时间时，触发定时器执行回调函数。 
- void deleteProcessingTimeTimer(long timestamp) 删除之前注册处理时间定时器。如果没有这个时间戳的定时器，则不执行。 
- void deleteEventTimeTimer(long timestamp) 删除之前注册的事件时间定时器，如果没有此时间戳的定时器，则不执行。

当定时器 timer 触发时， 会执行回调函数 onTimer()。 注意定时器 timer 只能在keyed streams 上面使用。 

下面举个例子说明 KeyedProcessFunction 如何操作 KeyedStream。

- 需求：监控温度传感器的温度值，如果温度值在 10 秒钟之内(processing time)连续上升， 则报警。 

~~~ java
public class ProcessFunTestCase {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

        //        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

        map.keyBy("id")
                .process(new TempIncrease(10l))
                .print();



        env.execute();
    }

    /**
     * 实现自定义处理函数，检测一段时间内温度连续上升，输出报警
     */
    public static class TempIncrease extends KeyedProcessFunction<String,SensorReading,String>{

//        定义时间间隔，当前统计时间的间隔
        private Integer interval;

        public TempIncrease(Integer interval) {
            this.interval = interval;
        }

//        定义状态，保存上一次的温度值，定义定时器时间戳
        private ValueState<Double> lastTempState;
        private ValueState<Long> timertsState;

        @Override
        public void open(Configuration parameters) throws Exception {
            lastTempState=getRuntimeContext().getState(new ValueStateDescriptor<Double>("last-time-state",Double.class,Double.MIN_VALUE));
            timertsState=getRuntimeContext().getState(new ValueStateDescriptor<Long>("timer-ts",Long.class));

        }

        @Override
        public void processElement(SensorReading value, Context ctx, Collector<String> out) throws Exception {
//            获取状态
            Double lastTemp = lastTempState.value();
//            获取时间戳
            Long timer = timertsState.value();
            lastTempState.update(value.getTemperature());

//            如果温度上升并且没有定时器，就注册10秒后的定时器开始等待
            if(value.getTemperature()>lastTemp && timer == null){
//                计算定时器的时间戳
                long ts=ctx.timerService().currentProcessingTime()+interval*1000l;
//                        注册处理时间的定时器
                ctx.timerService().registerProcessingTimeTimer(ts);
//                更新状态
                timertsState.update(ts);
            }
//            如果温度下降，就删除定时器
            else if(value.getTemperature() < lastTemp && timer != null){
                ctx.timerService().deleteProcessingTimeTimer(timer);
                timertsState.clear();
            }

//            更新温度的状态
            lastTempState.update(value.getTemperature());
        }

//        真正定时器的触发，调用的是ontime方法

        @Override
        public void onTimer(long timestamp, OnTimerContext ctx, Collector<String> out) throws Exception {
//            定时器触发，输出报警信息
            out.collect("传感器"+ctx.getCurrentKey()+"发出报警信息"+interval+"秒上升");
            timertsState.clear();
        }

        @Override
        public void close() throws Exception {
            lastTempState.clear();
        }
    }

}
~~~

### 侧输出流（ SideOutput） 

大部分的 DataStream API 的算子的输出是单一输出，也就是某种数据类型的流。除了 split 算子，可以将一条流分成多条流，这些流的数据类型也都相同。 process function 的 side outputs 功能可以产生多条流，并且这些流的数据类型可以不一样。一个 side output 可以定义为 OutputTag[X]对象， X 是输出流的数据类型。 process function 可以通过 Context 对象发射一个事件到一个或者多个 side outputs。

下面是一个示例程序，用来监控传感器温度值，将温度值低于 30 度的数据输出到 side output。 

~~~ java
public class ProcessFunTestsideoutput {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

        //        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });
        //        定义一个outputTag,用来输出低温输出流
        OutputTag<SensorReading> outputTag = new OutputTag<SensorReading>("low-stream") {
        };

        SingleOutputStreamOperator<SensorReading> highStream = map.process(new ProcessFunction<SensorReading, SensorReading>() {


            @Override
            public void processElement(SensorReading value, Context ctx, Collector<SensorReading> out) throws Exception {

//                如果当前是高温，那么就从主流输出，否则就从测输出流输出
                if(value.getTemperature() > 30){
                    out.collect(value);
                }else {
                    ctx.output(outputTag,value);
                }
            }
        });

        highStream.print("high-stream");


//        获取测输出流的输出结果
        highStream.getSideOutput(outputTag).print("low-stream");


        env.execute();
    }

}
~~~

### CoProcessFunction 

对于两条输入流， DataStream API 提供了 CoProcessFunction 这样的 low-level操作。 CoProcessFunction 提供了操作每一个输入流的方法: processElement1()和processElement2()。
类似于 ProcessFunction，这两种方法都通过 Context 对象来调用。这个 Context对象可以访问事件数据，定时器时间戳， TimerService，以及 side outputs。CoProcessFunction 也提供了 onTimer()回调函数。 

## 高级特性

### BroadcastState

#### BroadcastState介绍

在开发过程中，如果遇到需要下发/广播配置、规则等低吞吐事件流到下游所有 task 时，就可以使用 Broadcast State。Broadcast State 是 Flink 1.5 引入的新特性。

下游的 task 接收这些配置、规则并保存为 BroadcastState, 将这些配置应用到另一个数据流的计算中 。

- 场景举例
  -  动态更新计算规则: 如事件流需要根据最新的规则进行计算，则可将规则作为广播状态广播到下游Task中。
  -  实时增加额外字段: 如事件流需要实时增加用户的基础信息，则可将用户的基础信息作为广播状态广播到下游Task中。

#### API介绍

1. 首先创建一个Keyed 或Non-Keyed 的DataStream，

2. 然后再创建一个BroadcastedStream，

3. 最后通过DataStream来连接(调用connect 方法)到Broadcasted Stream 上，

4. 这样实现将BroadcastState广播到Data Stream 下游的每个Task中。

**继承关系**

![1623581448076](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/185050-105690.png)

一个类是处理非键值类型的流，一个是处理键值类型的数据流。

如果DataStream是KeyedStream ，则连接到Broadcasted Stream 后， 添加处理ProcessFunction 时需要使用KeyedBroadcastProcessFunction来实现， 下面是KeyedBroadcastProcessFunction 的API，代码如下所示：

~~~ java
public abstract class KeyedBroadcastProcessFunction<KS, IN1, IN2, OUT> extends BaseBroadcastProcessFunction {

	private static final long serialVersionUID = -2584726797564976453L;

	
	public abstract void processElement(final IN1 value, final ReadOnlyContext ctx, final Collector<OUT> out) throws Exception;

	public abstract void processBroadcastElement(final IN2 value, final Context ctx, final Collector<OUT> out) throws Exception;

	public void onTimer(final long timestamp, final OnTimerContext ctx, final Collector<OUT> out) throws Exception {
		// the default implementation does nothing.
	}

	public abstract class Context extends BaseBroadcastProcessFunction.Context {

		public abstract <VS, S extends State> void applyToKeyedState(
				final StateDescriptor<S, VS> stateDescriptor,
				final KeyedStateFunction<KS, S> function) throws Exception;
	}

	public abstract class ReadOnlyContext extends BaseBroadcastProcessFunction.ReadOnlyContext {

		public abstract TimerService timerService();

		public abstract KS getCurrentKey();
	}

	public abstract class OnTimerContext extends ReadOnlyContext {

		public abstract TimeDomain timeDomain();

		@Override
		public abstract KS getCurrentKey();
	}
}
~~~

**说明**

上面泛型中的各个参数的含义，说明如下：

- KS：表示Flink 程序从最上游的Source Operator 开始构建Stream，当调用keyBy 时所依赖的Key 的类型；

- IN1：表示非Broadcast 的Data Stream 中的数据记录的类型；

- IN2：表示Broadcast Stream 中的数据记录的类型；

- OUT：表示经过KeyedBroadcastProcessFunction 的processElement()和processBroadcastElement()方法处理后输出结果数据记录的类型。

如果Data Stream 是Non-KeyedStream，则连接到Broadcasted Stream 后，添加处理ProcessFunction 时需要使用BroadcastProcessFunction来实现， 下面是BroadcastProcessFunction 的API，代码如下所示：

~~~ java
public abstract class BroadcastProcessFunction<IN1, IN2, OUT> extends BaseBroadcastProcessFunction {

	private static final long serialVersionUID = 8352559162119034453L;
  
	public abstract void processElement(final IN1 value, final ReadOnlyContext ctx, final Collector<OUT> out) throws Exception;

	public abstract void processBroadcastElement(final IN2 value, final Context ctx, final Collector<OUT> out) throws Exception;

	public abstract class Context extends BaseBroadcastProcessFunction.Context {}

	public abstract class ReadOnlyContext extends BaseBroadcastProcessFunction.ReadOnlyContext {}
}
~~~

上面泛型中的各个参数的含义，与前面KeyedBroadcastProcessFunction 的泛型类型中的后3 个含义相同，只是没有调用keyBy 操作对原始Stream 进行分区操作，就不需要KS 泛型参数。

> 注意
>
> 1. Broadcast State 是Map 类型，即K-V 类型。
>
> 2. Broadcast State 只有在广播的一侧, 即在BroadcastProcessFunction 或KeyedBroadcastProcessFunction 的processBroadcastElement 方法中可以修改。在非广播的一侧， 即在BroadcastProcessFunction 或KeyedBroadcastProcessFunction 的processElement 方法中只读。
>
> 3. Broadcast State 中元素的顺序，在各Task 中可能不同。基于顺序的处理，需要注意。
>
> 4. Broadcast State 在Checkpoint 时，每个Task 都会Checkpoint 广播状态。
>
> 5. Broadcast State 在运行时保存在内存中，目前还不能保存在RocksDB State Backend 中。
>
>  

#### 案例需求

![1623544972039](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/093832-869681.png)

实时过滤出配置中的用户，并在事件流中补全这批用户的基础信息。

- 事件流：表示用户在某个时刻浏览或点击了某个商品，格式如下。

~~~ java
{"userID": "user_3", "eventTime": "2019-08-17 12:19:47", "eventType": "browse", "productID": 1}
{"userID": "user_2", "eventTime": "2019-08-17 12:19:48", "eventType": "click", "productID": 1}
~~~

- 配置数据: 表示用户的详细信息，在Mysql中，如下。

~~~ java
DROP TABLE IF EXISTS `user_info`;
CREATE TABLE `user_info`  (
  `userID` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `userName` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `userAge` int(11) NULL DEFAULT NULL,
  PRIMARY KEY (`userID`) USING BTREE
) ENGINE = MyISAM CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
-- ----------------------------
-- Records of user_info
-- ----------------------------
INSERT INTO `user_info` VALUES ('user_1', '张三', 10);
INSERT INTO `user_info` VALUES ('user_2', '李四', 20);
INSERT INTO `user_info` VALUES ('user_3', '王五', 30);
INSERT INTO `user_info` VALUES ('user_4', '赵六', 40);
SET FOREIGN_KEY_CHECKS = 1;
~~~

- 输出结果:


~~~ java
(user_3,2019-08-17 12:19:47,browse,1,王五,33)
(user_2,2019-08-17 12:19:48,click,1,李四,20)
~~~

**分析**

![1623545124232](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/084526-385177.png)

##### mysql中读取数据

  ~~~java
public static class MySource implements SourceFunction<Tuple4<String, String, String, Integer>>{
        private boolean isRunning = true;
        @Override
        public void run(SourceFunction.SourceContext<Tuple4<String, String, String, Integer>> ctx) throws Exception {
            Random random = new Random();
            SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            while (isRunning){
                int id = random.nextInt(4) + 1;
                String user_id = "user_" + id;
                String eventTime = df.format(new Date());
                String eventType = "type_" + random.nextInt(3);
                int productId = random.nextInt(4);
                ctx.collect(Tuple4.of(user_id,eventTime,eventType,productId));
                Thread.sleep(500);
            }
        }

        @Override
        public void cancel() {
            isRunning = false;
        }
    }
    /**
     * <用户id,<姓名,年龄>>
     */
    public static class MySQLSource extends RichSourceFunction<Map<String, Tuple2<String, Integer>>> {
        private boolean flag = true;
        private Connection conn = null;
        private PreparedStatement ps = null;
        private ResultSet rs = null;

        @Override
        public void open(Configuration parameters) throws Exception {
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/bigdata", "root", "root");
            String sql = "select `userID`, `userName`, `userAge` from `user_info`";
            ps = conn.prepareStatement(sql);
        }
        @Override
        public void run(SourceContext<Map<String, Tuple2<String, Integer>>> ctx) throws Exception {
            while (flag){
                Map<String, Tuple2<String, Integer>> map = new HashMap<>();
                ResultSet rs = ps.executeQuery();
                while (rs.next()){
                    String userID = rs.getString("userID");
                    String userName = rs.getString("userName");
                    int userAge = rs.getInt("userAge");
                    //Map<String, Tuple2<String, Integer>>
                    map.put(userID,Tuple2.of(userName,userAge));
                }
                ctx.collect(map);
                Thread.sleep(5000);//每隔5s更新一下用户的配置信息!
            }
        }
        @Override
        public void cancel() {
            flag = false;
        }
        @Override
        public void close() throws Exception {
            if (conn != null) conn.close();
            if (ps != null) ps.close();
            if (rs != null) rs.close();
        }
    }
  ~~~

##### 代码实现步骤

~~~ java
1.	env
2.	source
-1.构建实时数据事件流-自定义随机
<userID, eventTime, eventType, productID>
-2.构建配置流-从MySQL
<用户id,<姓名,年龄>>
3.	transformation
-1.定义状态描述器
MapStateDescriptor<Void, Map<String, Tuple2<String, Integer>>> descriptor =
new MapStateDescriptor<>("config",Types.VOID, Types.MAP(Types.STRING, Types.TUPLE(Types.STRING, Types.INT)));

-2.广播配置流
BroadcastStream<Map<String, Tuple2<String, Integer>>> broadcastDS = configDS.broadcast(descriptor);
-3.将事件流和广播流进行连接
BroadcastConnectedStream<Tuple4<String, String, String, Integer>, Map<String, Tuple2<String, Integer>>> connectDS =eventDS.connect(broadcastDS);
-4.处理连接后的流-根据配置流补全事件流中的用户的信息

4.	sink
5.	execute

~~~

##### 代码案例

~~~ java
public class Test24 {

    public static void main(String[] args) throws Exception {

            //1.env
            StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
            //2.source
            //-1.构建实时的自定义随机数据事件流-数据源源不断产生,量会很大
            //<userID, eventTime, eventType, productID>
            DataStreamSource<Tuple4<String, String, String, Integer>> eventDS = env.addSource(new MySource());

            //-2.构建配置流-从MySQL定期查询最新的,数据量较小
            //<用户id,<姓名,年龄>>
            DataStreamSource<Map<String, Tuple2<String, Integer>>> configDS = env.addSource(new MySQLSource());

            //3.transformation
            //-1.定义状态描述器-准备将配置流作为状态广播
            MapStateDescriptor<Void, Map<String, Tuple2<String, Integer>>> descriptor =
                    new MapStateDescriptor<>("config", Types.VOID, Types.MAP(Types.STRING, Types.TUPLE(Types.STRING, Types.INT)));
            //-2.将配置流根据状态描述器广播出去,变成广播状态流
            BroadcastStream<Map<String, Tuple2<String, Integer>>> broadcastDS = configDS.broadcast(descriptor);

            //-3.将事件流和广播流进行连接
            BroadcastConnectedStream<Tuple4<String, String, String, Integer>, Map<String, Tuple2<String, Integer>>> connectDS =eventDS.connect(broadcastDS);
            //-4.处理连接后的流-根据配置流补全事件流中的用户的信息
            SingleOutputStreamOperator<Tuple6<String, String, String, Integer, String, Integer>> result = connectDS
                    //BroadcastProcessFunction<IN1, IN2, OUT>
                    .process(new BroadcastProcessFunction<
                            //<userID, eventTime, eventType, productID> //事件流
                            Tuple4<String, String, String, Integer>,
                            //<用户id,<姓名,年龄>> //广播流
                            Map<String, Tuple2<String, Integer>>,
                            //<用户id，eventTime，eventType，productID，姓名，年龄> //需要收集的数据
                            Tuple6<String, String, String, Integer, String, Integer>>() {

                        //处理事件流中的元素
                        @Override
                        public void processElement(Tuple4<String, String, String, Integer> value, ReadOnlyContext ctx, Collector<Tuple6<String, String, String, Integer, String, Integer>> out) throws Exception {
                            //取出事件流中的userId
                            String userId = value.f0;
                            //根据状态描述器获取广播状态
                            ReadOnlyBroadcastState<Void, Map<String, Tuple2<String, Integer>>> broadcastState = ctx.getBroadcastState(descriptor);
                            if (broadcastState != null) {
                                //取出广播状态中的map<用户id,<姓名,年龄>>
                                Map<String, Tuple2<String, Integer>> map = broadcastState.get(null);
                                if (map != null) {
                                    //通过userId取map中的<姓名,年龄>
                                    Tuple2<String, Integer> tuple2 = map.get(userId);
                                    //取出tuple2中的姓名和年龄
                                    String userName = tuple2.f0;
                                    Integer userAge = tuple2.f1;
                                    out.collect(Tuple6.of(userId, value.f1, value.f2, value.f3, userName, userAge));
                                }
                            }
                        }

                        //处理广播流中的元素
                        @Override
                        public void processBroadcastElement(Map<String, Tuple2<String, Integer>> value, Context ctx, Collector<Tuple6<String, String, String, Integer, String, Integer>> out) throws Exception {
                            //value就是MySQLSource中每隔一段时间获取到的最新的map数据
                            //先根据状态描述器获取历史的广播状态
                            BroadcastState<Void, Map<String, Tuple2<String, Integer>>> broadcastState = ctx.getBroadcastState(descriptor);
                            //再清空历史状态数据
                            broadcastState.clear();
                            //最后将最新的广播流数据放到state中（更新状态数据）
                            broadcastState.put(null, value);
                        }
                    });
            //4.sink
            result.print();
            //5.execute
            env.execute();
        }


    public static class MySource implements SourceFunction<Tuple4<String, String, String, Integer>>{
        private boolean isRunning = true;
        @Override
        public void run(SourceFunction.SourceContext<Tuple4<String, String, String, Integer>> ctx) throws Exception {
            Random random = new Random();
            SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            while (isRunning){
                int id = random.nextInt(4) + 1;
                String user_id = "user_" + id;
                String eventTime = df.format(new Date());
                String eventType = "type_" + random.nextInt(3);
                int productId = random.nextInt(4);
                ctx.collect(Tuple4.of(user_id,eventTime,eventType,productId));
                Thread.sleep(500);
            }
        }

        @Override
        public void cancel() {
            isRunning = false;
        }
    }
    /**
     * <用户id,<姓名,年龄>>
     */
    public static class MySQLSource extends RichSourceFunction<Map<String, Tuple2<String, Integer>>> {
        private boolean flag = true;
        private Connection conn = null;
        private PreparedStatement ps = null;
        private ResultSet rs = null;

        @Override
        public void open(Configuration parameters) throws Exception {
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "root");
            String sql = "select `userID`, `userName`, `userAge` from `user_info`";
            ps = conn.prepareStatement(sql);
        }
        @Override
        public void run(SourceContext<Map<String, Tuple2<String, Integer>>> ctx) throws Exception {
            while (flag){
                Map<String, Tuple2<String, Integer>> map = new HashMap<>();
                ResultSet rs = ps.executeQuery();
                while (rs.next()){
                    String userID = rs.getString("userID");
                    String userName = rs.getString("userName");
                    int userAge = rs.getInt("userAge");
                    //Map<String, Tuple2<String, Integer>>
                    map.put(userID,Tuple2.of(userName,userAge));
                }
                ctx.collect(map);
                Thread.sleep(5000);//每隔5s更新一下用户的配置信息!
            }
        }
        @Override
        public void cancel() {
            flag = false;
        }
        @Override
        public void close() throws Exception {
            if (conn != null) conn.close();
            if (ps != null) ps.close();
            if (rs != null) rs.close();
        }
    }
}

2> (user_3,2021-06-13 09:36:47,type_1,1,王五,30)
3> (user_4,2021-06-13 09:36:47,type_2,1,赵六,40)
4> (user_3,2021-06-13 09:36:48,type_2,0,王五,30)
~~~

### 双流Join

#### 介绍

![1623549190628](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623549190628.png)

- Join大体分类只有两种：Window Join和Interval Join。
  - Window Join又可以根据Window的类型细分出3种：
    - Tumbling Window Join
    - Sliding Window Join
    - Session Widnow Join

Windows类型的join都是利用window的机制，先将数据缓存在Window State中，当窗口触发计算时，执行join操作；

interval join也是利用state存储数据再处理，区别在于state中的数据有失效机制，依靠数据触发数据清理；

目前Stream join的结果是数据的笛卡尔积；

#### Window Join

##### Tumbling Window Join

执行翻滚窗口联接时，具有公共键和公共翻滚窗口的所有元素将作为成对组合联接，并传递给JoinFunction或FlatJoinFunction。因为它的行为类似于内部连接，所以一个流中的元素在其滚动窗口中没有来自另一个流的元素，因此不会被发射！

如图所示，我们定义了一个大小为2毫秒的翻滚窗口，结果窗口的形式为[0,1]、[2,3]、。。。。该图显示了每个窗口中所有元素的成对组合，这些元素将传递给JoinFunction。注意，在翻滚窗口[6,7]中没有发射任何东西，因为绿色流中不存在与橙色元素⑥和⑦结合的元素。

![1623549405471](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/095724-454056.png)

**案例**

~~~ java
DataStream<Integer> orangeStream = ...DataStream<Integer> greenStream = ...
orangeStream.join(greenStream)
    .where(<KeySelector>)
    .equalTo(<KeySelector>)
    .window(TumblingEventTimeWindows.of(Time.milliseconds(2)))
    .apply (new JoinFunction<Integer, Integer, String> (){
        @Override
        public String join(Integer first, Integer second) {
            return first + "," + second;
        }
    });
~~~

##### SlidingWindow Join
在执行滑动窗口联接时，具有公共键和公共滑动窗口的所有元素将作为成对组合联接，并传递给JoinFunction或FlatJoinFunction。在当前滑动窗口中，一个流的元素没有来自另一个流的元素，则不会发射！请注意，某些元素可能会连接到一个滑动窗口中，但不会连接到另一个滑动窗口中！

在本例中，我们使用大小为2毫秒的滑动窗口，并将其滑动1毫秒，从而产生滑动窗口[-1，0]，[0,1]，[1,2]，[2,3]…。x轴下方的连接元素是传递给每个滑动窗口的JoinFunction的元素。在这里，您还可以看到，例如，在窗口[2,3]中，橙色②与绿色③连接，但在窗口[1,2]中没有与任何对象连接。

![1623549530640](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/095857-951864.png)

**案例**

~~~ java
DataStream<Integer> orangeStream = ...DataStream<Integer> greenStream = ...
orangeStream.join(greenStream)
    .where(<KeySelector>)
    .equalTo(<KeySelector>)
    .window(SlidingEventTimeWindows.of(Time.milliseconds(2) /* size */, Time.milliseconds(1) /* slide */))
    .apply (new JoinFunction<Integer, Integer, String> (){
        @Override
        public String join(Integer first, Integer second) {
            return first + "," + second;
        }
    });
~~~

##### Session Window Join

在执行会话窗口联接时，具有相同键（当“组合”时满足会话条件）的所有元素以成对组合方式联接，并传递给JoinFunction或FlatJoinFunction。同样，这执行一个内部连接，所以如果有一个会话窗口只包含来自一个流的元素，则不会发出任何输出！

在这里，我们定义了一个会话窗口连接，其中每个会话被至少1ms的间隔分割。有三个会话，在前两个会话中，来自两个流的连接元素被传递给JoinFunction。在第三个会话中，绿色流中没有元素，所以⑧和⑨没有连接！

![1623549664947](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623549664947.png)

**案例**

~~~ java
DataStream<Integer> orangeStream = ...DataStream<Integer> greenStream = ...
orangeStream.join(greenStream)
    .where(<KeySelector>)
    .equalTo(<KeySelector>)
    .window(EventTimeSessionWindows.withGap(Time.milliseconds(1)))
    .apply (new JoinFunction<Integer, Integer, String> (){
        @Override
        public String join(Integer first, Integer second) {
            return first + "," + second;
        }
    });
~~~

#### Interval Join

前面学习的Window Join必须要在一个Window中进行JOIN，那如果没有Window如何处理呢？

interval join也是使用相同的key来join两个流（流A、流B），

并且流B中的元素中的时间戳，和流A元素的时间戳，有一个时间间隔。

~~~ java
b.timestamp ∈ [a.timestamp + lowerBound; a.timestamp + upperBound] 
or 
a.timestamp + lowerBound <= b.timestamp <= a.timestamp + upperBound

也就是：
流B的元素的时间戳 ≥ 流A的元素时间戳 + 下界，且，流B的元素的时间戳 ≤ 流A的元素时间戳 + 上界。
~~~

![1623549785810](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/100308-794462.png)

在上面的示例中，我们将两个流“orange”和“green”连接起来，其下限为-2毫秒，上限为+1毫秒。默认情况下，这些边界是包含的，但是可以应用.lowerBoundExclusive（）和.upperBoundExclusive来更改行为

~~~ java
orangeElem.ts + lowerBound <= greenElem.ts <= orangeElem.ts + upperBound

//案例
DataStream<Integer> orangeStream = ...DataStream<Integer> greenStream = ...
orangeStream
    .keyBy(<KeySelector>)
    .intervalJoin(greenStream.keyBy(<KeySelector>))
    .between(Time.milliseconds(-2), Time.milliseconds(1))
    .process (new ProcessJoinFunction<Integer, Integer, String(){

        @Override
        public void processElement(Integer left, Integer right, Context ctx, Collector<String> out) {
            out.collect(first + "," + second);
        }
    });
~~~

#### Window Join代码演示

~~~ java
public class Test25 {

    public static void main(String[] args) {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);
        env.setParallelism(1);

        DataStreamSource goodSource = env.addSource(new GoodsSource11());

        DataStreamSource orderSource = env.addSource(new OrderItemSource());

//        给数据添加时间，直接使用系统时间作为事件时间
        SingleOutputStreamOperator singleOutputStreamOperator = goodSource.assignTimestampsAndWatermarks(new GoodsWatermark());
        SingleOutputStreamOperator singleOutputStreamOperator1 = orderSource.assignTimestampsAndWatermarks(new OrderItemWatermark());

        /*
         //关联结果，根据商品id进行关联
         //商品类(商品id,商品名字,商品价格)
         //订单明细类(订单id,商品id,订单数量)
         (商品id,商品名称，商品数量，商品价格*商品数量)
         */

        DataStream res = singleOutputStreamOperator.join(singleOutputStreamOperator1)
                .where(Goods::getGoodsId)
                .equalTo(OrderItem::getGoodsId)
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))
                .apply(new JoinFunction<Goods, OrderItem, FactOrderItem>() {

                    @Override
                    public FactOrderItem join(Goods first, OrderItem second) throws Exception {
                        return new FactOrderItem(first.goodsId,first.goodsName,new BigDecimal(second.count),first.goodsPrice.multiply(new BigDecimal(second.count)));
                    }
                });

    }
    //商品类(商品id,商品名字,商品价格)
    @Data
    public static class Goods {
        private String goodsId;
        private String goodsName;
        private BigDecimal goodsPrice;

        public static List<Goods> GOODS_LIST;
        public static Random r;

        static  {
            r = new Random();
            GOODS_LIST = new ArrayList<>();
            GOODS_LIST.add(new Goods("1", "小米12", new BigDecimal(4890)));
            GOODS_LIST.add(new Goods("2", "iphone12", new BigDecimal(12000)));
            GOODS_LIST.add(new Goods("3", "MacBookPro", new BigDecimal(15000)));
            GOODS_LIST.add(new Goods("4", "Thinkpad X1", new BigDecimal(9800)));
            GOODS_LIST.add(new Goods("5", "MeiZu One", new BigDecimal(3200)));
            GOODS_LIST.add(new Goods("6", "Mate 40", new BigDecimal(6500)));
        }

        public static Goods randomGoods() {
            int rIndex = r.nextInt(GOODS_LIST.size());
            return GOODS_LIST.get(rIndex);
        }

        public  String getGoodsId() {
            return goodsId;
        }

        public Goods() {
        }

        public Goods(String goodsId, String goodsName, BigDecimal goodsPrice) {
            this.goodsId = goodsId;
            this.goodsName = goodsName;
            this.goodsPrice = goodsPrice;
        }

        @Override
        public String toString() {
            return JSON.toJSONString(this);
        }
    }

    //订单明细类(订单id,商品id,订单数量)
    @Data
    public static class OrderItem {
        private String itemId;
        private String goodsId;
        private Integer count;

        @Override
        public String toString() {
            return JSON.toJSONString(this);
        }
    }

    //关联结果，根据商品id进行关联
    //商品类(商品id,商品名字,商品价格)
    //订单明细类(订单id,商品id,订单数量)
    @Data
    @AllArgsConstructor
    public static class FactOrderItem {
        private String goodsId;//商品id
        private String goodsName;//商品名字
        private BigDecimal count;//数量
        private BigDecimal totalMoney;//总计
        @Override
        public String toString() {
            return JSON.toJSONString(this);
        }
    }

    //构建一个商品Stream源（这个好比就是维表）
//    实时生成商品数据流
    public static class GoodsSource11 extends RichSourceFunction {
        private Boolean isCancel;
        @Override
        public void open(Configuration parameters) throws Exception {
            isCancel = false;
        }
        @Override
        public void run(SourceContext sourceContext) throws Exception {
            while(!isCancel) {
                Goods.GOODS_LIST.stream().forEach(goods -> sourceContext.collect(goods));
                TimeUnit.SECONDS.sleep(1);
            }
        }
        @Override
        public void cancel() {
            isCancel = true;
        }
    }

    //构建订单明细Stream源
//    实时生成订单明细表
    public static class OrderItemSource extends RichSourceFunction {
        private Boolean isCancel;
        private Random r;
        @Override
        public void open(Configuration parameters) throws Exception {
            isCancel = false;
            r = new Random();
        }
        @Override
        public void run(SourceContext sourceContext) throws Exception {
//            表示一个订单里面有多个商品
            while(!isCancel) {
                Goods goods = Goods.randomGoods();
                OrderItem orderItem = new OrderItem();
                orderItem.setGoodsId(goods.getGoodsId());
                orderItem.setCount(r.nextInt(10) + 1);
                orderItem.setItemId(UUID.randomUUID().toString());
                sourceContext.collect(orderItem);
                orderItem.setGoodsId("111");
                sourceContext.collect(orderItem);
                TimeUnit.SECONDS.sleep(1);
            }
        }

        @Override
        public void cancel() {
            isCancel = true;
        }
    }

    //构建水印分配器（此处为了简单），直接使用系统时间了
    public static class GoodsWatermark implements WatermarkStrategy<Goods> {

//        创建一个时间戳
        @Override
        public TimestampAssigner<Goods> createTimestampAssigner(TimestampAssignerSupplier.Context context) {
            return (element, recordTimestamp) -> System.currentTimeMillis();
        }

        @Override
        public WatermarkGenerator<Goods> createWatermarkGenerator(WatermarkGeneratorSupplier.Context context) {
            return new WatermarkGenerator<Goods>() {
                @Override
                public void onEvent(Goods event, long eventTimestamp, WatermarkOutput output) {
                    output.emitWatermark(new Watermark(System.currentTimeMillis()));
                }

                @Override
                public void onPeriodicEmit(WatermarkOutput output) {
                    output.emitWatermark(new Watermark(System.currentTimeMillis()));
                }
            };
        }
    }

    public static class OrderItemWatermark implements WatermarkStrategy<OrderItem> {
        @Override
        public TimestampAssigner<OrderItem> createTimestampAssigner(TimestampAssignerSupplier.Context context) {
            return (element, recordTimestamp) -> System.currentTimeMillis();
        }
        @Override
        public WatermarkGenerator<OrderItem> createWatermarkGenerator(WatermarkGeneratorSupplier.Context context) {
            return new WatermarkGenerator<OrderItem>() {
                @Override
                public void onEvent(OrderItem event, long eventTimestamp, WatermarkOutput output) {
                    output.emitWatermark(new Watermark(System.currentTimeMillis()));
                }
                @Override
                public void onPeriodicEmit(WatermarkOutput output) {
                    output.emitWatermark(new Watermark(System.currentTimeMillis()));
                }
            };
        }
    }
}
~~~

#### intervalJoin代码演示

~~~ java
public class Test26 {

    public static void main(String[] args) {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);
        env.setParallelism(1);

        DataStreamSource goodSource = env.addSource(new GoodsSource11());

        DataStreamSource orderSource = env.addSource(new OrderItemSource());

//        给数据添加时间，直接使用系统时间作为事件时间
        SingleOutputStreamOperator singleOutputStreamOperator = goodSource.assignTimestampsAndWatermarks(new GoodsWatermark());
        SingleOutputStreamOperator singleOutputStreamOperator1 = orderSource.assignTimestampsAndWatermarks(new OrderItemWatermark());

        /*
         //关联结果，根据商品id进行关联
         //商品类(商品id,商品名字,商品价格)
         //订单明细类(订单id,商品id,订单数量)
         (商品id,商品名称，商品数量，商品价格*商品数量)
         */
        /*
        join的条件
            1 商品的id要相等
            2 orderItem时间戳-2<=商品的时间戳<=orderItem时间戳+1
         */

        SingleOutputStreamOperator process = singleOutputStreamOperator.keyBy(Goods::getGoodsId)
                .intervalJoin(singleOutputStreamOperator1.keyBy(OrderItem::getGoodsId))
                .between(Time.seconds(-2), Time.seconds(1))
                .process(new ProcessJoinFunction<Goods, OrderItem, FactOrderItem>() {

                    @Override
                    public void processElement(Goods left, OrderItem right, Context ctx, Collector<FactOrderItem> out) throws Exception {
                        out.collect(new FactOrderItem(left.goodsId, left.goodsName, new BigDecimal(right.count), left.goodsPrice.multiply(new BigDecimal(right.count))));
                    }
                });

    }
    //商品类(商品id,商品名字,商品价格)
    @Data
    public static class Goods {
        private String goodsId;
        private String goodsName;
        private BigDecimal goodsPrice;

        public static List<Goods> GOODS_LIST;
        public static Random r;

        static  {
            r = new Random();
            GOODS_LIST = new ArrayList<>();
            GOODS_LIST.add(new Goods("1", "小米12", new BigDecimal(4890)));
            GOODS_LIST.add(new Goods("2", "iphone12", new BigDecimal(12000)));
            GOODS_LIST.add(new Goods("3", "MacBookPro", new BigDecimal(15000)));
            GOODS_LIST.add(new Goods("4", "Thinkpad X1", new BigDecimal(9800)));
            GOODS_LIST.add(new Goods("5", "MeiZu One", new BigDecimal(3200)));
            GOODS_LIST.add(new Goods("6", "Mate 40", new BigDecimal(6500)));
        }

        public static Goods randomGoods() {
            int rIndex = r.nextInt(GOODS_LIST.size());
            return GOODS_LIST.get(rIndex);
        }

        public  String getGoodsId() {
            return goodsId;
        }

        public Goods() {
        }

        public Goods(String goodsId, String goodsName, BigDecimal goodsPrice) {
            this.goodsId = goodsId;
            this.goodsName = goodsName;
            this.goodsPrice = goodsPrice;
        }

        @Override
        public String toString() {
            return JSON.toJSONString(this);
        }
    }

    //订单明细类(订单id,商品id,订单数量)
    @Data
    public static class OrderItem {
        private String itemId;
        private String goodsId;
        private Integer count;

        @Override
        public String toString() {
            return JSON.toJSONString(this);
        }
    }

    //关联结果，根据商品id进行关联
    //商品类(商品id,商品名字,商品价格)
    //订单明细类(订单id,商品id,订单数量)
    @Data
    @AllArgsConstructor
    public static class FactOrderItem {
        private String goodsId;//商品id
        private String goodsName;//商品名字
        private BigDecimal count;//数量
        private BigDecimal totalMoney;//总计
        @Override
        public String toString() {
            return JSON.toJSONString(this);
        }
    }

    //构建一个商品Stream源（这个好比就是维表）
//    实时生成商品数据流
    public static class GoodsSource11 extends RichSourceFunction {
        private Boolean isCancel;
        @Override
        public void open(Configuration parameters) throws Exception {
            isCancel = false;
        }
        @Override
        public void run(SourceContext sourceContext) throws Exception {
            while(!isCancel) {
                Goods.GOODS_LIST.stream().forEach(goods -> sourceContext.collect(goods));
                TimeUnit.SECONDS.sleep(1);
            }
        }
        @Override
        public void cancel() {
            isCancel = true;
        }
    }

    //构建订单明细Stream源
//    实时生成订单明细表
    public static class OrderItemSource extends RichSourceFunction {
        private Boolean isCancel;
        private Random r;
        @Override
        public void open(Configuration parameters) throws Exception {
            isCancel = false;
            r = new Random();
        }
        @Override
        public void run(SourceContext sourceContext) throws Exception {
//            表示一个订单里面有多个商品
            while(!isCancel) {
                Goods goods = Goods.randomGoods();
                OrderItem orderItem = new OrderItem();
                orderItem.setGoodsId(goods.getGoodsId());
                orderItem.setCount(r.nextInt(10) + 1);
                orderItem.setItemId(UUID.randomUUID().toString());
                sourceContext.collect(orderItem);
                orderItem.setGoodsId("111");
                sourceContext.collect(orderItem);
                TimeUnit.SECONDS.sleep(1);
            }
        }

        @Override
        public void cancel() {
            isCancel = true;
        }
    }

    //构建水印分配器（此处为了简单），直接使用系统时间了
    public static class GoodsWatermark implements WatermarkStrategy<Goods> {

//        创建一个时间戳
        @Override
        public TimestampAssigner<Goods> createTimestampAssigner(TimestampAssignerSupplier.Context context) {
            return (element, recordTimestamp) -> System.currentTimeMillis();
        }

        @Override
        public WatermarkGenerator<Goods> createWatermarkGenerator(WatermarkGeneratorSupplier.Context context) {
            return new WatermarkGenerator<Goods>() {
                @Override
                public void onEvent(Goods event, long eventTimestamp, WatermarkOutput output) {
                    output.emitWatermark(new Watermark(System.currentTimeMillis()));
                }

                @Override
                public void onPeriodicEmit(WatermarkOutput output) {
                    output.emitWatermark(new Watermark(System.currentTimeMillis()));
                }
            };
        }
    }

    public static class OrderItemWatermark implements WatermarkStrategy<OrderItem> {
        @Override
        public TimestampAssigner<OrderItem> createTimestampAssigner(TimestampAssignerSupplier.Context context) {
            return (element, recordTimestamp) -> System.currentTimeMillis();
        }
        @Override
        public WatermarkGenerator<OrderItem> createWatermarkGenerator(WatermarkGeneratorSupplier.Context context) {
            return new WatermarkGenerator<OrderItem>() {
                @Override
                public void onEvent(OrderItem event, long eventTimestamp, WatermarkOutput output) {
                    output.emitWatermark(new Watermark(System.currentTimeMillis()));
                }
                @Override
                public void onPeriodicEmit(WatermarkOutput output) {
                    output.emitWatermark(new Watermark(System.currentTimeMillis()));
                }
            };
        }
    }
}
~~~

### End-To-End-Exactly-Once

Flink 在1.4.0 版本引入『exactly-once』并号称支持『End-to-End Exactly-Once』“端到端的精确一次”语义。

#### 流处理的数据处理语义分类

对于批处理，fault-tolerant（容错性）很容易做，失败只需要replay(从新执行一次数据处理)，就可以完美做到容错。

对于流处理，数据流本身是动态，没有所谓的开始或结束，虽然可以replay buffer的部分数据，但fault-tolerant做起来会复杂的多

流处理（有时称为事件处理）可以简单地描述为是对无界数据或事件的连续处理。流或事件处理应用程序可以或多或少地被描述为有向图，并且通常被描述为有向无环图（DAG）。在这样的图中，每个边表示数据或事件流，每个顶点表示运算符，会使用程序中定义的逻辑处理来自相邻边的数据或事件。有两种特殊类型的顶点，通常称为 sources 和 sinks。sources读取外部数据/事件到应用程序中，而 sinks 通常会收集应用程序生成的结果。下图是流式应用程序的示例。有如下特点：

- 分布式情况下是由多个Source(读取数据)节点、多个Operator(数据处理)节点、多个Sink(输出)节点构成

- 每个节点的并行数可以有差异，且每个节点都有可能发生故障

- 对于数据正确性最重要的一点，就是当发生故障时，是怎样容错与恢复的。

![1623555514835](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/113845-828486.png)

流处理引擎通常为应用程序提供了三种数据处理语义：最多一次、至少一次和精确一次。

- At most Once:最多一次，数据可能发生丢失
- At last Once :至少一次，数据有可能重复处理
- Exactly Once:精确一次，恰好只被正确处理

如下是对这些不同处理语义的宽松定义(一致性由弱到强)：

At most noce < At least once < Exactly once < End to End Exactly once

>Flink程序分为三部分：source，Transform，sink，中间某一部分可以做到Exactly Once，那么只能够保证局部的Exactly Once，如果source,transform,sink都可以做到Exactly Once，那么就是end-to-end-Exactly-Once。
>
>Flink可以做到端到端的精确一致性保证

##### At-most-once-最多一次

有可能会有数据丢失

这本质上是简单的恢复方式，也就是直接从失败处的下个数据开始恢复程序，之前的失败数据处理就不管了。可以保证数据或事件最多由应用程序中的所有算子处理一次。 这意味着如果数据在被流应用程序完全处理之前发生丢失，则不会进行其他重试或者重新发送。

![1623556123058](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/114846-499608.png)

##### At-least-once-至少一次

有可能重复处理数据

应用程序中的所有算子都保证数据或事件至少被处理一次。这通常意味着如果事件在流应用程序完全处理之前丢失，则将从源头重放或重新传输事件。然而，由于事件是可以被重传的，因此一个事件有时会被处理多次(至少一次)，至于有没有重复数据，不会关心，所以这种场景需要人工干预自己处理重复数据

![1623556237603](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623556237603.png)

##### Exactly-once-精确一次

Exactly-Once 是 Flink、Spark 等流处理系统的核心特性之一，这种语义会保证每一条消息只被流处理系统处理一次。即使是在各种故障的情况下，流应用程序中的所有算子都保证事件只会被『精确一次』的处理。（也有文章将 Exactly-once 翻译为：完全一次，恰好一次）

Flink实现『精确一次』的分布式快照/状态检查点方法受到 Chandy-Lamport 分布式快照算法的启发。通过这种机制，流应用程序中每个算子的所有状态都会定期做 checkpoint。如果是在系统中的任何地方发生失败，每个算子的所有状态都回滚到最新的全局一致 checkpoint 点。在回滚期间，将暂停所有处理。源也会重置为与最近 checkpoint 相对应的正确偏移量。整个流应用程序基本上是回到最近一次的一致状态，然后程序可以从该状态重新启动。

虽然会重复处理，但是只有一次成功的正确处理，其他的处理可能不是正确的或者是失败的。

![1623556776050](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/115938-145582.png)

![1623556425173](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/115348-874699.png)

#### End-to-End Exactly-Once-端到端的精确一次

上面的数据处理语义只能保证中间某一部分精确一次，但是End-to-End Exactly-Once可以保证端到端的精确一次。

Flink 在1.4.0 版本引入『exactly-once』并号称支持『End-to-End Exactly-Once』“端到端的精确一次”语义。

它指的是 Flink 应用从 Source 端开始到 Sink 端结束，数据必须经过的起始点和结束点。

注意：『exactly-once』和『End-to-End Exactly-Once』的区别:

**Exactly-once**

![1623557212055](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/15/084207-723822.png)

**End-to-End Exactly-Once**

![1623557238448](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/120720-888676.png)

![1623557272956](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/120809-317181.png)

##### 注意：精确一次? 有效一次!

有些人可能认为『精确一次』描述了事件处理的保证，其中流中的每个事件只被处理一次。实际上，没有引擎能够保证正好只处理一次。在面对任意故障时，不可能保证每个算子中的用户定义逻辑在每个事件中只执行一次，因为用户代码被部分执行的可能性是永远存在的。

那么，当引擎声明『精确一次』处理语义时，它们能保证什么呢？如果不能保证用户逻辑只执行一次，那么什么逻辑只执行一次？当引擎声明『精确一次』处理语义时，它们实际上是在说，它们可以保证引擎管理的状态更新只提交一次到持久的后端存储。

事件的处理可以发生多次，但是该处理的效果只在持久后端状态存储中反映一次。因此，我们认为有效地描述这些处理语义最好的术语是『有效一次』（effectively once）

#### 如何实现Exactly-Once

- 去重+at last once
- 幂等性
- 分布式快照+checkpoint(flink使用的方法)

**at last once+去重**

![1623557390398](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/120953-591150.png)

at last once可能发生去重，如果可以把重复的数据去掉，那么就可以实现精确一次。

**幂等性+at last once**

幂等性的含义是：多次操作，结果都是一样的

![1623557604692](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/121328-286143.png)

at least once 数据可能发生重复，使用幂等性可以保证多次写入重复的数据，仅仅保留一次即可，也就是会覆盖前面的写入操作。

**分布式快照+checkpoint**

![1623557781295](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/121625-17303.png)

**小结**

![1623557838127](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/121721-206113.png)

#### 如何实现End-to-End Exactly-Once

![1623558410989](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/122653-809639.png)

- Source端：添加偏移量，如kafka的offset支持数据的重放或者重新传输。

- transform:借助于checkpoint

- sink:checkpoint+两阶段事务提交

通过前面的学习，我们了解到，Flink内部借助分布式快照Checkpoint已经实现了内部的Exactly-Once，但是Flink 自身是无法保证外部其他系统“精确一次”语义的，所以 Flink 若要实现所谓“端到端（End to End）的精确一次”的要求，那么外部系统必须支持“精确一次”语义；然后借助一些其他手段才能实现。如下：

##### Source

发生故障时需要支持重设数据的读取位置，如Kafka可以通过offset来实现（其他的没有offset系统，我们可以自己实现累加器计数）

##### Transformation

也就是Flink内部，已经通过Checkpoint保证了，如果发生故障或出错时，Flink应用重启后会从最新成功完成的checkpoint中恢复——重置应用状态并回滚状态到checkpoint中输入流的正确位置，之后再开始执行数据处理，就好像该故障或崩溃从未发生过一般。

**分布式快照机制**

我们在之前的课程中讲解过 Flink 的容错机制，Flink 提供了失败恢复的容错机制，而这个容错机制的核心就是持续创建分布式数据流的快照来实现。

![1623558529428](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/122851-330177.png)

同 Spark 相比，Spark 仅仅是针对 Driver 的故障恢复 Checkpoint。而 Flink 的快照可以到算子级别，并且对全局数据也可以做快照。Flink 的分布式快照受到  Chandy-Lamport 分布式快照算法启发，同时进行了量身定做。

**Barrier**

Flink 分布式快照的核心元素之一是 Barrier（数据栅栏），我们也可以把 Barrier 简单地理解成一个标记，该标记是严格有序的，并且随着数据流往下流动。每个 Barrier 都带有自己的 ID，Barrier极其轻量，并不会干扰正常的数据处理。

![1623558616790](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623558616790.png)

如上图所示，假如我们有一个从左向右流动的数据流，Flink 会依次生成 snapshot 1、 snapshot 2、snapshot 3……Flink 中有一个专门的“协调者”负责收集每个 snapshot 的位置信息，这个“协调者”也是高可用的。

Barrier 会随着正常数据继续往下流动，每当遇到一个算子，算子会插入一个标识，这个标识的插入时间是上游所有的输入流都接收到 snapshot n。与此同时，当我们的 sink 算子接收到所有上游流发送的 Barrier 时，那么就表明这一批数据处理完毕，Flink 会向“协调者”发送确认消息，表明当前的 snapshot n 完成了。当所有的 sink 算子都确认这批数据成功处理后，那么本次的 snapshot 被标识为完成。

这里就会有一个问题，因为 Flink 运行在分布式环境中，一个 operator 的上游会有很多流，每个流的 barrier n 到达的时间不一致怎么办？这里 Flink 采取的措施是：快流等慢流。

![1623558661151](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623558661151.png)

拿上图的 barrier n 来说，其中一个流到的早，其他的流到的比较晚。当第一个 barrier n到来后，当前的 operator 会继续等待其他流的 barrier n。直到所有的barrier n 到来后，operator 才会把所有的数据向下发送。

**异步和增量**

按照上面我们介绍的机制，每次在把快照存储到我们的状态后端时，如果是同步进行就会阻塞正常任务，从而引入延迟。因此 Flink 在做快照存储时，可采用异步方式。 

此外，由于 checkpoint 是一个全局状态，用户保存的状态可能非常大，多数达 G 或者 T 级别。在这种情况下，checkpoint 的创建会非常慢，而且执行时占用的资源也比较多，因此 Flink 提出了增量快照的概念。也就是说，每次都是进行的全量 checkpoint，是基于上次进行更新的。

##### sink

需要支持幂等写入或事务写入(Flink的两阶段提交需要事务支持)

##### 幂等写入

幂等写操作是指：任意多次向一个系统写入数据，只对目标系统产生一次结果影响。

例如，重复向一个HashMap里插入同一个Key-Value二元对，第一次插入时这个HashMap发生变化，后续的插入操作不会改变HashMap的结果，这就是一个幂等写操作。

HBase、Redis和Cassandra这样的KV数据库一般经常用来作为Sink，用以实现端到端的Exactly-Once。

需要注意的是，并不是说一个KV数据库就百分百支持幂等写。幂等写对KV对有要求，那就是Key-Value必须是可确定性（Deterministic）计算的。假如我们设计的Key是：name + curTimestamp，每次执行数据重发时，生成的Key都不相同，会产生多次结果，整个操作不是幂等的。因此，为了追求端到端的Exactly-Once，我们设计业务逻辑时要尽量使用确定性的计算逻辑和数据模型。

##### 事务写入（Transactional Writes）

Flink借鉴了数据库中的事务处理技术，同时结合自身的Checkpoint机制来保证Sink只对外部输出产生一次影响。大致的流程如下:

Flink先将待输出的数据保存下来暂时不向外部系统提交，等到Checkpoint结束时，Flink上下游所有算子的数据都是一致的时候，Flink将之前保存的数据全部提交（Commit）到外部系统。换句话说，只有经过Checkpoint确认的数据才向外部系统写入。

如下图所示，如果使用事务写，那只把时间戳3之前的输出提交到外部系统，时间戳3以后的数据（例如时间戳5和8生成的数据）暂时保存下来，等待下次Checkpoint时一起写入到外部系统。这就避免了时间戳5这个数据产生多次结果，多次写入到外部系统。

![1623558867291](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623558867291.png)

在事务写的具体实现上，Flink目前提供了两种方式：

1. 预写日志（Write-Ahead-Log，WAL）

2. 两阶段提交（Two-Phase-Commit，2PC）

这两种方式区别主要在于：

1. WAL方式通用性更强，适合几乎所有外部系统，但也不能提供百分百端到端的Exactly-Once，因为WAL预习日志会先写内存，而内存是易失介质。

2. 如果外部系统自身就支持事务（比如MySQL、Kafka），可以使用2PC方式，可以提供百分百端到端的Exactly-Once。

事务写的方式能提供端到端的Exactly-Once一致性，它的代价也是非常明显的，就是牺牲了延迟。输出数据不再是实时写入到外部系统，而是分批次地提交。目前来说，没有完美的故障恢复和Exactly-Once保障机制，对于开发者来说，需要在不同需求之间权衡。

#### Flink+Kafka的End-to-End Exactly-Once

在上一小节我们了解到Flink的 End-to-End Exactly-Once需要Checkpoint+事务的提交/回滚操作，在分布式系统中协调提交和回滚的一个常见方法就是使用两阶段提交协议。接下来我们了解下Flink的TwoPhaseCommitSinkFunction是如何支持End-to-End Exactly-Once的

Flink 1.4版本之前，支持Exactly Once语义，仅限于应用内部。

Flink 1.4版本之后，通过两阶段提交(TwoPhaseCommitSinkFunction)支持End-To-End Exactly Once，而且要求Kafka 0.11+。

利用TwoPhaseCommitSinkFunction是通用的管理方案，只要实现对应的接口，而且Sink的存储支持变乱提交，即可实现端到端的划一性语义。

![1623559036545](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/15/084429-45106.png)

##### 两阶段提交-API

在 Flink 中的Two-Phase-Commit-2PC两阶段提交的实现方法被封装到了
TwoPhaseCommitSinkFunction 这个抽象类中，只需要实现其中的beginTransaction、preCommit、commit、abort四个方法就可以实现“精确一次”的处理语义，如FlinkKafkaProducer就实现了该类并实现了这些方法

![1623559108050](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623559108050.png)

1. beginTransaction，在开启事务之前，我们在目标文件系统的临时目录中创建一个临时文件，后面在处理数据时将数据写入此文件；

2. preCommit，在预提交阶段，刷写（flush）文件，然后关闭文件，之后就不能写入到文件了，我们还将为属于下一个检查点的任何后续写入启动新事务；

3. commit，在提交阶段，我们将预提交的文件原子性移动到真正的目标目录中，请注意，这会增加输出数据可见性的延迟；

4. abort，在中止阶段，我们删除临时文件。

##### 两阶段提交简单流程

![1623559462940](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/124428-42354.png)

![1623559635327](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/124719-734505.png)

并不会出现重复提交的情况，因为首先会进行预提交，预提交成功后，在进行一次全部真正的提交。

##### 两阶段提交-详细流程

接下来将介绍两阶段提交协议，以及它如何在一个读写Kafka的Flink程序中实现端到端的Exactly-Once语义。Kafka经常与Flink一起使用，且Kafka在最近的0.11版本中添加了对事务的支持。这意味着现在通过Flink读写Kafaka，并提供`[端到端的Exactly-Once语义有了必要的支持](https://ci.apache.org/projects/flink/flink-docs-release-1.4/dev/connectors/kafka.html#kafka-011)。`

![1623559969911](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623559969911.png)

在上图中，我们有：

- 从Kafka读取的数据源（Flink内置的[KafkaConsumer])

- 窗口聚合

- 将数据写回Kafka的数据输出端（Flink内置的[KafkaProducer])

要使数据输出端提供Exactly-Once保证，它必须将所有数据通过一个事务提交给Kafka。提交捆绑了两个checkpoint之间的所有要写入的数据。这可确保在发生故障时能回滚写入的数据。

但是在分布式系统中，通常会有多个并发运行的写入任务的，简单的提交或回滚是不够的，因为所有组件必须在提交或回滚时“一致”才能确保一致的结果。

Flink使用两阶段提交协议及预提交阶段来解决这个问题。

 **预提交-内部状态**

在checkpoint开始的时候，即两阶段提交协议的“预提交”阶段。当checkpoint开始时，Flink的JobManager会将checkpoint barrier（将数据流中的记录分为进入当前checkpoint与进入下一个checkpoint）注入数据流。

brarrier在operator之间传递。对于每一个operator，它触发operator的状态快照写入到state backend。

![1623560123470](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623560123470.png)

数据源保存了消费Kafka的偏移量(offset)，之后将checkpoint barrier传递给下一个operator。

这种方式仅适用于operator具有『内部』状态。所谓内部状态，是指Flink state backend保存和管理的 -例如，第二个operator中window聚合算出来的sum值。当一个进程有它的内部状态的时候，除了在checkpoint之前需要将数据变更写入到state backend，不需要在预提交阶段执行任何其他操作。Flink负责在checkpoint成功的情况下正确提交这些写入，或者在出现故障时中止这些写入。

![1623560184225](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623560184225.png)

**预提交-外部状态**

但是，当进程具有『外部』状态时，需要作些额外的处理。外部状态通常以写入外部系统（如Kafka）的形式出现。在这种情况下，为了提供Exactly-Once保证，外部系统必须支持事务，这样才能和两阶段提交协议集成。

在该示例中的数据需要写入Kafka，因此数据输出端（Data Sink）有外部状态。在这种情况下，在预提交阶段，除了将其状态写入state backend之外，数据输出端还必须预先提交其外部事务。

![1623560271749](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/125804-409449.png)

当checkpoint barrier在所有operator都传递了一遍，并且触发的checkpoint回调成功完成时，预提交阶段就结束了。所有触发的状态快照都被视为该checkpoint的一部分。checkpoint是整个应用程序状态的快照，包括预先提交的外部状态。如果发生故障，我们可以回滚到上次成功完成快照的时间点。

**提交阶段**

下一步是通知所有operator，checkpoint已经成功了。这是两阶段提交协议的提交阶段，JobManager为应用程序中的每个operator发出checkpoint已完成的回调。

数据源和widnow operator没有外部状态，因此在提交阶段，这些operator不必执行任何操作。但是，数据输出端（Data Sink）拥有外部状态，此时应该提交外部事务。

![1623560344247](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623560344247.png)

**总结**

我们对上述知识点总结下：

1. 一旦所有operator完成预提交，就提交一个commit。

2. 如果只要有一个预提交失败，则所有其他提交都将中止，我们将回滚到上一个成功完成的checkpoint。

3. 在预提交成功之后，提交的commit需要保证最终成功 – operator和外部系统都需要保障这点。如果commit失败（例如，由于间歇性网络问题），整个Flink应用程序将失败，应用程序将根据用户的重启策略重新启动，还会尝试再提交。这个过程至关重要，因为如果commit最终没有成功，将会导致数据丢失。

4. 完整的实现两阶段提交协议可能有点复杂，这就是为什么Flink将它的通用逻辑提取到抽象类TwoPhaseCommitSinkFunction中的原因。

#### 代码演示

##### Flink+Kafka实现End-to-End Exactly-Once

~~~ java
/*
 * Desc
 * Kafka --> Flink-->Kafka  的End-To-End-Exactly-once
 * 直接使用
 * FlinkKafkaConsumer  +  Flink的Checkpoint  +  FlinkKafkaProducer
 */
public class Kafka_Flink_Kafka_EndToEnd_ExactlyOnce {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //===========Checkpoint参数设置====
        //===========类型1:必须参数=============
        //设置Checkpoint的时间间隔为1000ms做一次Checkpoint/其实就是每隔1000ms发一次Barrier!
        env.enableCheckpointing(1000);
        //设置State状态存储介质
        if (SystemUtils.IS_OS_WINDOWS) {
            env.setStateBackend(new FsStateBackend("file:///D:/ckp"));
        } else {
            env.setStateBackend(new FsStateBackend("hdfs://node1:8020/flink-checkpoint/checkpoint"));
        }
        //===========类型2:建议参数===========
        //设置两个Checkpoint 之间最少等待时间,如设置Checkpoint之间最少是要等 500ms(为了避免每隔1000ms做一次Checkpoint的时候,前一次太慢和后一次重叠到一起去了)
        //如:高速公路上,每隔1s关口放行一辆车,但是规定了两车之前的最小车距为500m
        env.getCheckpointConfig().setMinPauseBetweenCheckpoints(500);//默认是0
        //设置如果在做Checkpoint过程中出现错误，是否让整体任务失败：true是  false不是
        //env.getCheckpointConfig().setFailOnCheckpointingErrors(false);//默认是true
        env.getCheckpointConfig().setTolerableCheckpointFailureNumber(10);//默认值为0，表示不容忍任何检查点失败
        //设置是否清理检查点,表示 Cancel 时是否需要保留当前的 Checkpoint，默认 Checkpoint会在作业被Cancel时被删除
        //ExternalizedCheckpointCleanup.DELETE_ON_CANCELLATION：true,当作业被取消时，删除外部的checkpoint(默认值)
        //ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION：false,当作业被取消时，保留外部的checkpoint
        env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);

        //===========类型3:直接使用默认的即可===============
        //设置checkpoint的执行模式为EXACTLY_ONCE(默认)
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        //设置checkpoint的超时时间,如果 Checkpoint在 60s内尚未完成说明该次Checkpoint失败,则丢弃。
        env.getCheckpointConfig().setCheckpointTimeout(60000);//默认10分钟
        //设置同一时间有多少个checkpoint可以同时执行
        env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);//默认为1

        //=============重启策略===========
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, Time.of(10, TimeUnit.SECONDS)));

        //2.Source
        Properties props_source = new Properties();
        props_source.setProperty("bootstrap.servers", "node1:9092");
        props_source.setProperty("group.id", "flink");
        props_source.setProperty("auto.offset.reset", "latest");
        props_source.setProperty("flink.partition-discovery.interval-millis", "5000");//会开启一个后台线程每隔5s检测一下Kafka的分区情况
        //props_source.setProperty("enable.auto.commit", "true");//没有Checkpoint的时候使用自动提交偏移量到默认主题:__consumer_offsets中
        //props_source.setProperty("auto.commit.interval.ms", "2000");
        //kafkaSource就是KafkaConsumer
        FlinkKafkaConsumer<String> kafkaSource = new FlinkKafkaConsumer<>("flink_kafka", new SimpleStringSchema(), props_source);
        kafkaSource.setStartFromLatest();
        //kafkaSource.setStartFromGroupOffsets();//设置从记录的offset开始消费,如果没有记录从auto.offset.reset配置开始消费
        //kafkaSource.setStartFromEarliest();//设置直接从Earliest消费,和auto.offset.reset配置无关
        kafkaSource.setCommitOffsetsOnCheckpoints(true);//执行Checkpoint的时候提交offset到Checkpoint(Flink用),并且提交一份到默认主题:__consumer_offsets(外部其他系统想用的话也可以获取到)
        DataStreamSource<String> kafkaDS = env.addSource(kafkaSource);

        //3.Transformation
        //3.1切割出每个单词并直接记为1
        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOneDS = kafkaDS.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                //value就是每一行
                String[] words = value.split(" ");
                for (String word : words) {
                    Random random = new Random();
                    int i = random.nextInt(5);
                    if (i > 3) {
                        System.out.println("出bug了...");
                        throw new RuntimeException("出bug了...");
                    }
                    out.collect(Tuple2.of(word, 1));
                }
            }
        });
        //3.2分组
        //注意:批处理的分组是groupBy,流处理的分组是keyBy
        KeyedStream<Tuple2<String, Integer>, Tuple> groupedDS = wordAndOneDS.keyBy(0);
        //3.3聚合
        SingleOutputStreamOperator<Tuple2<String, Integer>> aggResult = groupedDS.sum(1);
        //3.4将聚合结果转为自定义的字符串格式
        SingleOutputStreamOperator<String> result = (SingleOutputStreamOperator<String>) aggResult.map(new RichMapFunction<Tuple2<String, Integer>, String>() {
            @Override
            public String map(Tuple2<String, Integer> value) throws Exception {
                return value.f0 + ":::" + value.f1;
            }
        });

        //4.sink
        //result.print();
        Properties props_sink = new Properties();
        props_sink.setProperty("bootstrap.servers", "node1:9092");
        props_sink.setProperty("transaction.timeout.ms", 1000 * 5 + "");//设置事务超时时间，也可在kafka配置中设置
        /*FlinkKafkaProducer<String> kafkaSink0 = new FlinkKafkaProducer<>(
                "flink_kafka",
                new SimpleStringSchema(),
                props_sink);*/
        FlinkKafkaProducer<String> kafkaSink = new FlinkKafkaProducer<>(
                "flink_kafka2",
                new KeyedSerializationSchemaWrapper<String>(new SimpleStringSchema()),
                props_sink,
                FlinkKafkaProducer.Semantic.EXACTLY_ONCE
        );
        result.addSink(kafkaSink);

        //5.execute
        env.execute();
        //测试:
        //1.创建主题 /export/server/kafka/bin/kafka-topics.sh --zookeeper node1:2181 --create --replication-factor 2 --partitions 3 --topic flink_kafka2
        //2.开启控制台生产者 /export/server/kafka/bin/kafka-console-producer.sh --broker-list node1:9092 --topic flink_kafka
        //3.开启控制台消费者 /export/server/kafka/bin/kafka-console-consumer.sh --bootstrap-server node1:9092 --topic flink_kafka2
    }
}
~~~

##### Flink+MySQL实现End-to-End Exactly-Once

1. checkpoint每10s进行一次，此时用FlinkKafkaConsumer实时消费kafka中的消息

2. 消费并处理完消息后，进行一次预提交数据库的操作

3. 如果预提交没有问题，10s后进行真正的插入数据库操作，如果插入成功，进行一次checkpoint，flink会自动记录消费的offset，可以将checkpoint保存的数据放到hdfs中

4. 如果预提交出错，比如在5s的时候出错了，此时Flink程序就会进入不断的重启中，重启的策略可以在配置中设置，checkpoint记录的还是上一次成功消费的offset，因为本次消费的数据在checkpoint期间，消费成功，但是预提交过程中失败了

5. 注意此时数据并没有真正的执行插入操作，因为预提交（preCommit）失败，提交（commit）过程也不会发生。等将异常数据处理完成之后，再重新启动这个Flink程序，它会自动从上一次成功的checkpoint中继续消费数据，以此来达到Kafka到Mysql的Exactly-Once。

~~~ java
public class Kafka_Flink_MySQL_EndToEnd_ExactlyOnce {

    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);//方便测试
        env.enableCheckpointing(10000);
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        env.getCheckpointConfig().setMinPauseBetweenCheckpoints(1000);
        //env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);
        env.setStateBackend(new FsStateBackend("file:///D:/ckp"));

        //2.Source
        String topic = "flink_kafka";
        Properties props = new Properties();
        props.setProperty(CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG,"node1:9092");
        props.setProperty("group.id","flink");
        props.setProperty("auto.offset.reset","latest");//如果有记录偏移量从记录的位置开始消费,如果没有从最新的数据开始消费
        props.setProperty("flink.partition-discovery.interval-millis","5000");//开一个后台线程每隔5s检查Kafka的分区状态
        FlinkKafkaConsumer<ObjectNode> kafkaSource = new FlinkKafkaConsumer<>("topic_in", new JSONKeyValueDeserializationSchema(true), props);

        kafkaSource.setStartFromGroupOffsets();//从group offset记录的位置位置开始消费,如果kafka broker 端没有该group信息，会根据"auto.offset.reset"的设置来决定从哪开始消费
        kafkaSource.setCommitOffsetsOnCheckpoints(true);//Flink执行Checkpoint的时候提交偏移量(一份在Checkpoint中,一份在Kafka的默认主题中__comsumer_offsets(方便外部监控工具去看))

        DataStreamSource<ObjectNode> kafkaDS = env.addSource(kafkaSource);

        //3.transformation

        //4.Sink
        kafkaDS.addSink(new MySqlTwoPhaseCommitSink()).name("MySqlTwoPhaseCommitSink");

        //5.execute
        env.execute();
    }
}

/**
 自定义kafka to mysql，继承TwoPhaseCommitSinkFunction,实现两阶段提交。
 功能：保证kafak to mysql 的Exactly-Once
 CREATE TABLE `t_test` (
   `id` bigint(20) NOT NULL AUTO_INCREMENT,
   `value` varchar(255) DEFAULT NULL,
   `insert_time` datetime DEFAULT NULL,
   PRIMARY KEY (`id`)
 ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
 */
class MySqlTwoPhaseCommitSink extends TwoPhaseCommitSinkFunction<ObjectNode, Connection, Void> {

    public MySqlTwoPhaseCommitSink() {
        super(new KryoSerializer<>(Connection.class, new ExecutionConfig()), VoidSerializer.INSTANCE);
    }

    /**
     * 执行数据入库操作
     */
    @Override
    protected void invoke(Connection connection, ObjectNode objectNode, Context context) throws Exception {
        System.err.println("start invoke.......");
        String date = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
        System.err.println("===>date:" + date + " " + objectNode);
        String value = objectNode.get("value").toString();
        String sql = "insert into `t_test` (`value`,`insert_time`) values (?,?)";
        PreparedStatement ps = connection.prepareStatement(sql);
        ps.setString(1, value);
        ps.setTimestamp(2, new Timestamp(System.currentTimeMillis()));
        //执行insert语句
        ps.execute();
        //手动制造异常
        if(Integer.parseInt(value) == 15) System.out.println(1/0);
    }

    /**
     * 获取连接，开启手动提交事务（getConnection方法中）
     */
    @Override
    protected Connection beginTransaction() throws Exception {
        String url = "jdbc:mysql://localhost:3306/bigdata?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&useSSL=false&autoReconnect=true";
        Connection connection = DBConnectUtil.getConnection(url, "root", "root");
        System.err.println("start beginTransaction......."+connection);
        return connection;
    }

    /**
     * 预提交，这里预提交的逻辑在invoke方法中
     */
    @Override
    protected void preCommit(Connection connection) throws Exception {
        System.err.println("start preCommit......."+connection);

    }

    /**
     * 如果invoke执行正常则提交事务
     */
    @Override
    protected void commit(Connection connection) {
        System.err.println("start commit......."+connection);
        DBConnectUtil.commit(connection);

    }

    @Override
    protected void recoverAndCommit(Connection connection) {
        System.err.println("start recoverAndCommit......."+connection);

    }

    @Override
    protected void recoverAndAbort(Connection connection) {
        System.err.println("start abort recoverAndAbort......."+connection);
    }

    /**
     * 如果invoke执行异常则回滚事务，下一次的checkpoint操作也不会执行
     */
    @Override
    protected void abort(Connection connection) {
        System.err.println("start abort rollback......."+connection);
        DBConnectUtil.rollback(connection);
    }
}

class DBConnectUtil {
    /**
     * 获取连接
     */
    public static Connection getConnection(String url, String user, String password) throws SQLException {
        Connection conn = null;
        conn = DriverManager.getConnection(url, user, password);
        //设置手动提交
        conn.setAutoCommit(false);
        return conn;
    }

    /**
     * 提交事务
     */
    public static void commit(Connection conn) {
        if (conn != null) {
            try {
                conn.commit();
            } catch (SQLException e) {
                e.printStackTrace();
            } finally {
                close(conn);
            }
        }
    }

    /**
     * 事务回滚
     */
    public static void rollback(Connection conn) {
        if (conn != null) {
            try {
                conn.rollback();
            } catch (SQLException e) {
                e.printStackTrace();
            } finally {
                close(conn);
            }
        }
    }

    /**
     * 关闭连接
     */
    public static void close(Connection conn) {
        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}

public class DataProducer {
    public static void main(String[] args) throws InterruptedException {
        Properties props = new Properties();
        props.put("bootstrap.servers", "node1:9092");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        Producer<String, String> producer = new org.apache.kafka.clients.producer.KafkaProducer<>(props);

        try {
            for (int i = 1; i <= 20; i++) {
                DataBean data = new DataBean(String.valueOf(i));
                ProducerRecord record = new ProducerRecord<String, String>("flink_kafka", null, null, JSON.toJSONString(data));
                producer.send(record);
                System.out.println("发送数据: " + JSON.toJSONString(data));
                Thread.sleep(1000);
            }
        }catch (Exception e){
            System.out.println(e);
        }
        producer.flush();
    }
}

@Data
@NoArgsConstructor
@AllArgsConstructor
class DataBean {
    private String value;
}
~~~

### Streaming File Sink

#### 介绍

把流数据写入到文件系统中。

StreamingFileSink是Flink1.7中推出的新特性，是为了解决如下的问题：

大数据业务场景中，经常有一种场景：外部数据发送到kafka中，flink作为中间件消费kafka数据并进行业务处理；处理完成之后的数据可能还需要写入到数据库或者文件系统中，比如写入hdfs中。

StreamingFileSink就可以用来将分区文件写入到支持 Flink FileSystem接口的文件系统中，支持Exactly-Once语义。

这种sink实现的Exactly-Once都是基于Flink checkpoint来实现的两阶段提交模式来保证的，主要应用在实时数仓、topic拆分、基于小时分析处理等场景下。

#### Bucket和SubTask、PartFile

- Bucket

StreamingFileSink可向由Flink FileSystem抽象支持的文件系统写入分区文件（因为是流式写入，数据被视为无界）。该分区行为可配，默认按时间，具体来说每小时写入一个Bucket，该Bucket包括若干文件，内容是这一小时间隔内流中收到的所有record。

- PartFile

每个Bukcket内部分为多个PartFile来存储输出数据，该Bucket生命周期内接收到数据的sink的每个子任务至少有一个PartFile。

而额外文件滚动由可配的滚动策略决定，默认策略是根据文件大小和打开超时（文件可以被打开的最大持续时间）以及文件最大不活动超时等决定是否滚动。

Bucket和SubTask、PartFile关系如图所示

![1623718522665](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623718522665.png)

#### 案例

**需求**

编写Flink程序，接收socket的字符串数据，然后将接收到的数据流式方式存储到hdfs

开发步骤

1. 初始化流计算运行环境

2. 设置Checkpoint（10s）周期性启动

3. 指定并行度为1

4. 接入socket数据源，获取数据

5. 指定文件编码格式为行编码格式

6. 设置桶分配策略

7. 设置文件滚动策略

8. 指定文件输出配置

9. 将streamingfilesink对象添加到环境

10. 执行任务

**代码实现**

~~~ java
import org.apache.flink.api.common.serialization.SimpleStringEncoder;
import org.apache.flink.core.fs.Path;
import org.apache.flink.runtime.state.filesystem.FsStateBackend;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.filesystem.OutputFileConfig;
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink;
import org.apache.flink.streaming.api.functions.sink.filesystem.bucketassigners.DateTimeBucketAssigner;
import org.apache.flink.streaming.api.functions.sink.filesystem.rollingpolicies.DefaultRollingPolicy;

import java.util.concurrent.TimeUnit;

public class StreamFileSinkDemo {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.enableCheckpointing(TimeUnit.SECONDS.toMillis(10));
        env.setStateBackend(new FsStateBackend("file:///D:/ckp"));

        //2.source
        DataStreamSource<String> lines = env.socketTextStream("node1", 9999);

        //3.sink
        //设置sink的前缀和后缀
        //文件的头和文件扩展名
        //prefix-xxx-.txt
        OutputFileConfig config = OutputFileConfig
                .builder()
                .withPartPrefix("prefix")
                .withPartSuffix(".txt")
                .build();

        //设置sink的路径
        String outputPath = "hdfs://node1:8020/FlinkStreamFileSink/parquet";

        //创建StreamingFileSink
        final StreamingFileSink<String> sink = StreamingFileSink
                .forRowFormat(
                        new Path(outputPath),
                        new SimpleStringEncoder<String>("UTF-8"))
                /**
                 * 设置桶分配政策
                 * DateTimeBucketAssigner --默认的桶分配政策，默认基于时间的分配器，每小时产生一个桶，格式如下yyyy-MM-dd--HH
                 * BasePathBucketAssigner ：将所有部分文件（part file）存储在基本路径中的分配器（单个全局桶）
                 */
                .withBucketAssigner(new DateTimeBucketAssigner<>())
                /**
                 * 有三种滚动政策
                 *  CheckpointRollingPolicy
                 *  DefaultRollingPolicy
                 *  OnCheckpointRollingPolicy
                 */
                .withRollingPolicy(
                        /**
                         * 滚动策略决定了写出文件的状态变化过程
                         * 1. In-progress ：当前文件正在写入中
                         * 2. Pending ：当处于 In-progress 状态的文件关闭（closed）了，就变为 Pending 状态
                         * 3. Finished ：在成功的 Checkpoint 后，Pending 状态将变为 Finished 状态
                         *
                         * 观察到的现象
                         * 1.会根据本地时间和时区，先创建桶目录
                         * 2.文件名称规则：part-<subtaskIndex>-<partFileIndex>
                         * 3.在macos中默认不显示隐藏文件，需要显示隐藏文件才能看到处于In-progress和Pending状态的文件，因为文件是按照.开头命名的
                         *
                         */
                        DefaultRollingPolicy.builder()
                                .withRolloverInterval(TimeUnit.SECONDS.toMillis(2)) //设置滚动间隔
                                .withInactivityInterval(TimeUnit.SECONDS.toMillis(1)) //设置不活动时间间隔
                                .withMaxPartSize(1024 * 1024 * 1024) // 最大尺寸
                                .build())
                .withOutputFileConfig(config)
                .build();

        lines.addSink(sink).setParallelism(1);

        env.execute();
    }
}
~~~

#### 配置详解

##### PartFile

前面提到过，每个Bukcket内部分为多个部分文件，该Bucket内接收到数据的sink的每个子任务至少有一个PartFile。而额外文件滚动由可配的滚动策略决定。

- 关于顺序性

对于任何给定的Flink子任务，PartFile索引都严格增加（按创建顺序），但是，这些索引并不总是顺序的。当作业重新启动时，所有子任务的下一个PartFile索引将是max PartFile索引+ 1，其中max是指在所有子任务中对所有计算的索引最大值。

~~~ java
return new Path(bucketPath, outputFileConfig.getPartPrefix() + '-' + subtaskIndex + '-' + partCounter + outputFileConfig.getPartSuffix());
~~~

###### PartFile生命周期

输出文件的命名规则和生命周期。由上图可知，部分文件（part file）可以处于以下三种状态之一：

- In-progress ：

当前文件正在写入中

- Pending ：

当处于 In-progress 状态的文件关闭（closed）了，就变为 Pending 状态

- Finished ：

在成功的 Checkpoint 后，Pending 状态将变为 Finished 状态,处于 Finished 状态的文件不会再被修改，可以被下游系统安全地读取。

- 注意： 

使用 StreamingFileSink 时需要启用 Checkpoint ，每次做 Checkpoint 时写入完成。如果 Checkpoint 被禁用，部分文件（part file）将永远处于 'in-progress' 或 'pending' 状态，下游系统无法安全地读取。

 ###### PartFile的生成规则

在每个活跃的Bucket期间，每个Writer的子任务在任何时候都只会有一个单独的In-progress PartFile，但可有多个Peding和Finished状态文件。

一个Sink的两个Subtask的PartFile分布情况实例如下:

- 初始状态，两个inprogress文件正在被两个subtask分别写入

~~~ java
└── 2020-03-25--12
    ├── part-0-0.inprogress.bd053eb0-5ecf-4c85-8433-9eff486ac334
    └── part-1-0.inprogress.ea65a428-a1d0-4a0b-bbc5-7a436a75e575
~~~

- 当part-1-0因文件大小超过阈值等原因发生滚动时，变为Pending状态等待完成，但此时不会被重命名。注意此时Sink会创建一个新的PartFile即part-1-1：

~~~ java
└── 2020-03-25--12
    ├── part-0-0.inprogress.bd053eb0-5ecf-4c85-8433-9eff486ac334
    ├── part-1-0.inprogress.ea65a428-a1d0-4a0b-bbc5-7a436a75e575
    └── part-1-1.inprogress.bc279efe-b16f-47d8-b828-00ef6e2fbd11
~~~

- 待下次checkpoint成功后，part-1-0完成变为Finished状态，被重命名：

~~~ java
└── 2020-03-25--12
    ├── part-0-0.inprogress.bd053eb0-5ecf-4c85-8433-9eff486ac334
    ├── part-1-0
    └── part-1-1.inprogress.bc279efe-b16f-47d8-b828-00ef6e2fbd11
~~~

- 一个Bucket周期到了，创建新的Bucket目录，不影响之前Bucket内的的in-progress文件，依然要等待文件RollingPolicy以及checkpoint来改变状态：

~~~ java
└── 2020-03-25--12
    ├── part-0-0.inprogress.bd053eb0-5ecf-4c85-8433-9eff486ac334
    ├── part-1-0
    └── part-1-1.inprogress.bc279efe-b16f-47d8-b828-00ef6e2fbd11
└── 2020-03-25--13
    └── part-0-2.inprogress.2b475fec-1482-4dea-9946-eb4353b475f1
~~~

#####  PartFile序列化编码

StreamingFileSink 支持行编码格式和批量编码格式，比如 Apache Parquet 。这两种变体可以使用以下静态方法创建：

- Row-encoded sink: 

~~~ java
StreamingFileSink.forRowFormat(basePath, rowEncoder)
  
  //行
StreamingFileSink.forRowFormat(new Path(path), new SimpleStringEncoder<T>())
        .withBucketAssigner(new PaulAssigner<>()) //分桶策略
        .withRollingPolicy(new PaulRollingPolicy<>()) //滚动策略
        .withBucketCheckInterval(CHECK_INTERVAL) //检查周期
        .build();
~~~

- Bulk-encoded sink:

~~~ java
StreamingFileSink.forBulkFormat(basePath, bulkWriterFactory)
  //列 parquet
StreamingFileSink.forBulkFormat(new Path(path), ParquetAvroWriters.forReflectRecord(clazz))
        .withBucketAssigner(new PaulBucketAssigner<>())
        .withBucketCheckInterval(CHECK_INTERVAL)
        .build();

~~~

创建行或批量编码的 Sink 时，我们需要指定存储桶的基本路径和数据的编码

这两种写入格式除了文件格式的不同，另外一个很重要的区别就是回滚策略的不同：

- orRowFormat行写可基于文件大小、滚动时间、不活跃时间进行滚动，

- forBulkFormat列写方式只能基于checkpoint机制进行文件滚动，即在执行snapshotState方法时滚动文件，如果基于大小或者时间滚动文件，那么在任务失败恢复时就必须对处于in-processing状态的文件按照指定的offset进行truncate，由于列式存储是无法针对文件offset进行truncate的，因此就必须在每次checkpoint使文件滚动，其使用的滚动策略实现是OnCheckpointRollingPolicy。

forBulkFormat只能和 `OnCheckpointRollingPolicy` 结合使用，每次做 checkpoint 时滚动文件。

###### Row Encoding

此时，StreamingFileSink会以每条记录为单位进行编码和序列化。

必须配置项：

- 输出数据的BasePath

- 序列化每行数据写入PartFile的Encoder

使用RowFormatBuilder可选配置项：

- 自定义RollingPolicy

默认使用DefaultRollingPolicy来滚动文件，可自定义

- bucketCheckInterval

默认1分钟。该值单位为毫秒，指定按时间滚动文件间隔时间

例子如下：

~~~ java
import org.apache.flink.api.common.serialization.SimpleStringEncoder
import org.apache.flink.core.fs.Path
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink

// 1. 构建DataStream
DataStream input  = ...
// 2. 构建StreamingFileSink，指定BasePath、Encoder、RollingPolicy
StreamingFileSink sink  = StreamingFileSink
    .forRowFormat(new Path(outputPath), new SimpleStringEncoder[String]("UTF-8"))
    .withRollingPolicy(
        DefaultRollingPolicy.builder()
            .withRolloverInterval(TimeUnit.MINUTES.toMillis(15))
            .withInactivityInterval(TimeUnit.MINUTES.toMillis(5))
            .withMaxPartSize(1024 * 1024 * 1024)
            .build())
    .build()
// 3. 添加Sink到InputDataSteam即可
input.addSink(sink)
~~~

以上例子构建了一个简单的拥有默认Bucket构建行为（继承自BucketAssigner的DateTimeBucketAssigner）的StreamingFileSink，每小时构建一个Bucket，内部使用继承自RollingPolicy的DefaultRollingPolicy，以下三种情况任一发生会滚动PartFile：

- PartFile包含至少15分钟的数据

- 在过去5分钟内没有接收到新数据

- 在最后一条记录写入后，文件大小已经达到1GB

除了使用DefaultRollingPolicy，也可以自己实现RollingPolicy接口来实现自定义滚动策略。

###### Bulk Encoding

要使用批量编码，请将StreamingFileSink.forRowFormat()替换为StreamingFileSink.forBulkFormat()，注意此时必须指定一个BulkWriter.Factory而不是行模式的Encoder。BulkWriter在逻辑上定义了如何添加、fllush新记录以及如何最终确定记录的bulk以用于进一步编码。

需要注意的是，使用Bulk Encoding时，Filnk1.9版本的文件滚动就只能使用OnCheckpointRollingPolicy的策略，该策略在每次checkpoint时滚动part-file。

Flink有三个内嵌的BulkWriter：

- ParquetAvroWriters

有一些静态方法来创建ParquetWriterFactory。

- SequenceFileWriterFactory

- CompressWriterFactory

Flink有内置方法可用于为Avro数据创建Parquet writer factory。

要使用ParquetBulkEncoder，需要添加以下Maven依赖：

~~~ java
<!-- streaming File Sink所需要的jar包-->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-parquet_2.12</artifactId>
    <version>1.12.0</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.apache.avro/avro -->
<dependency>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro</artifactId>
    <version>1.12.0</version>
</dependency>

<dependency>
    <groupId>org.apache.parquet</groupId>
    <artifactId>parquet-avro</artifactId>
    <version>1.12.0</version>
</dependency>

~~~

##### 桶分配策略

桶分配策略定义了将数据结构化后写入基本输出目录中的子目录，行格式和批量格式都需要使用。

具体来说，StreamingFileSink使用BucketAssigner来确定每条输入的数据应该被放入哪个Bucket，

默认情况下，DateTimeBucketAssigner 基于系统默认时区每小时创建一个桶：

格式如下：yyyy-MM-dd--HH。日期格式（即桶的大小）和时区都可以手动配置。

我们可以在格式构建器上调用 .withBucketAssigner(assigner) 来自定义 BucketAssigner。

Flink 有两个内置的 BucketAssigners ：

- DateTimeBucketAssigner：默认基于时间的分配器

- BasePathBucketAssigner：将所有部分文件（part file）存储在基本路径中的分配器（单个全局桶）

###### DateTimeBucketAssigner

Row格式和Bulk格式编码都使用DateTimeBucketAssigner作为默认BucketAssigner。 默认情况下，DateTimeBucketAssigner 基于系统默认时区每小时以格式yyyy-MM-dd--HH来创建一个Bucket，Bucket路径为/{basePath}/{dateTimePath}/。

- basePath是指StreamingFileSink.forRowFormat(new Path(outputPath)时的路径

- dateTimePath中的日期格式和时区都可在初始化DateTimeBucketAssigner时配置

~~~ java
public class DateTimeBucketAssigner<IN> implements BucketAssigner<IN, String> {
private static final long serialVersionUID = 1L;

	// 默认的时间格式字符串
	private static final String DEFAULT_FORMAT_STRING = "yyyy-MM-dd--HH";

	// 时间格式字符串
	private final String formatString;

	// 时区
	private final ZoneId zoneId;
	
	// DateTimeFormatter被用来通过当前系统时间和DateTimeFormat来生成时间字符串
	private transient DateTimeFormatter dateTimeFormatter;

	/**
	 * 使用默认的`yyyy-MM-dd--HH`和系统时区构建DateTimeBucketAssigner
	 */
	public DateTimeBucketAssigner() {
		this(DEFAULT_FORMAT_STRING);
	}

	/**
	 * 通过能被SimpleDateFormat解析的时间字符串和系统时区
	 * 来构建DateTimeBucketAssigner
	 */
	public DateTimeBucketAssigner(String formatString) {
		this(formatString, ZoneId.systemDefault());
	}

	/**
	 * 通过默认的`yyyy-MM-dd--HH`和指定的时区
	 * 来构建DateTimeBucketAssigner
	 */
	public DateTimeBucketAssigner(ZoneId zoneId) {
		this(DEFAULT_FORMAT_STRING, zoneId);
	}

	/**
	 * 通过能被SimpleDateFormat解析的时间字符串和指定的时区
	 * 来构建DateTimeBucketAssigner
	 */
	public DateTimeBucketAssigner(String formatString, ZoneId zoneId) {
		this.formatString = Preconditions.checkNotNull(formatString);
		this.zoneId = Preconditions.checkNotNull(zoneId);
	}

	/**
	 * 使用指定的时间格式和时区来格式化当前ProcessingTime，以获取BucketId
	 */
	@Override
	public String getBucketId(IN element, BucketAssigner.Context context) {
		if (dateTimeFormatter == null) {
			dateTimeFormatter = DateTimeFormatter.ofPattern(formatString).withZone(zoneId);
		}
		return dateTimeFormatter.format(Instant.ofEpochMilli(context.currentProcessingTime()));
	}

	@Override
	public SimpleVersionedSerializer<String> getSerializer() {
		return SimpleVersionedStringSerializer.INSTANCE;
	}

	@Override
	public String toString() {
		return "DateTimeBucketAssigner{" +
			"formatString='" + formatString + '\'' +
			", zoneId=" + zoneId +
			'}';
	}
}
~~~

###### BasePathBucketAssigner

将所有PartFile存储在BasePath中（此时只有单个全局Bucket）。

先看看BasePathBucketAssigner的源码，方便继续学习DateTimeBucketAssigner：

~~~ java
@PublicEvolving
public class BasePathBucketAssigner<T> implements BucketAssigner<T, String> {
	private static final long serialVersionUID = -6033643155550226022L;
	/**
	 * BucketId永远为""，即Bucket全路径为用户指定的BasePath
	 */
	@Override
	public String getBucketId(T element, BucketAssigner.Context context) {
		return "";
	}
	/**
	 * 用SimpleVersionedStringSerializer来序列化BucketId
	 */
	@Override
	public SimpleVersionedSerializer<String> getSerializer() {
		// in the future this could be optimized as it is the empty string.
		return SimpleVersionedStringSerializer.INSTANCE;
	}

	@Override
	public String toString() {
		return "BasePathBucketAssigner";
	}
}
~~~

##### 滚动策略   

滚动策略 RollingPolicy 定义了指定的文件在何时关闭（closed）并将其变为 Pending 状态，随后变为 Finished 状态。处于 Pending 状态的文件会在下一次 Checkpoint 时变为 Finished 状态，通过设置 Checkpoint 间隔时间，可以控制部分文件（part file）对下游读取者可用的速度、大小和数量。

Flink 有两个内置的滚动策略：

- DefaultRollingPolicy

- OnCheckpointRollingPolicy

需要注意的是，使用Bulk Encoding时，文件滚动就只能使用OnCheckpointRollingPolicy的策略，该策略在每次checkpoint时滚动part-file。

##### 案例


~~~ java
public class Test27 {

    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.enableCheckpointing(TimeUnit.SECONDS.toMillis(10));
        env.setStateBackend(new FsStateBackend("file:///D:/ckp"));

        //2.source
        DataStreamSource<String> lines = env.socketTextStream("node1", 9999);

        //3.sink
        //设置sink的前缀和后缀
        //文件的头和文件扩展名
        //prefix-xxx-.txt
        OutputFileConfig config = OutputFileConfig
                .builder()
                .withPartPrefix("prefix")
                .withPartSuffix(".txt")
                .build();

        //设置sink的路径
        String outputPath = "hdfs://node1:8020/FlinkStreamFileSink/parquet";

        //创建StreamingFileSink
        final StreamingFileSink<String> sink = StreamingFileSink
                .forRowFormat(
                        new Path(outputPath),
                        new SimpleStringEncoder<String>("UTF-8"))
                /**
                 * 设置桶分配政策
                 * DateTimeBucketAssigner --默认的桶分配政策，默认基于时间的分配器，每小时产生一个桶，格式如下yyyy-MM-dd--HH
                 * BasePathBucketAssigner ：将所有部分文件（part file）存储在基本路径中的分配器（单个全局桶）
                 */
                .withBucketAssigner(new DateTimeBucketAssigner<>())
                /**
                 * 有三种滚动政策
                 *  CheckpointRollingPolicy
                 *  DefaultRollingPolicy
                 *  OnCheckpointRollingPolicy
                 */
                .withRollingPolicy(
                        /**
                         * 滚动策略决定了写出文件的状态变化过程
                         * 1. In-progress ：当前文件正在写入中
                         * 2. Pending ：当处于 In-progress 状态的文件关闭（closed）了，就变为 Pending 状态
                         * 3. Finished ：在成功的 Checkpoint 后，Pending 状态将变为 Finished 状态
                         *
                         * 观察到的现象
                         * 1.会根据本地时间和时区，先创建桶目录
                         * 2.文件名称规则：part-<subtaskIndex>-<partFileIndex>
                         * 3.在macos中默认不显示隐藏文件，需要显示隐藏文件才能看到处于In-progress和Pending状态的文件，因为文件是按照.开头命名的
                         *
                         */
                        DefaultRollingPolicy.builder()
                                .withRolloverInterval(TimeUnit.SECONDS.toMillis(2)) //设置滚动间隔
                                .withInactivityInterval(TimeUnit.SECONDS.toMillis(1)) //设置不活动时间间隔
                                .withMaxPartSize(1024 * 1024 * 1024) // 最大尺寸
                                .build())
                .withOutputFileConfig(config)
                .build();

        lines.addSink(sink).setParallelism(1);

        env.execute();
    }

}
~~~

### File Sink

![1623562109507](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623562109507.png)

新的 Data Sink API (Beta)

之前发布的 Flink 版本中[1]，已经支持了 source connector 工作在流批两种模式下，因此在 Flink 1.12 中，社区着重实现了统一的 Data Sink API（FLIP-143）。新的抽象引入了 write/commit 协议和一个更加模块化的接口。Sink 的实现者只需要定义 what 和 how：SinkWriter，用于写数据，并输出需要 commit 的内容（例如，committables）；Committer 和 GlobalCommitter，封装了如何处理 committables。框架会负责 when 和 where：即在什么时间，以及在哪些机器或进程中 commit。

![1623562150355](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623562150355.png)

这种模块化的抽象允许为 BATCH 和 STREAMING 两种执行模式，实现不同的运行时策略，以达到仅使用一种 sink 实现，也可以使两种模式都可以高效执行。Flink 1.12 中，提供了统一的 FileSink connector，以替换现有的 StreamingFileSink connector （FLINK-19758）。其它的 connector 也将逐步迁移到新的接口。

Flink 1.12的 FileSink 为批处理和流式处理提供了一个统一的接收器，它将分区文件写入Flink文件系统抽象所支持的文件系统。这个文件系统连接器为批处理和流式处理提供了相同的保证，它是现有流式文件接收器的一种改进。

**案例**

~~~ java
public class FileSinkDemo {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.enableCheckpointing(TimeUnit.SECONDS.toMillis(10));
        env.setStateBackend(new FsStateBackend("file:///D:/ckp"));

        //2.source
        DataStreamSource<String> lines = env.socketTextStream("node1", 9999);

        //3.sink
        //设置sink的前缀和后缀
        //文件的头和文件扩展名
        //prefix-xxx-.txt
        OutputFileConfig config = OutputFileConfig
                .builder()
                .withPartPrefix("prefix")
                .withPartSuffix(".txt")
                .build();

        //设置sink的路径
        String outputPath = "hdfs://node1:8020/FlinkFileSink/parquet";

        final FileSink<String> sink = FileSink
                .forRowFormat(new Path(outputPath), new SimpleStringEncoder<String>("UTF-8"))
                .withBucketAssigner(new DateTimeBucketAssigner<>())
                .withRollingPolicy(
                        DefaultRollingPolicy.builder()
                                .withRolloverInterval(TimeUnit.MINUTES.toMillis(15))
                                .withInactivityInterval(TimeUnit.MINUTES.toMillis(5))
                                .withMaxPartSize(1024 * 1024 * 1024)
                                .build())
                .withOutputFileConfig(config)
                .build();

        lines.sinkTo(sink).setParallelism(1);

        env.execute();
    }
}
~~~

## Flink监控与优化

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

~~~ java
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
~~~

○  程序启动之后就可以在任务的ui界面上查看

![1623562965529](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/134247-604597.png)







### Flink性能优化

#### History Server

flink的HistoryServer主要是用来存储和查看任务的历史记录

~~~ java
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

~~~

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

~~~ java
stream
    .apply(new WindowFunction<WikipediaEditEvent, Tuple2<String, Long>, String, TimeWindow>() {
        @Override
        public void apply(String userName, TimeWindow timeWindow, Iterable<WikipediaEditEvent> iterable, Collector<Tuple2<String, Long>> collector) throws Exception {
            long changesCount = ...
            // A new Tuple instance is created on every execution
            collector.collect(new Tuple2<>(userName, changesCount));
        }
    }

~~~

可以看出，apply函数每执行一次，都会新建一个Tuple2类的实例，因此增加了对垃圾收集器的压力。解决这个问题的一种方法是反复使用相同的实例：

~~~ java
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
~~~

这种做法其实还间接创建了Long类的实例。

为了解决这个问题，Flink有许多所谓的value class:IntValue、LongValue、StringValue、FloatValue等。下面介绍一下如何使用它们：

~~~ java
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
~~~

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

![1623564913926](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623564913926.png)

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

![1623565054635](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623565054635.png)

针对前六种类型数据集，Flink皆可以自动生成对应的TypeSerializer，能非常高效地对数据集进行序列化和反序列化。对于最后一种数据类型，Flink会使用Kryo进行序列化和反序列化。每个TypeInformation中，都包含了serializer，类型会自动通过serializer进行序列化，然后用Java Unsafe接口(具有像C语言一样的操作内存空间的能力)写入MemorySegments。

![1623565090162](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623565090162.png)

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

![1623565154769](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623565154769.png)

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

![1623569442344](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623569442344.png)

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

![1623572163119](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623572163119.png)



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

![1623573551196](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623573551196.png)





**补充**

![1623573085238](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/13/163129-179167.png)